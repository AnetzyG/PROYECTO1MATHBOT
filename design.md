# Diseño Técnico — MathBot

## 1. Visión general

MathBot es un sistema de prompting estructurado sobre un LLM instruct que corre localmente en Google Colab. No es un producto desplegado — es un pipeline reproducible en notebooks que demuestra el uso de LLMs para educación matemática.

```
┌─────────────────────────────────────────────────────────┐
│                    Google Colab (T4)                     │
│                                                         │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │  Corpus  │───▶│ Prompt       │───▶│ Qwen2.5-1.5B │  │
│  │  (JSON)  │    │ Builder      │    │ Instruct     │  │
│  └──────────┘    └──────────────┘    └──────┬───────┘  │
│                                             │           │
│                                             ▼           │
│                                      ┌──────────────┐   │
│                                      │  Output      │   │
│                                      │  Parser      │   │
│                                      └──────┬───────┘   │
│                                             │           │
│                                             ▼           │
│                                      ┌──────────────┐   │
│                                      │  Evaluación  │   │
│                                      └──────────────┘   │
└─────────────────────────────────────────────────────────┘
```

## 2. Decisiones de arquitectura

### D1: Prompting vs Fine-tuning

**Decisión**: Empezar con prompting puro (zero-shot → few-shot → chain-of-thought). Fine-tuning solo si el prompting no alcanza el baseline esperado.

**Razón**: 
- Qwen2.5-1.5B-Instruct ya sigue instrucciones en español
- El dominio (aritmética de bachillerato) es conocimiento que el modelo ya tiene del pretraining
- Fine-tuning con LoRA es una fase posterior opcional
- Menos complejidad = más reproducible

### D2: Modelo único vs múltiples modelos

**Decisión**: Un solo modelo (`Qwen/Qwen2.5-1.5B-Instruct`) para las 3 tareas.

**Razón**:
- Simplifica el pipeline
- 1.5B cabe cómodamente en T4 (~3 GB de 15 GB disponibles)
- Las 3 tareas son variantes de generación condicional — mismo modelo, diferente prompt

### D3: RAG vs contexto directo

**Decisión**: Fase inicial sin RAG. El corpus se inyecta como few-shot examples directamente en el prompt. RAG se evalúa en fase 4 solo si el corpus crece más allá de lo que cabe en el contexto.

**Razón**:
- Con 50-100 ejercicios, los examples relevantes caben en el context window (32K tokens para Qwen2.5)
- RAG agrega complejidad (embeddings, vector store, retriever) sin beneficio claro para un corpus pequeño
- Si el corpus crece a 500+, RAG se justifica

### D4: Formato de salida estructurado

**Decisión**: El modelo genera en formato markdown con secciones fijas. Se parsea con regex simple.

**Razón**:
- JSON es frágil con modelos de 1.5B (tienden a romper la estructura)
- Markdown con headers (`## Enunciado`, `## Solución`) es más robusto
- Fácil de parsear y de leer para el docente

## 3. Componentes

### 3.1 Corpus (`data/corpus.json`)

Archivo JSON con la colección de ejercicios etiquetados.

```python
# Estructura
corpus = [
    {
        "id": "va_001",
        "tema": "valor_absoluto",
        "subtema": "ecuaciones",
        "problema": "Resuelve |2x - 3| = 5",
        "solucion": "x = 4, x = -1",
        "procedimiento": [
            "Caso 1: 2x - 3 = 5 → 2x = 8 → x = 4",
            "Caso 2: 2x - 3 = -5 → 2x = -2 → x = -1"
        ],
        "errores_comunes": [
            {
                "respuesta_incorrecta": "x = 4",
                "tipo": "omision_solucion",
                "explicacion": "Solo consideró el caso positivo, falta |2x-3| = -5"
            }
        ],
        "dificultad": "basico",
        "nivel": "ENP_mat4"
    }
]
```

### 3.2 Prompt Builder (`src/prompts.py`)

