# CompraFácil --- documentación del modelo de datos

## 1. Objetivo

CompraFácil administra empleados, productos, proveedores, solicitudes,
órdenes de compra y recepciones parciales. El modelo debe conservar el
precio histórico acordado y permitir calcular pendientes desde
cantidades ordenadas y recibidas.


## 2. Diccionario de datos

**PK** = clave primaria; **FK** = clave foránea; **UQ** = único; **NN**
= `NOT NULL`; **NULL** = permite nulos.

### `employees`

  Atributo            Tipo                Nulabilidad   Clave / restricción
  ------------------- ------------------- ------------- ----------------------
  `employee_id`       `BIGINT UNSIGNED`   NN            PK, `AUTO_INCREMENT`
  `employee_code`     `VARCHAR(30)`       NN            UQ
  `full_name`         `VARCHAR(150)`      NN            ---
  `corporate_email`   `VARCHAR(180)`      NN            UQ

### `products`

  Atributo            Tipo                Nulabilidad   Clave / restricción
  ------------------- ------------------- ------------- ----------------------
  `product_id`        `BIGINT UNSIGNED`   NN            PK, `AUTO_INCREMENT`
  `sku`               `VARCHAR(50)`       NN            UQ
  `description`       `VARCHAR(200)`      NN            ---
  `unit_of_measure`   `VARCHAR(30)`       NN            ---

### `suppliers`

  Atributo        Tipo                Nulabilidad   Clave / restricción
  --------------- ------------------- ------------- ----------------------
  `supplier_id`   `BIGINT UNSIGNED`   NN            PK, `AUTO_INCREMENT`
  `tax_id`        `VARCHAR(30)`       NN            UQ
  `legal_name`    `VARCHAR(180)`      NN            ---
  `email`         `VARCHAR(180)`      NN            ---
  `statusenum`    `BOOLEAN`           NN            Default `'ACTIVE'`

**Observación:** `statusenum` tiene nombre poco claro y `BOOLEAN` no
representa estados textuales como `ACTIVE`. Usar `status` con
`ENUM('ACTIVE','INACTIVE')` o `BOOLEAN NOT NULL DEFAULT TRUE`.

### `purchase_requests`

  -------------------------------------------------------------------------------------------------------------------------
  Atributo          Tipo                                                   Nulabilidad       Clave / restricción
  ----------------- ------------------------------------------------------ ----------------- ------------------------------
  `request_id`      `BIGINT UNSIGNED`                                      NN                PK, `AUTO_INCREMENT`; también
                                                                                             aparece como FK incorrecta
                                                                                             hacia
                                                                                             `purchase_orders.request_id`

  `employee_id`     `BIGINT UNSIGNED`                                      NN                FK → `employees.employee_id`

  `request_date`    `DATETIME`                                             NN                ---

  `status`          `ENUM('DRAFT','UNDER_REVIEW','APPROVED','REJECTED')`   NN                Default `DRAFT`
  -------------------------------------------------------------------------------------------------------------------------

**Corrección necesaria:** eliminar la FK circular desde
`purchase_requests.request_id` hacia `purchase_orders.request_id`. La
relación debe ser `purchase_orders.request_id` →
`purchase_requests.request_id`.

### `purchase_request_items`

  ----------------------------------------------------------------------------------------------
  Atributo               Tipo                Nulabilidad       Clave / restricción
  ---------------------- ------------------- ----------------- ---------------------------------
  `request_id`           `BIGINT UNSIGNED`   NN                FK →
                                                               `purchase_requests.request_id`;
                                                               debe integrar PK compuesta

  `product_id`           `BIGINT UNSIGNED`   NN                FK → `products.product_id`; debe
                                                               integrar PK compuesta

  `quantity_requested`   `DECIMAL(12,2)`     NN                Debe ser positiva
  ----------------------------------------------------------------------------------------------

**Corrección necesaria:** usar `PRIMARY KEY(request_id, product_id)` en
lugar de declarar una PK para cada columna por separado.

