# Plan de Implementación — MathBot

## Convenciones

- Cada tarea produce un artefacto concreto (archivo, notebook, resultado)
- Las tareas están ordenadas por dependencia
- ✅ = completada, 🔲 = pendiente, 🚧 = en progreso

---

## Fase 1: Corpus (Semana 1)

### T1.1 — Definir estructura del corpus
- **Artefacto**: `data/corpus.json` (esqueleto con 5 ejercicios de ejemplo)
- **Detalle**: Crear el JSON con el schema definido en design.md. Incluir al menos 1 ejercicio por tema para validar la estructura.
- **Estado**: 🔲

### T1.2 — Poblar corpus: Valor absoluto
- **Artefacto**: 10-15 ejercicios de valor absoluto en `corpus.json`
- **Detalle**: Ecuaciones y desigualdades con valor absoluto. Incluir errores comunes (omisión de caso negativo, error de signo). Dificultades: básico, intermedio, avanzado.
- **Estado**: 🔲

### T1.3 — Poblar corpus: Intervalos y desigualdades
- **Artefacto**: 10-15 ejercicios en `corpus.json`
- **Detalle**: Desigualdades lineales, representación en intervalos, errores de notación (abierto/cerrado), error al multiplicar por negativo.
- **Estado**: 🔲

### T1.4 — Poblar corpus: Operaciones con reales
- **Artefacto**: 10-15 ejercicios en `corpus.json`
- **Detalle**: Fracciones, potencias, raíces, jerarquía de operaciones. Errores aritméticos comunes.
- **Estado**: 🔲

### T1.5 — Poblar corpus: Recta numérica y clasificación
- **Artefacto**: 10-15 ejercicios en `corpus.json`
- **Detalle**: Ubicar números en la recta, clasificar (natural, entero, racional, irracional). Errores de clasificación.
- **Estado**: 🔲

### T1.6 — Validar corpus
- **Artefacto**: Notebook `01_corpus_exploration.ipynb`
- **Detalle**: Cargar corpus, verificar schema, mostrar distribución por tema/dificultad/tipo de error, detectar inconsistencias.
- **Estado**: 🔲

---

## Fase 2: Baseline (Semana 2)

### T2.1 — Setup del modelo
- **Artefacto**: Celda reutilizable de carga de modelo en notebook
- **Detalle**: Cargar `Qwen/Qwen2.5-1.5B-Instruct` en Colab T4. Verificar que genera texto en español. Medir VRAM usada. Documentar versiones de dependencias.
- **Estado**: 🔲

### T2.2 — Baseline de generación (zero-shot)
- **Artefacto**: Notebook `02_baseline_generation.ipynb` + `results/generation_baseline.json`
- **Detalle**:
  - Para cada tema, generar 5 ejercicios con prompt zero-shot
  - Guardar outputs crudos
  - Columna vacía para evaluación manual (correctitud: sí/no/parcial)
  - Calcular tasa de generación exitosa (¿el modelo produce algo parseable?)
- **Estado**: 🔲

### T2.3 — Baseline de detección (zero-shot)
- **Artefacto**: Notebook `03_baseline_detection.ipynb` + `results/detection_baseline.json`
- **Detalle**:
  - Tomar todos los pares (problema, error_comun) del corpus
  - Pedir al modelo: "¿hay error? ¿cuál?"
  - Comparar vs ground truth
  - Métricas: accuracy, precision, recall por tipo de error
- **Estado**: 🔲

### T2.4 — Evaluar y documentar baseline
- **Artefacto**: Sección de resultados en cada notebook
- **Detalle**: Anotar manualmente correctitud de generación (mínimo 20 ejercicios). Calcular métricas de detección automáticamente. Documentar observaciones cualitativas (¿en qué falla? ¿en qué es bueno?).
- **Estado**: 🔲

---

## Fase 3: Mejora con prompting (Semana 3)

### T3.1 — Implementar few-shot para generación
- **Artefacto**: Notebook `04_few_shot_improvement.ipynb` (sección generación)
- **Detalle**:
  - Seleccionar 3 ejercicios del corpus como examples
  - Inyectar en el prompt antes de la instrucción
  - Generar mismos 5 ejercicios por tema
  - Comparar correctitud vs baseline
- **Estado**: 🔲

