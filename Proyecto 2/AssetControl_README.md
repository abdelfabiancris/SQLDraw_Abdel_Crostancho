# AssetControl --- documentación del modelo de datos

## 1. Objetivo

AssetControl representa sedes, departamentos, empleados, activos, tipos
de activos, asignaciones y mantenimientos. Este documento se basa en el
SQL recibido; señala los tipos y relaciones tal como aparecen, y marca
los cambios recomendados.


## 2. Diccionario de datos

**PK** = clave primaria; **FK** = clave foránea; **UQ** = único; **NN**
= `NOT NULL`; **NULL** = permite nulos.

### `sites`

  --------------------------------------------------------------------------
  Atributo          Tipo                Nulabilidad       Clave /
                                                          restricción
  ----------------- ------------------- ----------------- ------------------
  `id`              `BIGINT UNSIGNED`   NN                PK,
                                                          `AUTO_INCREMENT`

  `site_name`       `VARCHAR(100)`      NN                ---

  `code`            `BIGINT`            NN                ---; valorar UQ y
                                                          usar texto si
                                                          puede incluir
                                                          ceros iniciales
  --------------------------------------------------------------------------

### `departaments`

  ------------------------------------------------------------------------------
  Atributo             Tipo                Nulabilidad       Clave / restricción
  -------------------- ------------------- ----------------- -------------------
  `id`                 `BIGINT UNSIGNED`   NN                PK,
                                                             `AUTO_INCREMENT`

  `departament_name`   `VARCHAR(100)`      NN                ---

  `site_id`            `BIGINT`            NN                FK → `sites.id`;
                                                             tipo debe coincidir
                                                             con
                                                             `BIGINT UNSIGNED`
  ------------------------------------------------------------------------------

### `employees`

  -----------------------------------------------------------------------------
  Atributo           Tipo                Nulabilidad       Clave / restricción
  ------------------ ------------------- ----------------- --------------------
  `id`               `BIGINT UNSIGNED`   NN                PK, `AUTO_INCREMENT`

  `name`             `VARCHAR(100)`      NN                ---

  `code`             `BIGINT`            NN                ---; valorar UQ y
                                                           tipo texto si es
                                                           código identificador

  `email`            `VARCHAR(100)`      NN                ---; valorar UQ

  `status`           `BOOLEAN`           NN                ---

  `departament_id`   `VARCHAR(100)`      NN                FK declarada →
                                                           `departaments.id`;
                                                           tipo incorrecto
  -----------------------------------------------------------------------------

### `assets`

  ------------------------------------------------------------------------------------
  Atributo                   Tipo                Nulabilidad       Clave / restricción
  -------------------------- ------------------- ----------------- -------------------
  `id`                       `BIGINT UNSIGNED`   NN                PK,
                                                                   `AUTO_INCREMENT`

  `patrimonial_identifier`   `VARCHAR(100)`      NN                UQ

  `serial_number`            `BIGINT`            NN                UQ

  `brand`                    `VARCHAR(100)`      NN                ---

  `model`                    `VARCHAR(100)`      NN                ---

  `purchase_date`            `DATETIME`          NN                ---

  `status`                   `BOOLEAN`           NN                ---

  `asset_types_id`           `BIGINT`            NN                FK →
                                                                   `asset_types.id`;
                                                                   tipo debe coincidir
                                                                   con
                                                                   `BIGINT UNSIGNED`

  `site_id`                  `BIGINT`            NN                FK → `sites.id`;
                                                                   tipo debe coincidir
                                                                   con
                                                                   `BIGINT UNSIGNED`
  ------------------------------------------------------------------------------------

### `asset_types`

  --------------------------------------------------------------------------
  Atributo          Tipo                Nulabilidad       Clave /
                                                          restricción
  ----------------- ------------------- ----------------- ------------------
  `id`              `BIGINT UNSIGNED`   NN                PK,
                                                          `AUTO_INCREMENT`

  `name`            `VARCHAR(100)`      NN                ---; valorar UQ si
                                                          los nombres de
                                                          tipo deben ser
                                                          únicos
  --------------------------------------------------------------------------

### `assignments`

  -------------------------------------------------------------------------------
  Atributo              Tipo                Nulabilidad       Clave / restricción
  --------------------- ------------------- ----------------- -------------------
  `id`                  `BIGINT UNSIGNED`   NN                PK,
                                                              `AUTO_INCREMENT`

  `assigned_date`       `DATETIME`          NN                ---

  `return_date`         `DATETIME`          NN                ---

  `collection_status`   `BOOLEAN`           NN                ---

  `employes_id`         `BIGINT`            NN                FK →
                                                              `employees.id`;
                                                              revisar ortografía
                                                              y tipo

  `asset_id`            `BIGINT`            NN                FK → `assets.id`;
                                                              tipo debe coincidir
                                                              con
                                                              `BIGINT UNSIGNED`
  -------------------------------------------------------------------------------

