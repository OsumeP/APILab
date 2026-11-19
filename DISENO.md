# Diseño de la API de Inscripciones

Grupo: Laura Natalia Duarte y Juan David Patarroyo.

## Paso 1 — Recursos

Un recurso es un **sustantivo** que la API expone con su propia URI.
La fila de *Curso* ya está hecha como ejemplo.

| Recurso     | URI de la colección | Atributos clave                        | Se relaciona con |
|-------------|---------------------|----------------------------------------|------------------|
| Curso       | `/cursos`           | codigo, nombre, cupos, prerrequisitos  | Inscripción      |
| Estudiante  | `/estudiantes`      |                                        | Inscripción      |
| Inscripción | `/inscripciones`    |                                        | Estudiante, Curso|

**Pregunta:** "inscribir" es un verbo. ¿Por qué en REST lo modelamos como el
recurso *Inscripción* y no como una URI `/inscribir`?

> Respuesta:Porque la arquitectura REST se orienta a recursos (sustantivos) y no a acciones (verbos). 
Las acciones ya están definidas por los métodos HTTP estándar (POST, GET, PATCH, DELETE). 
Modelarlo como /inscripciones permite gestionar todo el ciclo de vida del registro: 
crearlo (POST /inscripciones), consultarlo (GET /inscripciones/{id}) y 
actualizar su estado (PATCH /inscripciones/{id}), 
lo cual sería confuso o inconsistente con una URI basada en un verbo como /inscribir.

## Paso 2 — Contrato: los 5 endpoints

Completa el código de éxito y los errores posibles de cada endpoint.
Pistas: crear ? `201`; no existe ? `404`; choca con una regla de negocio ? `409`.

| # | Método | URI                                          | Cuerpo que recibe     | Éxito | Errores                                                     |
|---|--------|----------------------------------------------|-----------------------|-------|-------------------------------------------------------------|
| 1 | GET    | `/cursos`                                    | —                     | 200   | —                                                           | 
| 2 | POST   | `/inscripciones`                             | `InscripcionCrear`    | 201   |404 (estudiante o curso no existe)                           |
|   |        |                                              |                       |       |409 (sin cupos, falta prerrequisito o inscripción duplicada) |
|   |        |                                              |                       |       |422 (JSON inválido) devuelve FastAPI por defecto             |
| 3 | GET    | `/inscripciones/{inscripcion_id}`            | —                     | 200   |404 (la inscripción no existe)                               |
| 4 | GET    | `/estudiantes/{estudiante_id}/inscripciones` | —                     | 200   |404 (el estudiante no existe)                                |
| 5 | PATCH  | `/inscripciones/{inscripcion_id}`            | `CambioEstado`        | 200   |404 (la inscripción no existe)                               |
|   |        |                                              |                       |       |409 (ya estaba cancelada / transición de estado inválida)    |
|   |        |                                              |                       |       |422 (estado no permitido)                                    | 

## Paso 3 — Estados de una inscripción

```
   POST /inscripciones            PATCH {"estado": "CANCELADA"}
  ---------------------?  ACTIVA  ------------------------------?  CANCELADA
```

**Pregunta:** ¿Qué debe responder la API si alguien intenta cancelar una
inscripción que ya está CANCELADA? ¿Por qué?

> Respuesta:
 Debe responder con un código 409 Conflict.
¿Por qué? Porque la petición es sintácticamente válida y el recurso existe (no es un 400 ni un 404), 
pero la operación entra en conflicto con el estado actual del recurso y las reglas de negocio: 
el diagrama de estados establece que el flujo es unidireccional (ACTIVA --> CANCELADA), 
por lo que una inscripción cancelada no puede volver a cancelarse.