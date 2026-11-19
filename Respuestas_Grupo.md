# Respuestas de Discusi�n en Grupo � API de Inscripciones

## Paso 1

> **Pregunta:** �Qu� recurso relaciona a Estudiante con Curso? �D�nde se guardan los cupos y d�nde los cursos aprobados?

### Respuesta

El recurso que relaciona a `Estudiante` con `Curso` es **Inscripci�n**, los cupos se guardan en **Curso** y los cursos aprobados se guardan en **Estudiante**.

#### 1. �Qu� recurso relaciona a Estudiante con Curso?
* **El recurso `Inscripci�n`:** Act�a como la entidad intermedia (lo que en bases de datos ser�a una tabla de uni�n para una relaci�n muchos-a-muchos).
* Un estudiante puede inscribir muchos cursos y un curso tiene muchos estudiantes inscritos; por eso `Inscripci�n` guarda el `estudiante_id`, el `curso_codigo` y el `estado` de ese v�nculo.

#### 2. �D�nde se guardan los cupos?
* **En el recurso `Curso`:** Es un atributo propio de cada materia (en el c�digo se maneja como `curso["cupos"]`).
* Adem�s, los **cupos disponibles** no se guardan como un dato fijo, sino que se calculan din�micamente restando las inscripciones activas a los cupos totales del curso (`curso["cupos"] - len(datos.activas_del_curso(...))`).

#### 3. �D�nde se guardan los cursos aprobados?
* **En el recurso `Estudiante`:** Es un atributo propio del historial acad�mico de cada alumno (aparece como `estudiante["aprobados"]`, una lista con los c�digos de las materias que ya curs� y aprob�, necesaria para validar los `prerrequisitos` del curso).

---

## Paso 2

> **Pregunta:** ¿Por qué cancelar se hace con `PATCH` y no con `DELETE`? ¿Qué se pierde si se borra la inscripción?

### Respuesta

En sistemas reales (como universidades, bancos o tiendas), casi nunca se usa `DELETE` para cancelar trámites.

* **¿Por qué `PATCH`?** Porque la inscripción sigue existiendo como un registro histórico en la base de datos; lo único que cambia es una propiedad suya (`estado`: pasa de `"ACTIVA"` a `"CANCELADA"`). A esto en ingeniería de software se le llama **borrado lógico (*Soft Delete*)**.
* **¿Qué se pierde si se usa `DELETE`?**
  1. **Trazabilidad y auditoría:** Si se borra el registro físico, desaparece la evidencia de que el estudiante alguna vez estuvo inscrito, cuándo se inscribió y cuándo canceló.
  2. **Historial académico:** La universidad no podría saber cuántas veces un estudiante ha cancelado una materia (un dato clave para reglamentos estudiantiles o bloqueos académicos).
  3. **Estadísticas de demanda:** Se perderían métricas sobre qué cursos tienen mayor tasa de deserción o cancelación.

---

## Paso 3

> **Pregunta:** ¿Por qué no se borra una inscripción cancelada? ¿Por qué un duplicado responde `409` y no `400`?

### Respuesta

* **¿Por qué no se borra una inscripción cancelada?**
  Para mantener el historial y la auditoría académica (saber qué materias inscribió y retiró un estudiante en el semestre), además de que cambiar su estado a `CANCELADA` ya libera el cupo automáticamente sin destruir la trazabilidad.

* **¿Por qué un duplicado responde `409` y no `400`?**
  Porque **`400 Bad Request`** significa que la petición está mal formada (sintaxis rota o datos corruptos que el servidor ni siquiera puede procesar). En un duplicado, el JSON es perfectamente válido, pero la operación entra en **conflicto (`409 Conflict`)** con el estado actual de los datos en el servidor (ya existe una inscripción activa igual).

---

## Paso 4

> **Pregunta:** Si otro equipo solo tuviera openapi.json, ¿podría consumir la API sin hacerles
preguntas? ¿Qué información le faltaría?

### Respuesta

Cualquier equipo podría consumir la API sin problemas teniendo el openapi.json, ya que contiene toda la información que un consumidor necesita saber, como ejemplos de datos, esquemas, para que se usa cada endpoint y las respuestas que pueden recibir. Todo esto mientras no da detalles acerca de la implementación del endpoint usado. 

---