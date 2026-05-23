# MathBot: Generación de ejercicios y detección de errores en números reales usando LLMs

**Proyecto I**  
Facultad de Ciencias, UNAM · Semestre 2026-2

---

# Autora

**Anetzy Fernanda García Compean**  
Matemáticas Aplicadas  

---

# Descripción

En este proyecto implementaremos un sistema de asistencia educativa basado en modelos de lenguaje (LLMs) orientado al apoyo de docentes de matemáticas de nivel bachillerato para la generación de ejercicios y la detección de errores sencillos relacionados con el tema de números reales.

El sistema está diseñado específicamente para:

- Matemáticas IV (ENP)
- Matemáticas I y II (CCH)

A partir de los prompts estructurados y modelos preentrenados, el proyecto buscará generar ejercicios, ejemplos y retroalimentación básica para apoyar la preparación de clases y reforzar el aprendizaje de los estudiantes.

El objetivo principal del proyecto es explorar el uso de modelos de lenguaje en contextos educativos mediante:

- generación automática de ejercicios,
- question answering,
- explicación de procedimientos matemáticos,
- detección de errores sencillos.

---

# Objetivo

Construir un pipeline reproducible de procesamiento de lenguaje natural capaz de generar ejercicios sobre números reales y detectar errores básicos en procedimientos matemáticos de nivel bachillerato.

La tarea principal se modela como:

$$\[
f : X \rightarrow Y
\]$$

donde:

- $$ \(X\) $$ representa problemas matemáticos y respuestas del estudiante.
- $$ \(Y\) $$ representa:
  - explicaciones,
  - ejercicios generados,
  - detección de errores.

El sistema busca aproximar:
$$
\[
\hat{y} = \arg\max P(y \mid x;\theta)
\]
$$
utilizando modelos de lenguaje preentrenados.

---

# Problema educativo

El sistema educativo actual exige que el docente adapte la enseñanza a distintos ritmos de aprendizaje dentro del aula. En la UNAM, los grupos de matemáticas suelen tener entre 20 y 25 estudiantes, lo que dificulta proporcionar atención individualizada de manera eficiente.

Además, los docentes deben:

- diseñar ejercicios,
- generar ejemplos contextualizados,
- evaluar procedimientos,
- proporcionar retroalimentación,
- planear clases completas en tiempos reducidos.

Aunque existen herramientas digitales y modelos de inteligencia artificial, muchas de ellas no están adaptadas al contexto específico de matemáticas de bachillerato en español.

---

# Arquitectura

El proyecto utiliza una arquitectura **decoder-only** tipo GPT, especializada en tareas generativas y de question answering.

Pipeline general:

```text
Tema → Prompt → LLM → Generación de ejercicios → Explicación → Detección de errores
```

La salida del modelo se representa mediante:
$$
\[
y = \text{LLM}(x)
\]
$$
donde:

- $$ \(x\) $$ corresponde al prompt ingresado por el docente,
- $$ \(y\) $$ corresponde a ejercicios, ejemplos o retroalimentación generada.

Los modelos decoder-only fueron seleccionados con base en:

- facilidad de interacción conversacional,
- capacidad de generación de texto,
- adecuación para generación de explicaciones matemáticas.

---

# Corpus

El corpus será construido manualmente utilizando:

- ejercicios de álgebra de bachillerato,
- ejemplos sobre números reales,
- material educativo en español,
- errores comunes cometidos por estudiantes.

Cada ejercicio se representa como:
$$
\[
x_i = (p_i, s_i, e_i)
\]
$$
donde:

- $$ \(p_i\) $$: problema matemático,
- $$ \(s_i\) $$: solución correcta,
- $$ \(e_i\) $$: error común.

El dominio del proyecto corresponde a:

> Educación matemática en español para nivel bachillerato.

---

# Preprocesamiento

El pipeline de preparación de datos contempla:

- normalización de texto,
- estructuración de ejercicios,
- identificación de errores frecuentes,
- limpieza de inconsistencias.

---

# Baseline

Como baseline inicial se evaluará:

- un modelo preentrenado sin fine-tuning,
- generación directa mediante prompting,
- detección básica de errores matemáticos.

Se espera un desempeño moderado capaz de:

