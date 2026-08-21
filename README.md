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
  3. **Restricciones:** Regla de cero inferencia (anti-alucinación), tipado numérico estricto y formato de respuesta pura.
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

## 4. Evidencia de las 3 Corridas Finales (Salidas Estructuradas)

Las 3 salidas estructuradas se encuentran adjuntas en los archivos independientes:
* `Salida_corrida_1.json` (Pedido Válido Estándar - Bistró Luna)
* `Salida_corrida_2.json` (Pedido con Ambigüedades y Faltantes - Revisión Humana)
* `Salida_corrida_3.json` (Pedido Mixto con Cantidades Fraccionarias - Catering Gourmet)

---

## 5. Reflexión: Qué aprendí del Contrato

El desarrollo iterativo de este contrato evidenció que la mayor dificultad en agentes para negocios reales no radica en la capacidad de comprensión del lenguaje, sino en **delimitar los márgenes de certeza**.

1. **La complacencia del modelo:** Por defecto, los LLM tienden a "completar huecos" para agradar al usuario (asumiendo kilos o variedades comunes). En un entorno transaccional, esto genera costos reales. La restricción explícita de "prohibido asumir" es indispensable.
2. **Desacople System/User:** Separar las reglas estáticas y el esquema JSON en el *System Prompt* permite que el *User Prompt* sea ligero, dinámico y reutilizable en cada ejecución sin saturar la ventana de contexto.
3. **El esquema como contrato de interfaz:** Forzar la salida en un esquema JSON cerrado transforma la incertidumbre del lenguaje natural en datos deterministas, permitiendo conectar la IA directamente con sistemas tradicionales (como Google Sheets o ERPs) sin riesgo de quiebre estructural.
