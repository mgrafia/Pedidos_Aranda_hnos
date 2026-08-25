# USER PROMPT: Plantilla de Ejecución Puntual de Pedido

## [PIEZA 3: TAREA]
Procesa el siguiente mensaje recibido de un cliente gastronómico. Identifica al cliente, mapea cada ítem contra el catálogo de productos proporcionado, valida cantidades y unidades según las restricciones del sistema, y devuelve la estructura JSON requerida.

---

### DATOS DE ENTRADA:

**Canal:** {{canal_origen}} (Email / WhatsApp)  
**Remitente:** {{remitente}}  
**Fecha de recepción del mensaje:** {{fecha_recepcion}} (usala como referencia para resolver expresiones relativas como "mañana" o "el viernes")  
**Mensaje Recibido:**  
"""
{{texto_mensaje_pedido}}
"""

**Catálogo de Clientes Vigente:**
{{catalogo_clientes}}
*(si no se provee, tratar como catálogo vacío: `cliente.identificador` siempre `null`)*

**Catálogo de Referencia Vigente:**
- `PROD-010`: Papa Negra (Unidad oficial: Bolsa) | Sinónimos: papa, negra, papas
- `PROD-011`: Tomate Redondo (Unidades oficiales: Kilo, Cajón) | Sinónimos: tomate, redondo, tmate
- `PROD-012`: Tomate Cherry (Unidades oficiales: Kilo, Caja) | Sinónimos: cherry, cherries
- `PROD-013`: Limón (Unidad oficial: Kilo) | Sinónimos: limon, limones
- `PROD-014`: Pepino (Unidad oficial: Kilo) | Sinónimos: pepinos
- `PROD-015`: Naranja Elegida (Unidad oficial: Kilo) | Sinónimos: naranja, naranjas
- `PROD-016`: Banana Ecuador (Unidad oficial: Kilo) | Sinónimos: banana, bananas
- `PROD-017`: Zapallo Zucchini (Unidad oficial: Kilo) | Sinónimos: zapallo, zucchini, zuccini, zapallito
- `PROD-018`: Cebollón (Unidad oficial: Kilo) | Sinónimos: cebollon, cebolla
- `PROD-019`: Lima (Unidad oficial: Kilo) | Sinónimos: limas
- `PROD-020`: Rúcula (Unidad oficial: Paquete) | Sinónimos: rucula, roqueta, paquete rucula
- `PROD-030`: Zanahoria Elegida (Unidades oficiales: Kilo, Bolsa) | Sinónimos: zanahoria, zana, zanahorias