### `purchase_orders`

  -----------------------------------------------------------------------------------------------------------------------------------
  Atributo          Tipo                                                           Nulabilidad       Clave / restricción
  ----------------- -------------------------------------------------------------- ----------------- --------------------------------
  `order_id`        `BIGINT UNSIGNED`                                              NN                PK, `AUTO_INCREMENT`

  `request_id`      `BIGINT UNSIGNED`                                              NN                UQ; FK esperada →
                                                                                                     `purchase_requests.request_id`

  `supplier_id`     `BIGINT UNSIGNED`                                              NN                FK → `suppliers.supplier_id`

  `order_date`      `DATETIME`                                                     NN                ---

  `status`          `ENUM('ISSUED','PARTIALLY_RECEIVED','RECEIVED','CANCELLED')`   NN                Default `ISSUED`
  -----------------------------------------------------------------------------------------------------------------------------------

La restricción UQ sobre `request_id` sí expresa como máximo una orden
por solicitud.

### `purchase_order_items`

  -------------------------------------------------------------------------------------------------
  Atributo              Tipo                Nulabilidad       Clave / restricción
  --------------------- ------------------- ----------------- -------------------------------------
  `order_id`            `BIGINT UNSIGNED`   NN                FK → `purchase_orders.order_id`; debe
                                                              integrar PK compuesta

  `request_id`          `BIGINT UNSIGNED`   NN                Debe vincularse a la solicitud
                                                              original; FK actual apunta a
                                                              `purchase_request_items.request_id` y
                                                              no identifica por sí sola un renglón

  `product_id`          `BIGINT UNSIGNED`   NN                FK a producto/renglón solicitado; el
                                                              diseño necesita una FK compuesta
                                                              correcta

  `quantity_ordered`    `DECIMAL(12,2)`     NN                Debe ser positiva

  `agreed_unit_price`   `DECIMAL(14,2)`     NN                Precio histórico acordado; debe ser
                                                              no negativo
  -------------------------------------------------------------------------------------------------

**Corrección sugerida:** definir una PK compuesta como
`(order_id, product_id)` si cada producto solo puede aparecer una vez
por orden. Para comprobar que el producto pertenece a la solicitud, usar
una FK compuesta adecuada hacia `(request_id, product_id)` de
`purchase_request_items`, y garantizar que `request_id` coincida con la
solicitud de la orden.

### `receipts`

  -------------------------------------------------------------------------------------------
  Atributo                 Tipo                Nulabilidad       Clave / restricción
  ------------------------ ------------------- ----------------- ----------------------------
  `receipt_id`             `BIGINT UNSIGNED`   NN                PK, `AUTO_INCREMENT`

  `order_id`               `BIGINT UNSIGNED`   NN                FK →
                                                                 `purchase_orders.order_id`

  `delivery_note_number`   `VARCHAR(80)`       NN                UQ

  `receipt_date`           `DATETIME`          NN                ---

  `notes`                  `TEXT`              NULL              ---
  -------------------------------------------------------------------------------------------

### `receipt_items`

  ------------------------------------------------------------------------------------------------
  Atributo              Tipo                Nulabilidad       Clave / restricción
  --------------------- ------------------- ----------------- ------------------------------------
  `receipt_id`          `BIGINT UNSIGNED`   NN                FK → `receipts.receipt_id`; debe
                                                              integrar PK compuesta

  `order_id`            `BIGINT UNSIGNED`   NN                Debe identificar el renglón de orden
                                                              correspondiente

  `product_id`          `BIGINT UNSIGNED`   NN                FK actual incorrecta hacia
                                                              `purchase_order_items.product_id`;
                                                              necesita FK compuesta que
                                                              identifique el renglón de orden

  `quantity_received`   `DECIMAL(12,2)`     NN                Debe ser positiva
  ------------------------------------------------------------------------------------------------

**Corrección necesaria:** usar una PK compuesta, normalmente
`(receipt_id, product_id)` si cada producto aparece una sola vez por
recepción, y diseñar la FK para garantizar que ese producto pertenece a
la orden asociada a la recepción.

## 3. Relaciones y cardinalidades

-   `employees` 1:N `purchase_requests`.
-   `purchase_requests` N:M `products`, resuelta mediante
    `purchase_request_items`.
-   `purchase_requests` 1:0..1 `purchase_orders`, expresada por
    `UNIQUE(purchase_orders.request_id)`.
-   `suppliers` 1:N `purchase_orders`.
-   `purchase_orders` 1:N `purchase_order_items`.
-   `purchase_orders` 1:N `receipts`.
-   `receipts` N:M renglones/productos de la orden, resuelta mediante
    `receipt_items`.
