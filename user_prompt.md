---

```markdown
# USER PROMPT: Plantilla de Ejecución Puntual de Pedido

## [PIEZA 3: TAREA]
Procesa el siguiente mensaje recibido de un cliente gastronómico. Identifica al cliente, mapea cada ítem contra el catálogo de productos proporcionado, valida cantidades y unidades según las restricciones del sistema, y devuelve la estructura JSON requerida.

---

### DATOS DE ENTRADA:

**Canal:** {{canal_origen}} (Email / WhatsApp)  
**Remitente:** {{remitente}}  
**Mensaje Recibido:**  
"""
{{texto_mensaje_pedido}}
"""

**Catálogo de Referencia Vigente:**
- `PROD-010`: Papa Negra (Unidad oficial: Bolsa) | Sinónimos: papa, negra, papas
- `PROD-011`: Tomate Redondo (Unidad oficial: Cajón) | Sinónimos: tomate, redondo, tmate
- `PROD-012`: Tomate Cherry (Unidad oficial: Caja) | Sinónimos: cherry, cherries
- `PROD-020`: Rúcula (Unidad oficial: Paquete) | Sinónimos: rucula, roqueta, paquete rucula
- `PROD-030`: Zanahoria (Unidad oficial: Bolsa / Kg) | Sinónimos: zanahoria, zana
