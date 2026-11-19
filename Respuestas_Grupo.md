# Respuestas de Discusión en Grupo — API de Inscripciones

## Paso 1

> **Pregunta:** ¿Qué recurso relaciona a Estudiante con Curso? ¿Dónde se guardan los cupos y dónde los cursos aprobados?

### Respuesta

El recurso que relaciona a `Estudiante` con `Curso` es **Inscripción**, los cupos se guardan en **Curso** y los cursos aprobados se guardan en **Estudiante**.

#### 1. ¿Qué recurso relaciona a Estudiante con Curso?
* **El recurso `Inscripción`:** Actúa como la entidad intermedia (lo que en bases de datos sería una tabla de unión para una relación muchos-a-muchos).
* Un estudiante puede inscribir muchos cursos y un curso tiene muchos estudiantes inscritos; por eso `Inscripción` guarda el `estudiante_id`, el `curso_codigo` y el `estado` de ese vínculo.

#### 2. ¿Dónde se guardan los cupos?
* **En el recurso `Curso`:** Es un atributo propio de cada materia (en el código se maneja como `curso["cupos"]`).
* Además, los **cupos disponibles** no se guardan como un dato fijo, sino que se calculan dinámicamente restando las inscripciones activas a los cupos totales del curso (`curso["cupos"] - len(datos.activas_del_curso(...))`).

#### 3. ¿Dónde se guardan los cursos aprobados?
* **En el recurso `Estudiante`:** Es un atributo propio del historial académico de cada alumno (aparece como `estudiante["aprobados"]`, una lista con los códigos de las materias que ya cursó y aprobó, necesaria para validar los `prerrequisitos` del curso).

---