-   La participación mínima de proveedor y solicitud en una orden es
    obligatoria porque las columnas son `NOT NULL`; el SQL debe
    corregirse para que la FK de solicitud tenga la dirección correcta.

## 4. Formas normales

### Primera forma normal (1FN)

Los productos de una solicitud y las cantidades recibidas se representan
en filas de detalle, en lugar de guardar listas de productos en una
celda. Los atributos de cada fila contienen un valor individual. Esto
favorece 1FN, aunque las PK de detalle deben corregirse para permitir
varias filas por solicitud, orden o recepción.

### Segunda forma normal (2FN)

Las tablas de detalle deben utilizar claves compuestas, porque sus
atributos describen una asociación entre un documento y un producto. Por
ejemplo, `quantity_requested` depende de la combinación
`(request_id, product_id)`, y `quantity_ordered` y `agreed_unit_price`
dependen del renglón de orden. Las PK actuales, declaradas por separado
en cada columna, impiden representar correctamente la estructura
prevista; se deben reemplazar por PK compuestas.

### Tercera forma normal (3FN)

Los datos de empleados, productos y proveedores están separados de
solicitudes y órdenes. El precio acordado se conserva en
`purchase_order_items`, porque es un valor histórico propio de la orden,
no un atributo que deba actualizarse automáticamente desde `products`.
Para afirmar 3FN en el diseño operativo, deben corregirse las FK y
evitar duplicar atributos que puedan derivarse de otra tabla sin
necesidad.

## 5. Evaluación de cuarta forma normal (4FN)

Una solicitud contiene varios productos, y una orden puede tener varias
recepciones; cada recepción puede registrar varios renglones. Es
importante mantener separados los conjuntos de productos solicitados,
renglones ordenados y renglones recibidos. Si se mezclaran en una tabla
todas las combinaciones de productos, órdenes y recepciones, aparecerían
duplicaciones y combinaciones artificiales. No es un atributo individual
el que se normaliza a 4FN: se evalúan dependencias multivaluadas
independientes.

## 6. Reglas de integridad adicionales

-   Cantidades solicitadas, ordenadas y recibidas deben ser mayores que
    cero.
-   El precio acordado no debe ser negativo.
-   Los productos de una orden deben estar en la solicitud que la
    originó.
-   La recepción debe pertenecer a la misma orden que sus renglones.
-   La suma de cantidades recibidas por producto no puede superar la
    cantidad ordenada.
-   La cantidad pendiente se calcula como
    `quantity_ordered - SUM(quantity_received)`; no se debe guardar como
    saldo manual redundante.
-   La orden solo puede emitirse para una solicitud aprobada, según la
    regla de negocio.
-   La solicitud debe originar como máximo una orden.
-   La remisión debe ser única.
-   La validación de acumulados debe ser transaccional para evitar que
    dos recepciones concurrentes excedan el límite.
-   Corregir dirección y composición de las claves foráneas en las
    tablas de detalle.

## 7. Fragmentos de datos de prueba

### `purchase_requests`

    request_id   employee_id status
  ------------ ------------- ----------
           100             7 APPROVED

### `purchase_request_items`

    request_id   product_id   quantity_requested
  ------------ ------------ --------------------
           100           20                   10
           100           21                    5

### `purchase_order_items` y `receipt_items`

    order_id   product_id   quantity_ordered   agreed_unit_price
  ---------- ------------ ------------------ -------------------
         500           20                 10               25.00
         500           21                  5                8.00

    receipt_id   product_id   quantity_received
  ------------ ------------ -------------------
           900           20                   6
           901           20                   4

Para el producto 20, el total recibido es 10 y el pendiente es 0. Una
recepción adicional de 1 debería rechazarse.

## 8. Pregunta de sustentación

La información duplicada produce anomalías: cambiar un dato obliga a
actualizar varias filas y puede generar inconsistencias. Una lista de
identificadores en una celda ---por ejemplo, varios `product_id`
separados por comas--- dificulta validar cada producto con una FK,
consultar cantidades por producto y registrar recepciones parciales. Las
tablas de detalle guardan una fila por producto y documento, con claves
que permiten validar cada relación y calcular el pendiente sin almacenar
saldos manuales.
