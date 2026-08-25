# Pedidos_Aranda_hnos
# Entrega: Contrato de Agente para Procesamiento de Pedidos Mayoristas

## 1. Descripción de la Tarea Recurrente
La tarea seleccionada es el **procesamiento, control e interpretación de pedidos no estructurados** recibidos por canales digitales (Email y WhatsApp) en una distribuidora mayorista de frutas y verduras. El agente recibe texto libre, identifica clientes y productos, valida unidades de medida y cantidades, y genera una salida estructurada en JSON para su inserción en bases de datos o derivación a revisión humana.

---

## 2. Especificación de las 6 Piezas del Contrato

El contrato se encuentra desacoplado en dos archivos independientes:

* **En `system_prompt.md`:**
  1. **Rol:** Especialista en operaciones y estructuración transaccional de pedidos mayoristas.
  2. **Contexto:** Dominio gastronómico mayorista, manejo de catálogo de frescos y prevención de errores de entrega.
  3. **Restricciones:** Regla de cero inferencia (anti-alucinación), tipado numérico estricto, formato de respuesta pura, manejo de productos fuera de catálogo, unidades múltiples válidas por producto, identificación de cliente contra catálogo, estado agregado a nivel pedido y manejo de mensajes sin pedido interpretable.
  4. **Formato:** JSON Schema estricto y determinista.
  5. **Ejemplos:** Casos *few-shot* de pedidos válidos y pedidos con ambigüedades.
* **En `user_prompt.md`:**
  6. **Tarea:** Instrucción puntual de extracción, validación y cruce de datos para un mensaje recibido, inyectando el catálogo de referencia.

---

## 3. Documentación de las Iteraciones de Mejora

### 🔁 Iteración 1: Corrección de Suposición de Datos
* **Caso de Prueba:** Mensaje real: *"Hola, pasame para mañana 3 tomates y lechuga."*
* **Qué falló (Textual):** En la corrida inicial (V1), el agente asumió automáticamente que *"3 tomates"* equivalían a 3 kilogramos de Tomate Redondo (`"cantidad": 3, "unidad_medida": "Kg", "codigo_producto": "PROD-011"`), pasando el pedido como `PROCESADO_OK`.
* **Pieza del contrato modificada:** **[RESTRICCIONES]** en `system_prompt.md`. Se añadió explícitamente: *"Si un producto no indica unidad de medida, o si la variedad del producto es ambigua, DEBES marcar el pedido con estado_pedido: 'REQUIERE_REVISION'. Queda ESTRICTAMENTE PROHIBIDO asumir valores por defecto."*
* **Qué cambió en la salida (Después):** En la corrida posterior (V2), el agente clasificó el pedido como `REQUIERE_REVISION`, dejó `codigo_producto: null` y detalló en `motivo_duda`: *"Falta unidad y variedad de tomate"*.

---

### 🔁 Iteración 2: Estandarización de Tipado Numérico y Fracciones
* **Caso de Prueba:** Mensaje real: *"Mandame dos cajones y medio de papa negra y 4 paq de rucula."*
* **Qué falló (Textual):** En la corrida intermedia (V2), el agente devolvió el campo cantidad como un string con texto coloquial: `"cantidad": "2 y medio"`, lo que provocó un error de parsing al intentar guardar el número en la base de datos.
* **Pieza del contrato modificada:** **[FORMATO]** en `system_prompt.md`. Se definió el esquema estricto de tipado: `cantidad: 0.0 (number float)` y se agregó en las restricciones la instrucción de convertir expresiones fraccionarias a formato decimal numérico (`2.5`).
* **Qué cambió en la salida (Después):** En la corrida final (V3), el agente devolvió correctamente `"cantidad": 2.5` como valor numérico flotante y `"unidad_medida": "Cajón"`.

---

### 🔁 Iteración 3: Estado Agregado y Productos Fuera de Catálogo
* **Caso de Prueba:** Mensaje real: *"Para Bistro Central mandame 3 kilos de tomate redondo para mañana y 2 de acelga."*
* **Qué falló (Textual):** La política de cero-inferencia original invalidaba **todo el pedido** ante un solo ítem dudoso, sin distinguir entre "falta un dato" y "el producto directamente no existe en el catálogo". Además no había forma de que un pedido quedara `PROCESADO_OK` en los ítems válidos mientras un ítem puntual se deriva a revisión.
* **Pieza del contrato modificada:** **[RESTRICCIONES]** en `system_prompt.md`. Se agregó una regla específica para "producto fuera de catálogo" (distinta de "variedad ambigua"), y se redefinió `estado_pedido` como agregado a nivel ítem: `PROCESADO_OK` solo si TODOS los ítems son válidos; si hay al menos uno inválido, el pedido pasa a `REQUIERE_REVISION` pero los ítems válidos quedan igualmente documentados y listos para registrar. También se agregó el campo `fecha_entrega_solicitada` al schema (el caso de uso incluye pedidos "para mañana" que antes no se guardaban en ningún lado) y una regla para mensajes sin pedido interpretable (saludos, spam).
* **Qué cambió en la salida (Después):** El tomate quedó `valido: true` con sus datos completos, la acelga quedó `valido: false` con `motivo_duda: "Producto fuera de catálogo."`, y el pedido completo pasó a `REQUIERE_REVISION` sin perder la información ya validada del tomate.