Módulo que construye los prompts para cada tarea. Cada función retorna una lista de mensajes en formato chat template.

```python
# Interfaz
def build_generation_prompt(tema: str, dificultad: str, examples: list[dict]) -> list[dict]:
    """Construye prompt para generar un ejercicio."""
    # Retorna [{"role": "system", ...}, {"role": "user", ...}]

def build_detection_prompt(problema: str, respuesta_estudiante: str, examples: list[dict]) -> list[dict]:
    """Construye prompt para detectar errores."""

def build_explanation_prompt(ejercicio: str, examples: list[dict]) -> list[dict]:
    """Construye prompt para explicar un procedimiento."""
```

**System prompts por tarea:**

- **Generación**: "Eres un profesor de matemáticas de bachillerato. Genera ejercicios sobre {tema} de dificultad {dificultad}. Incluye enunciado, procedimiento paso a paso y respuesta final."
- **Detección**: "Eres un profesor de matemáticas revisando la tarea de un estudiante. Analiza si la respuesta es correcta. Si hay error, identifica el tipo de error y explica la corrección."
- **Explicación**: "Eres un profesor de matemáticas explicando a un estudiante de bachillerato. Usa lenguaje claro y sencillo. Muestra cada paso del procedimiento."

### 3.3 Modelo (`src/model.py`)

Carga y configuración del modelo. Wrapper simple sobre transformers.

```python
# Interfaz
def load_model(model_id: str = "Qwen/Qwen2.5-1.5B-Instruct") -> tuple[AutoTokenizer, AutoModelForCausalLM]:
    """Carga modelo y tokenizer. Detecta GPU automáticamente."""

def generate(messages: list[dict], tokenizer, model, max_new_tokens: int = 512, temperature: float = 0.7) -> str:
    """Genera respuesta dado un prompt en formato chat."""
```

### 3.4 Output Parser (`src/parser.py`)

Extrae las secciones de la respuesta del modelo.

```python
# Interfaz
def parse_exercise(response: str) -> dict:
    """Extrae enunciado, procedimiento y respuesta de un ejercicio generado."""
    # Retorna {"enunciado": ..., "procedimiento": ..., "respuesta": ...}

def parse_error_detection(response: str) -> dict:
    """Extrae error identificado, tipo y corrección."""
    # Retorna {"tiene_error": bool, "tipo": ..., "explicacion": ..., "correccion": ...}
```

### 3.5 Evaluador (`src/evaluator.py`)

Métricas automáticas donde sea posible, registro para evaluación manual.

```python
# Interfaz
def evaluate_generation(generated: list[dict], corpus: list[dict]) -> dict:
    """Evalúa batch de ejercicios generados. Retorna métricas."""

def evaluate_detection(predictions: list[dict], ground_truth: list[dict]) -> dict:
    """Evalúa detección de errores contra ground truth. Retorna accuracy, F1 por tipo."""
```

## 4. Estructura de notebooks

```
Notebook/
├── 01_corpus_exploration.ipynb      ← Carga, visualiza y valida el corpus
├── 02_baseline_generation.ipynb     ← Generación zero-shot, mide correctitud
├── 03_baseline_detection.ipynb      ← Detección zero-shot, mide accuracy
├── 04_few_shot_improvement.ipynb    ← Few-shot + CoT, compara vs baseline
├── 05_evaluation.ipynb              ← Métricas consolidadas, análisis de errores
└── 06_demo_gradio.ipynb             ← (Opcional) Interfaz interactiva
```

Cada notebook:
1. Es autocontenido (ejecutable de inicio a fin en Colab)
2. Instala dependencias al inicio
3. Carga el modelo una sola vez
4. Documenta resultados con visualizaciones
5. Guarda outputs para el siguiente notebook

## 5. Flujo de datos

