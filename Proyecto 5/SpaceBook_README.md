# SpaceBook --- documentación del modelo de datos

## 1. Objetivo

SpaceBook administra sedes, empleados, salas, reservas, participantes,
recursos y bloqueos de salas. Este documento describe el SQL recibido,
sus claves y nulabilidad, y señala las correcciones necesarias antes de
afirmar que el modelo cumple todas las formas normales.



## 2. Diccionario de datos

Convenciones: **PK** = clave primaria; **FK** = clave foránea; **UQ** =
valor único; **NN** = `NOT NULL`; **NULL** = permite nulos. Los campos
sin indicación `NULL` explícita son `NOT NULL` en el SQL.

### `sites`

  Atributo      Tipo                Nulabilidad   Clave / restricción
  ------------- ------------------- ------------- ----------------------
  `site_id`     `BIGINT UNSIGNED`   NN            PK, `AUTO_INCREMENT`
  `site_code`   `VARCHAR(30)`       NN            UQ
  `name`        `VARCHAR(100)`      NN            ---
  `city`        `VARCHAR(100)`      NN            ---

### `employees`

  Atributo          Tipo                Nulabilidad   Clave / restricción
  ----------------- ------------------- ------------- ----------------------
  `employee_id`     `BIGINT UNSIGNED`   NN            PK, `AUTO_INCREMENT`
  `employee_code`   `VARCHAR(30)`       NN            UQ
  `full_name`       `VARCHAR(100)`      NN            ---
  `email`           `VARCHAR(100)`      NN            UQ
  `home_site_id`    `BIGINT UNSIGNED`   NN            FK → `sites.site_id`

### `rooms`

  --------------------------------------------------------------------------------------------------------
  Atributo          Tipo                                              Nulabilidad       Clave /
                                                                                        restricción
  ----------------- ------------------------------------------------- ----------------- ------------------
  `room_id`         `BIGINT UNSIGNED`                                 NN                PK,
                                                                                        `AUTO_INCREMENT`

  `site_id`         `BIGINT UNSIGNED`                                 NN                FK →
                                                                                        `sites.site_id`

  `room_code`       `VARCHAR(30)`                                     NN                ---

  `name`            `VARCHAR(100)`                                    NN                ---

  `capacity`        `INT UNSIGNED`                                    NN                ---

  `status`          `ENUM('AVAILABLE','UNAVAILABLE','MAINTENANCE')`   NN                Default
                                                                                        `AVAILABLE`
  --------------------------------------------------------------------------------------------------------

**Recomendación:** si `room_code` solo debe ser único dentro de una
sede, agregar `UNIQUE(site_id, room_code)`.

### `reservations`

  -------------------------------------------------------------------------------------------------------------------------------
  Atributo                  Tipo                                                      Nulabilidad       Clave / restricción
  ------------------------- --------------------------------------------------------- ----------------- -------------------------
  `reservation_id`          `BIGINT UNSIGNED`                                         NN                PK, `AUTO_INCREMENT`

  `room_id`                 `BIGINT UNSIGNED`                                         NN                FK → `rooms.room_id`

  `organizer_employee_id`   `BIGINT UNSIGNED`                                         NN                FK →
                                                                                                        `employees.employee_id`

  `start_at`                `DATETIME`                                                NN                ---

  `end_at`                  `DATETIME`                                                NN                ---

  `reason`                  `VARCHAR(500)`                                            NN                ---

  `status`                  `ENUM('REQUESTED','CONFIRMED','CANCELLED','COMPLETED')`   NN                Default `REQUESTED`

  `created_at`              `DATETIME`                                                NN                ---
  -------------------------------------------------------------------------------------------------------------------------------