---

### 🔁 Iteración 4: Catálogo Real y Unidades Múltiples por Producto
* **Caso de Prueba:** Al confrontar el catálogo de ejemplo (5 productos, cada uno con una única unidad oficial) contra la lista de precios real de la distribuidora (12 productos), aparecieron dos problemas: (1) varios productos que el ejemplo daba por Cajón/Caja/Bolsa en realidad se facturan por Kilo, y (2) en la operatoria real los clientes piden indistintamente por Kilo suelto o por bulto (Cajón/Bolsa) con peso pre-estimado — algo que el contrato original no contemplaba.
* **Qué falló (Textual):** Con la regla original, un pedido de *"2 cajones de tomate"* se hubiera marcado como `valido: false` por "unidad ambigua", cuando en realidad Cajón es una unidad de venta perfectamente válida para ese producto — el problema real solo existe si el cliente no aclara ninguna unidad.
* **Pieza del contrato modificada:** **[RESTRICCIONES] y catálogo** en `system_prompt.md` / `user_prompt.md`. Se reescribió la regla de unidades para aceptar múltiples unidades oficiales por producto (ej. Tomate Redondo: Kilo o Cajón), marcando inválido solo cuando el cliente no menciona ninguna unidad reconocible — nunca por el simple hecho de que el catálogo liste más de una opción.
* **Qué cambió en la salida (Después):** *"2 cajones de tomate"* ahora se procesa como `cantidad: 2.0, unidad_medida: "Cajón", valido: true`, en vez de derivarse a revisión humana innecesariamente.

---

## 4. Evidencia de las 3 Corridas Finales (Salidas Estructuradas)

Las 3 salidas estructuradas se encuentran adjuntas en los archivos independientes, ya actualizadas contra el contrato final (con `fecha_entrega_solicitada` y unidades múltiples):
* `Salida_corrida_1.json` — Entrada: *"Para Bistró Luna mandame 5 bolsas de papa negra y 10 kg de zanahoria para el viernes."* (Pedido Válido Estándar)
* `Salida_corrida_2.json` — Entrada: *"Mandame 3 de tomate, lechuga criolla y algo de apio."* (Pedido con Ambigüedades y Productos Fuera de Catálogo)
* `Salida_corrida_3.json` — Entrada: *"Para Catering Gourmet mandame 2 cajones y medio de tomate redondo y 6 paquetes de rúcula para el jueves, y unas paltas."* (Pedido Mixto: ítems válidos + 1 fuera de catálogo)

---

## 5. Reflexión:

El desarrollo iterativo de este agente evidenció que la mayor dificultad en agentes para negocios reales no radica en la capacidad de comprensión del lenguaje, sino en **delimitar los márgenes de certeza**.

1. **La complacencia del modelo:** Por defecto, los LLM tienden a "completar huecos" para agradar al usuario (asumiendo kilos o variedades comunes). En un entorno transaccional, esto genera costos reales. La restricción explícita de "prohibido asumir" es indispensable.
2. **Desacople System/User:** Separar las reglas estáticas y el esquema JSON en el *System Prompt* permite que el *User Prompt* sea ligero, dinámico y reutilizable en cada ejecución sin saturar la ventana de contexto.
3. **El esquema como contrato de interfaz:** Forzar la salida en un esquema JSON cerrado transforma la incertidumbre del lenguaje natural en datos deterministas, permitiendo conectar la IA directamente con sistemas tradicionales (como Google Sheets o ERPs) sin riesgo de quiebre estructural.
4. **Catálogo como dato, no como regla:** Confrontar el contrato contra el catálogo real de productos reveló que varias reglas de negocio (qué unidades acepta cada producto, qué tan flexible es "ambiguo") no debían vivir hardcodeadas en el prompt, sino en una fuente de datos editable (el catálogo). Esto permite agregar productos o unidades nuevas sin volver a escribir el contrato.