- generar ejercicios coherentes,
- explicar procedimientos sencillos,
- detectar entre 80% y 90% de errores básicos.

Sin embargo, reconocemos las limitaciones como:

- errores de razonamiento,
- explicaciones inconsistentes,
- respuestas incorrectas,
- dificultad en problemas más complejos.

El objetivo del proyecto es mejorar la calidad de las respuestas mediante prompts estructurados y refinamiento del pipeline.

---

# Generación de ejercicios

El sistema será capaz de generar ejercicios relacionados con:

- operaciones con números reales,
- intervalos,
- desigualdades,
- valor absoluto,
- representación en recta numérica.

Sea:
$$
\[
T = \{t_1, t_2, ..., t_n\}
\]
$$
el conjunto de temas matemáticos disponibles.

El sistema generará ejercicios condicionados al tema seleccionado:
$$
\[
P(e \mid t)
\]
$$
donde:

- $$ \(t\) $$ representa el tema,
- $$ \(e\) $$ representa el ejercicio generado.

---

# Detección de errores

Además de generar ejercicios, el sistema analizará respuestas ingresadas por el usuario para detectar errores sencillos.

Sea:
$$
\[
R = \{r_1, r_2, ..., r_n\}
\]
$$
el conjunto de respuestas estudiantiles.

El sistema buscará identificar:

- errores de signo,
- errores aritméticos,
- errores en intervalos,
- omisión de soluciones,
- errores en valor absoluto.

---

# Tecnologías utilizadas

| Área | Herramientas |
|---|---|
| NLP | HuggingFace Transformers |
| Deep Learning | PyTorch |
| Procesamiento de datos | pandas |
| Experimentación | Jupyter Notebook |
| Control de versiones | GitHub |
| Hardware | Google Colab |

---

# Estructura del repositorio

```text
MATHBOT_PROJECT/
│
├── README.md
│
├── notebooks/
│   ├── 01_dataset_exploration.ipynb
│   ├── 02_preprocessing.ipynb
│   ├── 03_baseline.ipynb
│   └── 04_error_detection.ipynb
│
├── data/
│   ├── raw/
│   └── processed/
│
├── models/
│
├── src/
│
├── docs/
│
└── results/
```

---

# Roadmap

| Fase | Estado |
|---|---|
| Planeación | ✅ |
| Revisión bibliográfica | 🟡 |
| Construcción del corpus | 🟡 |
| Preprocesamiento | ⬜ |
| Baseline | ⬜ |
| Generación de ejercicios | ⬜ |
| Detección de errores | ⬜ |
| Evaluación | ⬜ |
| Interfaz | ⬜ |
| Deployment | ⬜ |

---

# Motivación

La enseñanza de las matemáticas representa uno de los mayores desafíos dentro del nivel medio superior debido a la diversidad de ritmos de aprendizaje y la necesidad de proporcionar retroalimentación constante.

En este contexto, los modelos de lenguaje pueden utilizarse para:

- apoyar la preparación de clases,
- generar ejercicios automáticamente,
- proporcionar ejemplos adicionales,
- detectar errores frecuentes,
- reforzar el aprendizaje autónomo.

Con este proyecto buscamos explorar el potencial de los LLMs como herramientas de apoyo educativo en español, particularmente dentro del contexto de la educación matemática en bachillerato.

---

# Papers y referencias principales
- Sebastian Schorcht1 et Al.
  No one size fits all: a study of prompt techniques and large language modelstoenhance AI’smathematics educational quality.
  2026
- Wensheng Gan, et Al.
  Large Language Models in Education: Vision and Opportunities.
  2023
- Janice Ahn, et Al.
  Large Language Models for Mathematical Reasoning: Progresses and Challenges.
  2024
- Hanyi Xu, et Al.
  Large Language Models for Education: A Survey.
  2024
---

# Reproducibilidad

Todos los experimentos están diseñados para ejecutarse en hardware accesible:

- Google Colab
- Kaggle Free Tier *pendiente

Cada notebook documentará:

- dependencias,
- prompts utilizados,
- configuración,
- ejemplos de salida,
- observaciones experimentales.

---

# Licencia

Este proyecto tiene fines académicos y educativos.

Los modelos, datasets y bibliotecas utilizadas mantienen las licencias originales de sus respectivos autores.

---