### `reservation_participants`

  ---------------------------------------------------------------------------------------------------------------------
  Atributo                 Tipo                                      Nulabilidad       Clave / restricción
  ------------------------ ----------------------------------------- ----------------- --------------------------------
  `reservation_id`         `BIGINT UNSIGNED`                         NN                FK →
                                                                                       `reservations.reservation_id`;
                                                                                       debe formar PK compuesta

  `employee_id`            `BIGINT UNSIGNED`                         NN                FK → `employees.employee_id`;
                                                                                       debe formar PK compuesta

  `invited_at`             `DATETIME`                                NN                ---

  `participation_status`   `ENUM('INVITED','ACCEPTED','DECLINED')`   NN                Default `INVITED`
  ---------------------------------------------------------------------------------------------------------------------

**Corrección necesaria:** usar
`PRIMARY KEY (reservation_id, employee_id)` y eliminar las dos
declaraciones de PK separadas.

### `resources`

  ---------------------------------------------------------------------------------------------------------
  Atributo           Tipo                                              Nulabilidad       Clave /
                                                                                         restricción
  ------------------ ------------------------------------------------- ----------------- ------------------
  `resource_id`      `BIGINT UNSIGNED`                                 NN                PK,
                                                                                         `AUTO_INCREMENT`

  `inventory_code`   `VARCHAR(50)`                                     NN                UQ

  `name`             `VARCHAR(100)`                                    NN                ---

  `resource_type`    `VARCHAR(80)`                                     NN                ---

  `status`           `ENUM('AVAILABLE','UNAVAILABLE','MAINTENANCE')`   NN                Default
                                                                                         `AVAILABLE`
  ---------------------------------------------------------------------------------------------------------

### `reservation_resources`

  ------------------------------------------------------------------------------------------------------------------
  Atributo           Tipo                                         Nulabilidad       Clave / restricción
  ------------------ -------------------------------------------- ----------------- --------------------------------
  `reservation_id`   `BIGINT UNSIGNED`                            NN                FK →
                                                                                    `reservations.reservation_id`;
                                                                                    debe formar PK compuesta

  `resource_id`      `BIGINT UNSIGNED`                            NN                FK → `resources.resource_id`;
                                                                                    debe formar PK compuesta

  `requested_at`     `DATETIME`                                   NN                ---

  `request_status`   `ENUM('REQUESTED','ASSIGNED','CANCELLED')`   NN                Default `REQUESTED`
  ------------------------------------------------------------------------------------------------------------------

**Corrección necesaria:** usar
`PRIMARY KEY (reservation_id, resource_id)` y eliminar las dos
declaraciones de PK separadas.

### `room_blocks`

  Atributo          Tipo                Nulabilidad   Clave / restricción
  ----------------- ------------------- ------------- ----------------------
  `room_block_id`   `BIGINT UNSIGNED`   NN            PK, `AUTO_INCREMENT`
  `room_id`         `BIGINT UNSIGNED`   NN            FK → `rooms.room_id`
  `start_at`        `DATETIME`          NN            ---
  `end_at`          `DATETIME`          NN            ---
  `reason`          `VARCHAR(300)`      NN            ---
  `created_at`      `DATETIME`          NN            ---

## 3. Relaciones y cardinalidades

-   `sites` 1:N `employees`: una sede puede ser sede de referencia de
    muchos empleados; cada empleado apunta a una sede.
-   `sites` 1:N `rooms`: una sede puede tener muchas salas; cada sala
    pertenece a una sede.
-   `rooms` 1:N `reservations`: una sala puede tener muchas reservas en
    distintos horarios; cada reserva corresponde a una sala.
-   `employees` 1:N `reservations`: un empleado puede organizar muchas
    reservas; cada reserva tiene un organizador.
-   `reservations` N:M `employees`, resuelta mediante
    `reservation_participants`.
-   `reservations` N:M `resources`, resuelta mediante
    `reservation_resources`.
-   `rooms` 1:N `room_blocks`: una sala puede tener múltiples bloqueos.
-   Participación mínima: los campos FK son `NOT NULL`, así que cada
    registro hijo debe referenciar un padre. El SQL no impone que toda
    reserva tenga al menos un participante adicional ni que cada sala
    tenga una reserva o bloqueo.