### T3.2 — Implementar chain-of-thought para detección
- **Artefacto**: Notebook `04_few_shot_improvement.ipynb` (sección detección)
- **Detalle**:
  - Agregar "Analiza paso a paso:" al prompt
  - Incluir 2-3 examples de análisis de errores
  - Evaluar mismos pares que el baseline
  - Comparar accuracy vs baseline
- **Estado**: 🔲

### T3.3 — Experimentar con temperature y parámetros
- **Artefacto**: Sección en notebook 04
- **Detalle**: Probar temperature 0.1 vs 0.5 vs 0.9 para generación. Probar top_p. Documentar efecto en calidad y diversidad.
- **Estado**: 🔲

### T3.4 — Comparativa consolidada
- **Artefacto**: Tabla en notebook 04
- **Detalle**: Tabla con columnas [método, accuracy_generación, accuracy_detección, observaciones]. Zero-shot vs few-shot vs CoT.
- **Estado**: 🔲

---

## Fase 4: RAG — opcional (Semana 4, si aplica)

### T4.1 — Evaluar necesidad de RAG
- **Artefacto**: Decisión documentada en notebook 04
- **Detalle**: Si few-shot ya alcanza ≥70% generación y ≥80% detección, RAG no es necesario. Si no, proceder con T4.2.
- **Estado**: 🔲

### T4.2 — Implementar RAG simple
- **Artefacto**: Notebook `04b_rag_experiment.ipynb`
- **Detalle**: sentence-transformers para embeddings del corpus + FAISS para retrieval + inyectar chunks relevantes en prompt. Seguir patrón del notebook del profesor (sesion2_rag_chatbot_joyeria).
- **Estado**: 🔲

---

## Fase 5: Evaluación final (Semana 4-5)

### T5.1 — Evaluación cuantitativa
- **Artefacto**: Notebook `05_evaluation.ipynb`
- **Detalle**:
  - Consolidar todos los resultados (baseline, few-shot, CoT, RAG si aplica)
  - Tablas comparativas con métricas
  - Gráficas de accuracy por tema y por tipo de error
  - Análisis de errores del modelo (¿en qué temas falla más?)
- **Estado**: 🔲

### T5.2 — Evaluación cualitativa
- **Artefacto**: Sección en notebook 05
- **Detalle**: Seleccionar 10 outputs representativos (5 buenos, 5 malos). Analizar por qué el modelo acierta o falla. Documentar limitaciones concretas.
- **Estado**: 🔲

### T5.3 — Documentar limitaciones y trabajo futuro
- **Artefacto**: Sección final en notebook 05
- **Detalle**: Lista de limitaciones observadas. Sugerencias de mejora (fine-tuning, modelo más grande, más corpus). Comparación con lo que prometía el README.
- **Estado**: 🔲

---

## Fase 6: Demo — opcional (Semana 5)

### T6.1 — Interfaz Gradio
- **Artefacto**: Notebook `06_demo_gradio.ipynb`
- **Detalle**: Interfaz simple con 3 tabs (Generar, Detectar, Explicar). Dropdown para tema y dificultad. Textbox para input del estudiante. Output formateado. Seguir patrón del notebook del profesor (sesion1_demo_presentacion_final).
- **Estado**: 🔲

---

## Resumen de entregables

| # | Archivo | Fase |
|---|---------|------|
| 1 | `data/corpus.json` | 1 |
| 2 | `01_corpus_exploration.ipynb` | 1 |
| 3 | `02_baseline_generation.ipynb` | 2 |
| 4 | `03_baseline_detection.ipynb` | 2 |
| 5 | `04_few_shot_improvement.ipynb` | 3 |
| 6 | `05_evaluation.ipynb` | 5 |
| 7 | `06_demo_gradio.ipynb` | 6 |
| 8 | `results/` (JSONs con outputs) | 2-5 |

## Orden de ejecución

```
T1.1 → T1.2, T1.3, T1.4, T1.5 (paralelas) → T1.6
                                                 │
                                                 ▼
                                    T2.1 → T2.2, T2.3 → T2.4
                                                          │
                                                          ▼
                                             T3.1, T3.2 → T3.3 → T3.4
                                                                   │
                                                                   ▼
                                                          T4.1 → (T4.2)
                                                                   │
                                                                   ▼
                                                     T5.1 → T5.2 → T5.3
                                                                     │
                                                                     ▼
                                                                   T6.1
```
