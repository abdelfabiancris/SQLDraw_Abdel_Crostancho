# TalentTrack --- documentación del modelo de datos

## 1. Objetivo

TalentTrack modela organizaciones, empleados, instructores, cursos,
ediciones, sesiones, matrículas, asistencia y certificados. El documento
describe el SQL entregado y distingue las estructuras presentes de los
ajustes recomendados.



## 2. Diccionario de datos

**PK** = clave primaria; **FK** = clave foránea; **UQ** = único; **NN**
= `NOT NULL`; **NULL** = permite nulos.

### `organizations`

  Atributo            Tipo                Nulabilidad   Clave / restricción
  ------------------- ------------------- ------------- ----------------------
  `organization_id`   `BIGINT UNSIGNED`   NN            PK, `AUTO_INCREMENT`
  `nit`               `VARCHAR(20)`       NN            UQ
  `legal_name`        `VARCHAR(100)`      NN            ---
  `status`            `BOOLEAN`           NULL          Default `'ACTIVE'`

**Observación:** `BOOLEAN` representa un valor booleano, no estados
textuales como `ACTIVE`. Cambiarlo por `ENUM('ACTIVE','INACTIVE')` o
usar `BOOLEAN NOT NULL DEFAULT TRUE`, según la intención.

### `employees`

  -------------------------------------------------------------------------------------------
  Atributo            Tipo                Nulabilidad       Clave / restricción
  ------------------- ------------------- ----------------- ---------------------------------
  `employee_id`       `BIGINT UNSIGNED`   NN                PK, `AUTO_INCREMENT`

  `organization_id`   `BIGINT UNSIGNED`   NN                FK →
                                                            `organizations.organization_id`

  `full_name`         `VARCHAR(100)`      NN                ---

  `email`             `VARCHAR(100)`      NN                ---
  -------------------------------------------------------------------------------------------

### `instructors`

  Atributo          Tipo                Nulabilidad   Clave / restricción
  ----------------- ------------------- ------------- ----------------------
  `instructor_id`   `BIGINT UNSIGNED`   NN            PK, `AUTO_INCREMENT`
  `full_name`       `VARCHAR(100)`      NN            ---
  `email`           `VARCHAR(100)`      NN            UQ

### `courses`

  Atributo                    Tipo                Nulabilidad   Clave / restricción
  --------------------------- ------------------- ------------- ----------------------
  `course_id`                 `BIGINT UNSIGNED`   NN            PK, `AUTO_INCREMENT`
  `code`                      `VARCHAR(30)`       NN            UQ
  `name`                      `VARCHAR(100)`      NN            ---
  `description`               `TEXT`              NULL          ---
  `expected_duration_hours`   `DECIMAL(6,2)`      NN            ---

### `course_editions`

  -----------------------------------------------------------------------------------------------------------------------
  Atributo          Tipo                                                  Nulabilidad       Clave / restricción
  ----------------- ----------------------------------------------------- ----------------- -----------------------------
  `edition_id`      `BIGINT UNSIGNED`                                     NN                PK, `AUTO_INCREMENT`

  `course_id`       `BIGINT UNSIGNED`                                     NN                FK → `courses.course_id`

  `instructor_id`   `BIGINT UNSIGNED`                                     NN                FK →
                                                                                            `instructors.instructor_id`

  `code`            `VARCHAR(40)`                                         NN                UQ

  `start_date`      `DATE`                                                NN                ---

  `end_date`        `DATE`                                                NN                ---

  `capacity`        `INT UNSIGNED`                                        NN                ---

  `status`          `ENUM('DRAFT','PUBLISHED','COMPLETED','CANCELLED')`   NN                Default `DRAFT`
  -----------------------------------------------------------------------------------------------------------------------

### `sessions`

  --------------------------------------------------------------------------------------
  Atributo          Tipo                Nulabilidad       Clave / restricción
  ----------------- ------------------- ----------------- ------------------------------
  `session_id`      `BIGINT UNSIGNED`   NN                PK, `AUTO_INCREMENT`

  `edition_id`      `BIGINT UNSIGNED`   NN                FK →
                                                          `course_editions.edition_id`

  `session_date`    `DATE`              NN                ---

  `start_time`      `TIME`              NN                ---

  `end_time`        `TIME`              NN                ---
  --------------------------------------------------------------------------------------

### `enrollments`

  -----------------------------------------------------------------------------------------------------------------
  Atributo            Tipo                                         Nulabilidad       Clave / restricción
  ------------------- -------------------------------------------- ----------------- ------------------------------
  `enrollment_id`     `BIGINT UNSIGNED`                            NN                PK, `AUTO_INCREMENT`; además
                                                                                     el SQL lo declara FK hacia
                                                                                     `certificates.enrollment_id`

  `employee_id`       `BIGINT UNSIGNED`                            NN                FK → `employees.employee_id`

  `edition_id`        `BIGINT UNSIGNED`                            NN                FK →
                                                                                     `course_editions.edition_id`

  `enrollment_date`   `DATETIME`                                   NN                Default `CURRENT_TIMESTAMP`

  `status`            `ENUM('ENROLLED','COMPLETED','CANCELLED')`   NN                Default `ENROLLED`
  -----------------------------------------------------------------------------------------------------------------

