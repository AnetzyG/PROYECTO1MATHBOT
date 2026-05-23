# Requerimientos — MathBot

## Objetivo

Construir un asistente educativo basado en LLMs que apoye a docentes de matemáticas de bachillerato (ENP/CCH, UNAM) en:
1. Generación automática de ejercicios sobre números reales
2. Detección de errores comunes en procedimientos matemáticos
3. Explicación de procedimientos paso a paso

## Infraestructura

- **Hardware**: Google Colab gratuito (GPU T4, ~15 GB VRAM)
- **Framework**: HuggingFace `transformers` + PyTorch
- **Modelos**: Descargados de HuggingFace Hub, ejecutados localmente en Colab
- **Sin APIs externas**: Todo corre en la GPU de Colab

## Modelo base

- **Candidato principal**: `Qwen/Qwen2.5-1.5B-Instruct`
  - Buen manejo del español
  - Cabe en T4 sin cuantización (~3 GB VRAM)
  - Sigue instrucciones de sistema (chat template)
  - Mismo modelo usado en el curso del profesor
- **Alternativas**: `Qwen/Qwen2.5-0.5B-Instruct` (si hay problemas de memoria), `microsoft/Phi-3-mini-4k-instruct` (3.8B, requiere 4-bit)

## Funcionalidades

### F1: Generación de ejercicios

- **Input**: Tema seleccionado por el docente (operaciones con reales, intervalos, desigualdades, valor absoluto, recta numérica)
- **Output**: Ejercicio con enunciado + solución correcta
- **Método**: Prompting estructurado con system prompt que define el rol de generador de ejercicios
- **Niveles de dificultad**: Básico, intermedio, avanzado
- **Formato de salida**: Enunciado, procedimiento, respuesta

### F2: Detección de errores

- **Input**: Problema matemático + respuesta/procedimiento del estudiante
- **Output**: Identificación del error + explicación de por qué es incorrecto + solución correcta
- **Tipos de errores a detectar**:
  - Errores de signo
  - Errores aritméticos
  - Errores en intervalos (abierto/cerrado)
  - Omisión de soluciones (valor absoluto)
  - Errores en desigualdades (no invertir signo al multiplicar por negativo)
- **Método**: Prompt con few-shot examples de errores comunes y sus correcciones

### F3: Explicación de procedimientos

- **Input**: Ejercicio o concepto matemático
- **Output**: Explicación paso a paso en español, nivel bachillerato
- **Método**: Prompting con instrucciones de nivel pedagógico

## Corpus

### Datos necesarios

- Ejercicios de álgebra de bachillerato (números reales)
- Pares (problema, solución correcta)
- Pares (problema, respuesta incorrecta, tipo de error)
- Material de ENP Matemáticas IV y CCH Matemáticas I/II

### Formato

Cada entrada del corpus:
```json
{
  "tema": "valor_absoluto",
  "problema": "Resuelve |2x - 3| = 5",
  "solucion": "x = 4, x = -1",
  "procedimiento": "2x-3=5 → x=4; 2x-3=-5 → x=-1",
  "errores_comunes": [
    {"respuesta": "x = 4", "tipo": "omision_solucion", "explicacion": "Falta considerar el caso negativo"}
  ],
  "dificultad": "basico",
  "nivel": "ENP_mat4"
}
```

### Fuentes

- Construcción manual a partir de libros de texto de ENP/CCH
- Errores recopilados de experiencia docente
- Ejercicios tipo de exámenes

## Pipeline

```
Docente selecciona tarea
        │
        ├── Generar ejercicio → [tema + dificultad] → Prompt → LLM → Ejercicio + solución
        │
        ├── Detectar error → [problema + respuesta estudiante] → Prompt → LLM → Error + corrección
        │
        └── Explicar → [concepto/ejercicio] → Prompt → LLM → Explicación paso a paso
```

## Evaluación

### Métricas para generación de ejercicios
- Correctitud matemática (evaluación manual por experto)
- Coherencia del enunciado (¿tiene sentido? ¿es resoluble?)
- Adecuación al nivel (¿corresponde a bachillerato?)
- Diversidad (¿genera ejercicios variados o se repite?)

### Métricas para detección de errores
- Accuracy: ¿identifica correctamente el error?
- Tasa de detección por tipo de error
- Calidad de la explicación (evaluación manual)

### Baseline
- Modelo preentrenado sin fine-tuning, solo con prompting
- Se espera 70-85% de correctitud en generación
- Se espera 80-90% en detección de errores básicos

### Meta post-baseline
- Mejorar con prompts más estructurados (few-shot, chain-of-thought)
- Evaluar si RAG con el corpus mejora la calidad
- Documentar limitaciones (errores de razonamiento, alucinaciones)

## Fases de desarrollo

| Fase | Entregable | Dependencia |
|------|-----------|-------------|
| 1. Corpus | 50-100 ejercicios etiquetados con errores comunes | — |
| 2. Baseline | Notebook con generación y detección via prompting directo | Corpus |
| 3. Prompts estructurados | Few-shot + chain-of-thought, medir mejora vs baseline | Baseline |
| 4. RAG (opcional) | Retrieval del corpus para contextualizar respuestas | Corpus + Baseline |
| 5. Evaluación | Métricas cuantitativas + evaluación cualitativa por experto | Todas |
| 6. Interfaz (opcional) | Demo con Gradio para interacción docente | Baseline |

## Restricciones

- El modelo debe correr en Colab gratuito (T4, 15 GB VRAM)
- Las respuestas deben ser en español
- El sistema es de apoyo al docente, no reemplaza la evaluación humana
- No se requiere fine-tuning obligatorio — si el prompting es suficiente, se documenta
- Reproducibilidad: cada notebook debe ser ejecutable de inicio a fin en Colab

## Temas matemáticos (alcance)

1. Números reales: naturales, enteros, racionales, irracionales
2. Operaciones: suma, resta, multiplicación, división, potencias, raíces
3. Intervalos: abiertos, cerrados, semiabiertos, representación en recta
4. Desigualdades: lineales, con valor absoluto
5. Valor absoluto: ecuaciones y desigualdades
6. Representación en recta numérica
