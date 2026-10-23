# HelpDesk --- documentación del modelo de datos

## 1. Objetivo

HelpDesk gestiona empresas, contactos, tickets, categorías, historial de
cambios, comentarios, técnicos y asignaciones. La documentación describe
el SQL recibido e indica los puntos que deben completarse para asegurar
que las restricciones reflejen las reglas del sistema.


## 2. Diccionario de datos

**PK** = clave primaria; **FK** = clave foránea; **UQ** = único; **NN**
= `NOT NULL`; **NULL** = permite nulos.

### `companies`

  --------------------------------------------------------------------------
  Atributo          Tipo                Nulabilidad       Clave /
                                                          restricción
  ----------------- ------------------- ----------------- ------------------
  `id`              `BIGINT UNSIGNED`   NN                PK,
                                                          `AUTO_INCREMENT`

  `legal_name`      `VARCHAR(100)`      NN                UQ

  `nit`             `BIGINT`            NN                UQ; valorar
                                                          `VARCHAR` para
                                                          conservar formato
                                                          y ceros iniciales

  `status`          `BOOLEAN`           NN                ---
  --------------------------------------------------------------------------

### `contacts`

  ---------------------------------------------------------------------------
  Atributo          Tipo                Nulabilidad       Clave / restricción
  ----------------- ------------------- ----------------- -------------------
  `id`              `BIGINT UNSIGNED`   NN                PK,
                                                          `AUTO_INCREMENT`

  `name`            `VARCHAR(100)`      NN                ---

  `email`           `VARCHAR(100)`      NN                UQ

  `is_active`       `BOOLEAN`           NN                ---

  `company_id`      `BIGINT`            NN                FK →
                                                          `companies.id`;
                                                          alinear tipo con
                                                          `BIGINT UNSIGNED`
  ---------------------------------------------------------------------------

### `tickets`

  ---------------------------------------------------------------------------
  Atributo          Tipo                Nulabilidad       Clave / restricción
  ----------------- ------------------- ----------------- -------------------
  `id`              `BIGINT UNSIGNED`   NN                PK,
                                                          `AUTO_INCREMENT`

  `subject`         `VARCHAR(100)`      NN                ---

  `description`     `VARCHAR(100)`      NN                ---; valorar `TEXT`
                                                          para descripciones
                                                          largas

  `start_date`      `DATETIME`          NN                ---

  `priority`        `ENUM('')`          NN                **Incompleto**;
                                                          definir valores
                                                          válidos

  `status`          `ENUM('')`          NN                **Incompleto**;
                                                          definir valores
                                                          válidos

  `contact_id`      `BIGINT`            NN                FK → `contacts.id`;
                                                          alinear tipo con
                                                          `BIGINT UNSIGNED`

  `category_id`     `BIGINT`            NN                FK →
                                                          `categories.id`;
                                                          alinear tipo con
                                                          `BIGINT UNSIGNED`
  ---------------------------------------------------------------------------

### `categories`

  Atributo        Tipo                Nulabilidad   Clave / restricción
  --------------- ------------------- ------------- ----------------------
  `id`            `BIGINT UNSIGNED`   NN            PK, `AUTO_INCREMENT`
  `name`          `VARCHAR(100)`      NN            ---; valorar UQ
  `description`   `VARCHAR(100)`      NN            ---

### `ticket_histories`

  -----------------------------------------------------------------------------
  Atributo            Tipo                Nulabilidad       Clave / restricción
  ------------------- ------------------- ----------------- -------------------
  `id`                `BIGINT UNSIGNED`   NN                PK,
                                                            `AUTO_INCREMENT`

  `previous_status`   `VARCHAR(100)`      NN                ---

  `new_status`        `VARCHAR(100)`      NN                ---

  `created_at`        `DATETIME`          NN                ---

  `description`       `VARCHAR(100)`      NN                ---

  `ticket_id`         `BIGINT`            NN                No tiene FK
                                                            declarada en el SQL
                                                            recibido

  `technician_id`     `BIGINT`            NN                FK →
                                                            `technicians.id`;
                                                            alinear tipo con
                                                            `BIGINT UNSIGNED`
  -----------------------------------------------------------------------------

### `comments`

  ---------------------------------------------------------------------------
  Atributo          Tipo                Nulabilidad       Clave / restricción
  ----------------- ------------------- ----------------- -------------------
  `id`              `BIGINT UNSIGNED`   NN                PK,
                                                          `AUTO_INCREMENT`

  `content`         `VARCHAR(100)`      NN                ---; valorar `TEXT`

  `created_at`      `DATETIME`          NN                ---

  `contact_id`      `BIGINT`            NN                FK → `contacts.id`;
                                                          alinear tipo con
                                                          `BIGINT UNSIGNED`

  `ticket_id`       `BIGINT`            NN                FK → `tickets.id`;
                                                          alinear tipo con
                                                          `BIGINT UNSIGNED`

  `technician_id`   `BIGINT`            NN                FK →
                                                          `technicians.id`;
                                                          alinear tipo con
                                                          `BIGINT UNSIGNED`
  ---------------------------------------------------------------------------

### `technicians`

  Atributo              Tipo                Nulabilidad   Clave / restricción
  --------------------- ------------------- ------------- ----------------------
  `id`                  `BIGINT UNSIGNED`   NN            PK, `AUTO_INCREMENT`
  `name`                `VARCHAR(100)`      NN            ---
  `corporate_email`     `VARCHAR(100)`      NN            UQ
  `primary_specialty`   `VARCHAR(100)`      NN            ---
  `is_available`        `BOOLEAN`           NN            ---

