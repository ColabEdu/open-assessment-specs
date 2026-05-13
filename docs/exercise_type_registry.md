# Registro de ExerciseTypes (Capa C1)

Este documento es el catálogo oficial de los `ExerciseTypes` soportados por la arquitectura de ColabEdu. Los **ExerciseTypes** definen cómo se renderiza un ejercicio en el frontend (Data-Driven UI) y cómo viaja la información ("submission payload") hacia el backend para su evaluación.

Sirve como guía para desarrolladores, curadores de contenido y sistemas de IA al redactar especificaciones C2.

---

## 1. Texto Libre (`free_text`)

Diseñado para preguntas de respuesta abierta, ensayos y argumentaciones cortas.

- **Identificador (`type`)**: `free_text`
- **Widgets de UX (Submission)**:
  - `markdown_editor_widget`: Un editor de texto enriquecido para el alumno.
- **Configuración YAML Adicional**: No requiere parámetros extra obligatorios más allá del `prompt`.
- **Evaluación**: Subjetiva. Delegada a los LLMs basándose en la rúbrica C0 referenciada.
- **Submission Payload**: 
  El frontend enviará un bloque de texto Markdown como respuesta del usuario.

```yaml
# Ejemplo
- id: "q1_ensayo"
  type: "free_text"
  prompt: "Elabore un texto argumentativo sobre el tema."
  expectedOutputFormat: "markdown"
  maxScore: 3.0
  targetCompetencies:
    - "3.1"
```

---

## 2. Opción Múltiple (`mcq`)

Diseñado para evaluaciones deterministas tipo test (EBAU, PISA, INEE).

- **Identificador (`type`)**: `mcq`
- **Widgets de UX (Submission)**:
  - `radio_button_group_widget`: Renderiza las opciones proporcionadas.
- **Configuración YAML Adicional**:
  - `options` (Array): Lista de identificadores (`id`) y textos (`text`).
  - `correctOptionId` (String): El ID de la respuesta correcta.
- **Evaluación**: Determinista. Matemática directa en el backend (0 si falla, `maxScore` si acierta). El LLM no evalúa esta pregunta.
- **Submission Payload**:
  El frontend enviará únicamente el `id` de la opción seleccionada (ej. `"B"`).

```yaml
# Ejemplo
- id: "q2_test"
  type: "mcq"
  prompt: "¿Cuál es el propósito principal?"
  options:
    - id: "A"
      text: "Opción A"
    - id: "B"
      text: "Opción B (Correcta)"
  correctOptionId: "B"
  maxScore: 1.0
  targetCompetencies:
    - "2.1"
```

---

## 3. Actividad Guiada con Tutor de IA (`guided_activity`)

Diseñado para experiencias inmersivas donde el alumno debate o analiza un recurso junto a un agente conversacional antes de entregar su respuesta final.

- **Identificador (`type`)**: `guided_activity`
- **Widgets de UX (Submission)**:
  - `split_screen_layout`: Divide la pantalla en dos paneles.
  - `document_viewer_widget`: (Panel izquierdo) Visor nativo de PDF o Markdown interactivo.
  - `chatbot_companion_widget`: (Panel derecho) Interfaz de chat en tiempo real con el Gem configurado.
  - `markdown_editor_widget`: Formulario final para escribir la conclusión.
- **Configuración YAML Adicional**:
  - `documentUrl` (String): Ruta del archivo (ej. URL o GCS path) a renderizar en el visor.
  - `companionGem` (String): ID del Gem/Tutor a instanciar (el sistema debe tener este Gem registrado).
  - `expectedLabels` (Array): Tópicos clave que el Gem intentará extraer durante la conversación.
- **Evaluación**: Mixta. El Gem realiza supervisión proactiva, y el texto final se evalúa contra la rúbrica C0.
- **Submission Payload**:
  El frontend envía el texto conclusivo escrito por el alumno, y de manera silenciosa, el historial de interacciones con el Gem y los "labels" alcanzados.

```yaml
# Ejemplo
- id: "q3_actividad_guiada"
  type: "guided_activity"
  prompt: "Lee el documento y discute con el tutor sobre las implicaciones sociales."
  documentUrl: "gcs://colabedu-assets/pdf/recurso_lectura.pdf"
  companionGem: "science_tutor" # Debe ser un ID de Gem válido
  expectedLabels: ["sociedad", "etica"]
  expectedOutputFormat: "markdown"
  maxScore: 4.0
  targetCompetencies:
    - "3.1"
```

---

## 4. Entrega de Documentos (`document_submission`)

Diseñado para proyectos largos, código fuente o archivos ofimáticos (ej. hojas de cálculo, presentaciones).

- **Identificador (`type`)**: `document_submission`
- **Widgets de UX (Submission)**:
  - `file_picker_widget`: Componente nativo para examinar y subir archivos.
- **Configuración YAML Adicional**:
  - `allowedExtensions` (Array opcional): Lista de extensiones permitidas (ej. `["pdf", "docx"]`).
- **Evaluación**: El backend sube el archivo a un bucket de Storage (GCS), extrae el texto usando servicios de procesado (OCR/Parsers) y el LLM evalúa el texto resultante.
- **Submission Payload**:
  El frontend envía el archivo binario / multipart.

```yaml
# Ejemplo
- id: "q4_proyecto"
  type: "document_submission"
  prompt: "Sube tu investigación final en formato PDF."
  allowedExtensions: ["pdf"]
  maxScore: 5.0
  targetCompetencies:
    - "6.1"
```