### `maintenance`

  ---------------------------------------------------------------------------
  Atributo          Tipo                Nulabilidad       Clave / restricción
  ----------------- ------------------- ----------------- -------------------
  `id`              `BIGINT UNSIGNED`   NN                PK,
                                                          `AUTO_INCREMENT`

  `start_date`      `DATETIME`          NN                ---

  `type`            `BOOLEAN`           NN                ---; revisar si
                                                          debe ser un tipo
                                                          textual o catálogo

  `description`     `VARCHAR(100)`      NN                ---

  `cost`            `BIGINT`            NN                ---; valorar
                                                          `DECIMAL(12,2)`
                                                          para dinero

  `end_date`        `DATETIME`          NN                ---

  `result`          `VARCHAR(100)`      NN                ---

  `asset_id`        `BIGINT`            NN                FK → `assets.id`;
                                                          tipo debe coincidir
                                                          con
                                                          `BIGINT UNSIGNED`
  ---------------------------------------------------------------------------

## 3. Relaciones y cardinalidades

-   `sites` 1:N `departaments`.
-   `departaments` 1:N `employees`.
-   `sites` 1:N `assets`.
-   `asset_types` 1:N `assets`.
-   `employees` 1:N `assignments`.
-   `assets` 1:N `assignments` a lo largo del tiempo.
-   `assets` 1:N `maintenance`.
-   La tabla `assignments` representa la asociación histórica entre
    empleados y activos; conceptualmente permite que un empleado reciba
    distintos activos y un activo sea asignado a distintos empleados en
    momentos diferentes.
-   Las FK están declaradas como `NOT NULL`, por lo que cada fila hija
    requiere un padre. El SQL no especifica que cada empleado o activo
    deba tener asignaciones.

## 4. Formas normales

### Primera forma normal (1FN)

Los atributos contienen valores individuales como un nombre, un correo,
una fecha o un estado. Las asignaciones y mantenimientos se registran
como filas separadas. Esto favorece 1FN, siempre que no se guarden
listas de activos o empleados dentro de una columna.

### Segunda forma normal (2FN)

Las tablas usan claves primarias simples (`id`), por lo que no hay
dependencia parcial de una clave compuesta. La tabla `assignments` tiene
atributos propios de la asignación, como fechas y estado, asociados a
una fila identificada por `id`. Si el negocio exige impedir dos
asignaciones simultáneas del mismo activo, esa regla necesita una
validación adicional; no se resuelve solo con 2FN.

### Tercera forma normal (3FN)

Los datos de sede, departamento, empleado, tipo de activo y activo están
separados en tablas propias, evitando repetir sus nombres y
descripciones en cada asignación o mantenimiento. El modelo apunta a 3FN
por esa separación, pero la integridad del diseño requiere corregir los
tipos incompatibles de las FK y revisar atributos como
`maintenance.type`, que podría representar una categoría de
mantenimiento y quizá pertenecer a un catálogo si se necesita
administrar tipos de forma independiente.

## 5. Evaluación de cuarta forma normal (4FN)

Las asignaciones y los mantenimientos son conjuntos de hechos diferentes
sobre un activo. Mantenerlos en tablas distintas evita combinar cada
asignación con cada mantenimiento y producir filas repetidas. Si un
activo puede tener múltiples mantenimientos y múltiples asignaciones
independientes, no se deben almacenar ambas listas en una misma tabla de
activos. La 4FN se evalúa sobre dependencias multivaluadas; no suele ser
un solo atributo el que se modifica.

## 6. Reglas de integridad adicionales

-   `return_date` debe ser posterior o igual a `assigned_date`, si una
    asignación puede devolverse el mismo día.
-   `end_date` debe ser posterior o igual a `start_date` en
    mantenimientos.
-   `cost` no debe ser negativo.
-   Un activo no debe tener dos asignaciones activas simultáneas si la
    política lo prohíbe.
-   El estado del activo debe actualizarse coherentemente al asignar,
    devolver o enviar a mantenimiento.
-   Si `serial_number` puede contener letras o ceros iniciales, debe ser
    `VARCHAR`, no numérico.
-   Revisar si `code`, `email` y `asset_types.name` necesitan
    restricciones `UNIQUE`.
-   Alinear el tipo y signo de cada FK con el tipo de su PK
    referenciada.
-   Evaluar si `maintenance.type` debe ser `ENUM` o FK a una tabla de
    tipos de mantenimiento.

## 7. Fragmentos de datos de prueba

### `sites`

    id site_name          code
  ---- ---------------- ------
     1 Sede principal      100

### `departaments`

    id departament_name     site_id
  ---- ------------------ ---------
    10 Tecnología                 1

### `assets`

     id patrimonial_identifier     asset_types_id   site_id
  ----- ------------------------ ---------------- ---------
    200 ACT-0200                                3         1

Estos fragmentos ilustran que un departamento y varios activos pueden
pertenecer a una sede, y que cada activo pertenece a un tipo. Los tipos
de FK deben corregirse para que el motor de base de datos acepte las
referencias.

## 8. Pregunta de sustentación

Duplicar datos puede causar inconsistencias y anomalías de
actualización, inserción o eliminación. Por ejemplo, repetir el nombre
de la sede en cada activo obliga a actualizar múltiples filas cuando
cambia. Guardar una lista de IDs de activos en una celda de empleado
dificulta comprobar que cada activo existe, buscar asignaciones
individuales y registrar fechas o estados por activo. Una tabla de
relación como `assignments` permite guardar cada asignación en una fila
con sus propias claves y atributos.