## 4. Formas normales

### Primera forma normal (1FN)

Los atributos definidos son valores individuales ---por ejemplo, un
`email`, un `start_at` o un `status`--- y no listas de participantes o
recursos dentro de una celda. Las relaciones múltiples se representan
mediante tablas de asociación. Esto favorece 1FN, siempre que no se
almacenen listas serializadas en los campos de texto.

### Segunda forma normal (2FN)

Las tablas con PK simple cumplen 2FN de manera natural si cada atributo
no clave depende de la PK completa. En las tablas de asociación, la
intención correcta es una PK compuesta: `(reservation_id, employee_id)`
y `(reservation_id, resource_id)`. Los atributos `invited_at`,
`participation_status`, `requested_at` y `request_status` describen cada
asociación completa, no solamente uno de sus lados. El SQL actual debe
corregirse para expresar esas claves compuestas.

### Tercera forma normal (3FN)

Los datos descriptivos se separan por entidad: los datos de sede están
en `sites`, los del empleado en `employees`, los de sala en `rooms` y
los del recurso en `resources`. Por ejemplo, no se repite `name` de la
sede dentro de cada empleado o sala. Bajo estas dependencias funcionales
esperadas, el diseño apunta a 3FN; deben verificarse las reglas reales
de negocio y corregirse las PK de las tablas de asociación.

## 5. Evaluación de cuarta forma normal (4FN)

Las listas independientes de participantes y recursos de una reserva
están separadas en `reservation_participants` y `reservation_resources`.
Esto evita combinar cada participante con cada recurso en una misma
tabla, lo cual generaría combinaciones redundantes. No hay un único
atributo que "se normalice" por sí solo: la 4FN evalúa dependencias
multivaluadas entre conjuntos de atributos. La separación actual es
apropiada para ese riesgo, una vez corregidas las claves primarias.

## 6. Reglas de integridad que requieren validación adicional

-   `end_at > start_at` en reservas y bloqueos.
-   Evitar reservas confirmadas superpuestas para una misma sala.
-   Evitar reservas durante un bloqueo de sala.
-   Evitar asignar un recurso a reservas que coincidan en el tiempo si
    el recurso no puede compartirse.
-   Verificar que la cantidad de participantes no exceda la capacidad de
    la sala.
-   Definir si el organizador cuenta como participante y evitar contarlo
    dos veces.
-   Validar transiciones permitidas de estados.
-   Para evitar conflictos concurrentes, validar solapamientos dentro de
    una transacción o mecanismo equivalente.

## 7. Fragmentos de datos de prueba

### `sites`

    site_id site_code   name               city
  --------- ----------- ------------------ -------------
          1 BGA         Sede Bucaramanga   Bucaramanga

### `rooms`

    room_id   site_id room_code   name       capacity
  --------- --------- ----------- -------- ----------
         10         1 SALA-A      Sala A            8

### `reservation_participants` (después de corregir la PK compuesta)

    reservation_id   employee_id participation_status
  ---------------- ------------- ----------------------
               100             1 ACCEPTED
               100             2 INVITED
               101             1 INVITED

Estos datos muestran que un empleado puede participar en distintas
reservas y una reserva puede tener varios empleados.

## 8. Pregunta de sustentación

Guardar información duplicada puede producir inconsistencias y anomalías
de actualización, inserción o eliminación. Por ejemplo, repetir el
nombre de una sede en muchas filas de empleados obliga a actualizarlo en
varios lugares y puede dejar valores distintos. Guardar una lista como
`1,2,3` en una sola celda impide relacionar cada identificador de forma
limpia mediante una FK, dificulta buscar y validar participantes y
complica registrar atributos propios de cada relación. Es preferible una
fila por asociación en una tabla intermedia, con claves y restricciones.