```
corpus.json
    │
    ├──▶ 01_corpus_exploration ──▶ Estadísticas, validación
    │
    ├──▶ 02_baseline_generation
    │         │
    │         ├── Selecciona N temas aleatorios
    │         ├── Genera ejercicio por tema (zero-shot)
    │         ├── Guarda en results/generation_baseline.json
    │         └── Métricas: correctitud manual (columna para anotar)
    │
    ├──▶ 03_baseline_detection
    │         │
    │         ├── Toma pares (problema, error) del corpus
    │         ├── Pide al modelo detectar el error
    │         ├── Compara predicción vs ground truth
    │         └── Guarda en results/detection_baseline.json
    │
    ├──▶ 04_few_shot_improvement
    │         │
    │         ├── Mismos inputs que 02 y 03
    │         ├── Agrega 3-5 examples del corpus al prompt
    │         ├── Agrega chain-of-thought ("Piensa paso a paso")
    │         └── Compara métricas vs baseline
    │
    └──▶ 05_evaluation
              │
              ├── Consolida resultados de 02, 03, 04
              ├── Tablas comparativas
              ├── Análisis de errores del modelo
              └── Conclusiones y limitaciones
```

## 6. Prompting strategy

### Zero-shot (baseline)

```
[system] Eres un profesor de matemáticas de bachillerato...
[user] Genera un ejercicio de dificultad básica sobre valor absoluto.
```

### Few-shot (mejora)

```
[system] Eres un profesor de matemáticas de bachillerato...
[user] Aquí hay ejemplos de ejercicios bien formulados:

Ejemplo 1:
## Enunciado
Resuelve |x + 2| = 7
## Procedimiento
...
## Respuesta
x = 5, x = -9

Ejemplo 2:
...

Ahora genera un ejercicio nuevo de dificultad intermedia sobre valor absoluto.
```

### Chain-of-thought (mejora para detección)

```
[system] Eres un profesor revisando tarea. Analiza paso a paso antes de dar tu veredicto.
[user] Problema: Resuelve |2x - 1| > 3
Respuesta del estudiante: x > 2

Analiza paso a paso:
1. ¿Cuál es el procedimiento correcto?
2. ¿Qué hizo el estudiante?
3. ¿Dónde está el error (si lo hay)?
4. ¿Cuál es la respuesta correcta?
```

## 7. Configuración del modelo

```python
GENERATION_CONFIG = {
    "max_new_tokens": 512,
    "temperature": 0.7,      # Variedad en generación de ejercicios
    "top_p": 0.9,
    "repetition_penalty": 1.1,
}

DETECTION_CONFIG = {
    "max_new_tokens": 256,
    "temperature": 0.1,      # Determinista para detección
    "top_p": 0.95,
}

EXPLANATION_CONFIG = {
    "max_new_tokens": 512,
    "temperature": 0.3,      # Algo de variedad pero consistente
    "top_p": 0.9,
}
```

## 8. Riesgos y mitigaciones

| Riesgo | Impacto | Mitigación |
|--------|---------|------------|
| Modelo genera ejercicios con errores matemáticos | Alto | Evaluación manual obligatoria; el sistema es apoyo, no reemplazo |
| Modelo no detecta errores sutiles | Medio | Limitar alcance a errores "básicos"; documentar tasa de fallo |
| Respuestas en inglés o mixtas | Bajo | System prompt explícito en español; few-shot en español |
| Modelo alucina procedimientos | Alto | Chain-of-thought para forzar razonamiento explícito |
| Corpus demasiado pequeño para few-shot variado | Medio | Priorizar calidad sobre cantidad; 10 ejercicios por tema mínimo |
| Colab desconecta durante evaluación larga | Bajo | Guardar resultados incrementalmente; batches pequeños |

## 9. Criterios de éxito

El proyecto se considera exitoso si:

1. **Generación**: ≥70% de ejercicios generados son matemáticamente correctos y adecuados al nivel
2. **Detección**: ≥80% accuracy en identificar si hay error o no en respuestas del corpus
3. **Reproducibilidad**: Cualquier persona puede ejecutar los notebooks en Colab y obtener resultados similares
4. **Documentación**: Cada decisión, resultado y limitación está documentada en los notebooks
