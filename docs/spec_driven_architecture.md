# Arquitectura Guiada por Especificaciones (Spec-Driven Architecture) en ColabEdu

## 1. El Concepto Base: Separación entre Plataforma y Legislación

ColabEdu emplea un patrón de diseño conocido como **Arquitectura Guiada por Especificaciones (Spec-Driven Architecture)**. El objetivo principal de este modelo es **desacoplar completamente el código fuente de la plataforma (Java, Flutter) de las normativas y metodologías educativas** de cada país o institución.

En este modelo, el repositorio central `ce-specs` actúa como el **ADN** de la plataforma. Los archivos de texto estructurado (YAML) almacenados aquí contienen la taxonomía, las rúbricas y las directivas globales. Cuando se añade o modifica un archivo YAML, la plataforma entera "muta" y se adapta automáticamente en tiempo de ejecución sin necesidad de recompilar código o lanzar nuevas versiones de software.

## 2. Impacto en el Ciclo de Vida de la Plataforma

Una única especificación en `ce-specs` actúa como una "fuente de la verdad" que orquesta y moldea el comportamiento de todos los flujos de trabajo (workflows) de la plataforma:

### A. Course Creation Workflow (Creación de Aulas)
*   **Comportamiento**: El sistema consulta el catálogo de especificaciones disponibles. El profesor elige la configuración base (ej. España -> LOMLOE -> 1º Bachillerato -> Lengua Castellana).
*   **Anclaje**: El aula virtual queda anclada permanentemente a un identificador único (ej. `es.c0.lomloe.lcl.bach.1.v1`).

### B. Exercise Creation Workflow (Creación de Ejercicios)
*   **Comportamiento**: Al crear un ejercicio dentro de un curso, el *wizard* inteligente lee el *Spec ID* del aula y se autoconfigura:
    1. **Filtro de Formatos**: Solo muestra los formatos de ejercicio y "Personas" de IA que tienen sentido pedagógico en esa normativa (ej. mostrará "Simulacro EBAU" para LOMLOE, pero ocultará "Examen Oral Individual" propio del Bachillerato Internacional).
    2. **Filtro de Criterios**: Restringe la lista de criterios de evaluación seleccionables estricta y únicamente a los de la asignatura y curso correspondientes. La IA puede pre-seleccionar los correctos leyendo el enunciado del profesor.

### C. Submission Workflow (Entrega del Alumno)
*   **Comportamiento**: La interfaz del estudiante se adapta a lo que la especificación requiere. Si un criterio de la rúbrica exige "expresión oral fluida", el flujo de entrega habilitará y exigirá automáticamente la grabación de audio.

### D. Grading V2 Workflow (Corrección Automatizada)
*   **Comportamiento**: El motor de IA (*Grading V2*) utiliza la especificación para construir su marco mental. En lugar de un *prompt* genérico, el sistema inyecta **dinámicamente** solo los criterios específicos que el ejercicio evalúa, junto con las directivas globales aplicables (ej. penalizaciones ortográficas).
*   **Ventaja**: Garantiza una corrección hiper-precisa, alineada 100% con la legalidad, reduciendo drásticamente el coste y el tiempo de procesamiento al enviar al LLM solo la información estrictamente necesaria.

### E. Reporting Workflow (Boletines y Analítica)
*   **Comportamiento**: Las rúbricas (C0) contienen referencias cruzadas a una taxonomía superior (ej. Competencias Clave). El motor de *reporting* utiliza estos metadatos para consolidar las notas de ejercicios individuales en informes agregados.
*   **Ventaja**: Traduce las puntuaciones locales al lenguaje oficial del estándar (ej. calcular la media sobre 10 para LOMLOE, o calcular una "Banda" del 1 al 7 para el Bachillerato Internacional).

## 3. Beneficios Estratégicos
1. **Escalabilidad Global Instantánea**: Expandir ColabEdu a un nuevo país (ej. México, SEP) o añadir un currículo privado (ej. Método Montessori) es una tarea de **entrada de datos**, no de ingeniería de software. Basta con subir los nuevos YAMLs.
2. **Resiliencia ante Cambios Legales**: Si el Ministerio de Educación cambia una ley (ej. de LOMCE a LOMLOE), los ingenieros no tienen que reprogramar la lógica de negocio. Se introduce una nueva versión de la especificación y los profesores migran sus cursos.
3. **Estandarización de la IA**: Evita el comportamiento errático de los Modelos de Lenguaje. Al inyectar fragmentos exactos y oficiales de la rúbrica, la IA actúa como un evaluador experto constreñido por el rigor legal y académico del país del usuario.

## 4. Interoperabilidad Universal de Formatos (El caso MCQ)
La separación entre la **Definición del Ejercicio (C2)** y el **Mecanismo de Entrega (SubmissionProvider)** genera un ecosistema de interoperabilidad universal. Tomando como ejemplo los ejercicios de Opción Múltiple (`mcq`):

1. **Creation / Preview Workflow**: Un ejercicio `mcq` puede originarse desde un archivo QTI importado, extraerse de un LMS (ej. Canvas), o diseñarse manualmente en nuestro *Wizard*. Independientemente del origen, el sistema lo normaliza al esquema YAML C2. Durante el *Preview*, el frontend siempre sabrá renderizar un `radio_button_group_widget` leyendo esta fuente unificada.
2. **Submission Workflow**: El alumno puede contestar el test directamente en nuestra plataforma (generando un payload limpio con la opción elegida, ej. `"C"`) o haberlo contestado en Canvas (donde importamos sus respuestas vía `CanvasCsvSubmissionProvider` como texto crudo).
3. **Grading Workflow**: Al estar el formato estandarizado en la capa C2, el `UniversalAssessmentService` evalúa con la misma precisión: si el alumno usó nuestra UI, realiza una corrección matemática determinista. Si la respuesta viene como texto extraído de un CSV de Canvas, el sistema cruza ese texto con el YAML para determinar el acierto o lo delega a la IA para evaluación de "Respuesta Corta". La rúbrica C0 se aplica igual.
4. **Feedback & Reporting Workflow**: Como todos los orígenes confluyen en un mismo esquema de datos, el motor de reporting genera las mismas métricas analíticas y boletines comparativos ("Apples to Apples"), independientemente de si el ejercicio nació en Canvas, Moodle o ColabEdu.