### `attendance`

  -----------------------------------------------------------------------------------------
  Atributo              Tipo                Nulabilidad       Clave / restricción
  --------------------- ------------------- ----------------- -----------------------------
  `attendance_id`       `BIGINT UNSIGNED`   NN                PK, `AUTO_INCREMENT`

  `enrollment_id`       `BIGINT UNSIGNED`   NN                FK →
                                                              `enrollments.enrollment_id`

  `session_id`          `BIGINT UNSIGNED`   NN                FK → `sessions.session_id`

  `edition_id`          `BIGINT UNSIGNED`   NN                Sin FK declarada en el SQL

  `attendance_status`   `BOOLEAN`           NN                ---

  `recorded_at`         `DATETIME`          NN                Default `CURRENT_TIMESTAMP`
  -----------------------------------------------------------------------------------------

**Observación:** `edition_id` puede derivarse desde
`sessions.edition_id`; mantener ambos permite inconsistencias si no se
comprueba que coincidan.

### `certificates`

  -----------------------------------------------------------------------------
  Atributo             Tipo                Nulabilidad       Clave /
                                                             restricción
  -------------------- ------------------- ----------------- ------------------
  `certificate_id`     `BIGINT UNSIGNED`   NN                PK,
                                                             `AUTO_INCREMENT`

  `enrollment_id`      `BIGINT UNSIGNED`   NN                UQ; en el SQL no
                                                             tiene FK hacia
                                                             `enrollments`

  `code`               `VARCHAR(60)`       NN                UQ

  `issue_date`         `DATE`              NN                ---

  `verification_url`   `VARCHAR(2048)`     NN                ---
  -----------------------------------------------------------------------------

## 3. Relaciones y cardinalidades

-   `organizations` 1:N `employees`.
-   `courses` 1:N `course_editions`.
-   `instructors` 1:N `course_editions`.
-   `course_editions` 1:N `sessions`.
-   `employees` N:M `course_editions`, resuelta mediante `enrollments`.
-   `enrollments` 1:N `attendance` y `sessions` 1:N `attendance`; el
    registro de asistencia enlaza una matrícula con una sesión.
-   `enrollments` y `certificates` parecen tener una relación 1:0..1 por
    el `UNIQUE` en `certificates.enrollment_id`, pero la FK está
    declarada en dirección inversa y debe corregirse.
-   Participación mínima: las FK declaradas son `NOT NULL`, por lo que
    cada fila hija debe tener el registro padre correspondiente. No se
    exige que cada curso tenga ediciones o que cada matrícula tenga
    asistencia.

## 4. Formas normales

### Primera forma normal (1FN)

Los atributos son valores individuales: nombre, correo, fecha, estado o
URL. Las matrículas y asistencias se registran en filas separadas, no
como listas dentro de una celda. Esto favorece 1FN.

### Segunda forma normal (2FN)

Las tablas usan PK simples, por lo que no hay dependencia parcial de una
clave compuesta dentro de esas tablas. La relación N:M entre empleados y
ediciones se representa con `enrollments`. Para impedir matrículas
duplicadas de un empleado en la misma edición, convendría añadir
`UNIQUE(employee_id, edition_id)` si esa es la regla del negocio.

### Tercera forma normal (3FN)

Los datos de curso, instructor, edición, empleado y organización están
separados en sus propias tablas, evitando repetir atributos descriptivos
en registros dependientes. Sin embargo, `attendance.edition_id` es
potencialmente redundante: la edición se determina a través de
`session_id` → `sessions.edition_id`. Para mantener 3FN y evitar
inconsistencias, se recomienda eliminar `edition_id` de `attendance`, o
implementar una restricción que garantice su correspondencia si existe
una razón válida para conservarlo.

## 5. Evaluación de cuarta forma normal (4FN)

La asistencia a sesiones y las matrículas son relaciones distintas y
deben mantenerse separadas. Si en una misma tabla de edición se
almacenaran listas independientes de empleados inscritos, sesiones e
instructores, se generarían combinaciones redundantes. El diseño usa
tablas separadas para estos conceptos, lo que ayuda a evitar ese
problema. La 4FN no se evalúa eligiendo un atributo aislado, sino
revisando dependencias multivaluadas independientes.

## 6. Reglas de integridad adicionales

-   `end_date >= start_date` para una edición.
-   `end_time > start_time` para una sesión, teniendo en cuenta sesiones
    que pudieran cruzar medianoche.
-   La capacidad de una edición no debe excederse.
-   La asistencia debe corresponder a una sesión de la misma edición en
    la que está matriculado el empleado.
-   Evitar dos matrículas activas del mismo empleado en una misma
    edición si la política no permite duplicados.
-   Emitir certificados solo para matrículas que cumplan los requisitos.
-   Corregir la dirección de la FK entre `certificates` y `enrollments`.
-   Cambiar `organizations.status` a un tipo coherente con los valores
    de estado deseados.

## 7. Fragmentos de datos de prueba

### `courses`

    course_id code     name
  ----------- -------- -------------------
            1 SEC-01   Seguridad digital

### `course_editions`

    edition_id   course_id   instructor_id code
  ------------ ----------- --------------- --------------
            10           1               5 SEC-01-2026A

### `enrollments`

    enrollment_id   employee_id   edition_id status
  --------------- ------------- ------------ ----------
              100            20           10 ENROLLED
              101            21           10 ENROLLED

Estos ejemplos muestran cómo una edición puede tener múltiples
matrículas. Los identificadores de ejemplo son ilustrativos.

## 8. Pregunta de sustentación

La duplicación puede producir anomalías de actualización, inserción y
eliminación: por ejemplo, repetir datos del curso en cada matrícula
obliga a corregir varias filas si cambia el nombre. Una lista de
identificadores dentro de una celda ---por ejemplo, `20,21,22` como
lista de empleados inscritos--- dificulta validar cada empleado con una
FK, buscar matrículas individuales y mantener atributos de cada
inscripción. Una fila por relación en `enrollments` mantiene cada
vínculo verificable y consultable.