### `assignments`

  ---------------------------------------------------------------------------
  Atributo          Tipo                Nulabilidad       Clave / restricción
  ----------------- ------------------- ----------------- -------------------
  `id`              `BIGINT UNSIGNED`   NN                PK,
                                                          `AUTO_INCREMENT`

  `started_at`      `DATETIME`          NN                ---

  `ended_at`        `DATETIME`          NN                ---

  `ticket_id`       `BIGINT`            NN                FK → `tickets.id`;
                                                          alinear tipo con
                                                          `BIGINT UNSIGNED`

  `technician_id`   `BIGINT`            NN                FK →
                                                          `technicians.id`;
                                                          alinear tipo con
                                                          `BIGINT UNSIGNED`
  ---------------------------------------------------------------------------

## 3. Relaciones y cardinalidades

-   `companies` 1:N `contacts`.
-   `contacts` 1:N `tickets`.
-   `categories` 1:N `tickets`.
-   `tickets` 1:N `ticket_histories` conceptualmente; falta declarar la
    FK `ticket_histories.ticket_id` → `tickets.id`.
-   `technicians` 1:N `ticket_histories`.
-   `tickets` 1:N `comments`; `contacts` 1:N `comments`; `technicians`
    1:N `comments`.
-   `tickets` N:M `technicians` a lo largo del tiempo, resuelta mediante
    `assignments`.
-   La participación mínima de los registros hijos es obligatoria en las
    FK declaradas, porque son `NOT NULL`. El SQL no exige que un ticket
    tenga comentarios o historial.

## 4. Formas normales

### Primera forma normal (1FN)

Cada fila representa una empresa, contacto, ticket, comentario, cambio
de estado o asignación. Los comentarios y el historial se registran como
filas separadas, en lugar de listas dentro de una celda. Esto favorece
1FN. Los valores de `priority` y `status` deben definirse como valores
individuales válidos, no como listas.

### Segunda forma normal (2FN)

Las tablas usan claves primarias simples, por lo que no existe
dependencia parcial respecto a una clave compuesta en las tablas
actuales. La asociación entre tickets y técnicos se registra en
`assignments`, donde las fechas describen una asignación específica. Si
se desea impedir que se duplique una misma asignación activa, esa regla
requiere una restricción o validación adicional.

### Tercera forma normal (3FN)

Los datos de empresa, contacto, categoría, técnico y ticket están
separados en tablas distintas. Por ejemplo, el nombre de la empresa se
almacena en `companies` y no se repite en cada ticket. El historial y
los comentarios también se guardan en tablas separadas. Para sostener la
normalización y la integridad, hay que completar los ENUM, declarar la
FK faltante de `ticket_histories.ticket_id` y alinear los tipos de las
FK con sus PK.

## 5. Evaluación de cuarta forma normal (4FN)

Los tickets pueden tener muchos comentarios, muchos eventos de historial
y varias asignaciones a técnicos. Son conjuntos de hechos distintos;
deben permanecer en tablas separadas. Si se guardaran comentarios,
eventos y técnicos en una misma tabla, un ticket con varios registros de
cada tipo podría generar combinaciones artificiales y duplicación. No es
un solo atributo el que se normaliza a 4FN: se analizan dependencias
multivaluadas independientes.

## 6. Reglas de integridad adicionales

-   Definir los valores válidos de `priority` y `status`.
-   `ended_at` debe ser posterior a `started_at` en una asignación.
-   Evitar asignaciones activas superpuestas de un técnico a tickets
    incompatibles, según la política de trabajo.
-   Validar transiciones permitidas entre estados de ticket.
-   Cada cambio de estado debe conservar el ticket al que pertenece;
    declarar la FK de `ticket_histories.ticket_id`.
-   Validar que el contacto asociado a un comentario tenga relación
    válida con el ticket si la regla del negocio así lo exige.
-   Revisar si la descripción y el contenido deben ser `TEXT`.
-   Alinear tipos de FK y PK; usar `VARCHAR` para NIT si puede contener
    formato o ceros iniciales.
-   Decidir si las categorías deben tener nombres únicos.

## 7. Fragmentos de datos de prueba

### `companies`

    id legal_name            nit
  ---- --------------------- -----------
     1 Empresa Ejemplo SAS   900123456

### `contacts`

    id name            company_id
  ---- ------------- ------------
    10 Laura Pérez              1

### `tickets`

     id subject                   contact_id   category_id
  ----- ----------------------- ------------ -------------
    100 No funciona el correo             10             2

### `assignments`

    id   ticket_id   technician_id started_at            ended_at
  ---- ----------- --------------- --------------------- ---------------------
     1         100               7 2026-01-10 09:00:00   2026-01-10 10:00:00
     2         100               8 2026-01-10 10:00:00   2026-01-10 11:00:00

Estos datos ilustran que un contacto puede crear tickets y que un ticket
puede tener asignaciones registradas a técnicos en momentos distintos.
Son datos ilustrativos, no insertados en la base de datos.

## 8. Pregunta de sustentación

La duplicación puede provocar que el mismo dato tenga valores diferentes
en distintas filas y generar anomalías al actualizar, insertar o
eliminar. Por ejemplo, guardar el nombre de la empresa en cada ticket
exige modificar todos los tickets cuando cambia ese nombre. Una lista de
IDs de técnicos dentro de una celda dificulta validar cada técnico con
una FK, consultar las asignaciones y guardar fechas de inicio y fin por
asignación. La tabla `assignments` permite representar cada vínculo como
una fila independiente y mantener la integridad.
