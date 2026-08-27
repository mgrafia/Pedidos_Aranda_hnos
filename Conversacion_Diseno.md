# Bitácora de la Conversación de Diseño

Transcripción (reconstruida) de la conversación con Claude que llevó a las iteraciones 3 y 4 documentadas en el `README.md`, y al diseño de la implementación (Apps Script + Google Sheet). Se conserva el orden y la sustancia de cada intercambio; se omiten los detalles técnicos internos de las llamadas a herramientas.

---

**Usuario:** Vamos a crear un agente, necesito que levantes info de mi repo https://github.com/mgrafia/Pedidos_Aranda_hnos

**Claude:** Explora el repo (`README.md`, `system_prompt.md`, `user_prompt.md`, 3 JSON de evidencia) y resume la estructura: un contrato de agente para procesar pedidos mayoristas no estructurados, con política de cero-inferencia, tipado estricto de cantidades y salida JSON determinista.

---

**Usuario:** ¿Las instrucciones de qué tiene que hacer el agente son claras?

**Claude:** Señala 6 ambigüedades del contrato original: falta de catálogo de clientes (el schema pide `identificador` pero nunca se provee de dónde sacarlo), unidad ambigua para Zanahoria (Bolsa/Kg sin desempate), sin regla para productos fuera de catálogo, sin definición de qué pasa con pedidos mixtos (algunos ítems válidos y otros no), sin campo de fecha de entrega pese a que el caso de uso la menciona ("para mañana"), y sin manejo de mensajes sin pedido real.

---

**Usuario:** Dale, sé lo más eficiente posible con el gasto de tokens.

**Claude:** Clona el repo localmente y corrige los 6 puntos: agrega restricciones 4-8 en `system_prompt.md` (fuera de catálogo, unidad ambigua, identificación de cliente, estado agregado, mensaje sin pedido), agrega el campo `fecha_entrega_solicitada` al schema, agrega `{{catalogo_clientes}}` y `{{fecha_recepcion}}` a `user_prompt.md`, y suma un tercer ejemplo few-shot mostrando un pedido mixto.

---

**Usuario:** Ok me gusta, ahora esto va a leer los mails, va a completar los datos en el Excel, pero suceden dos cosas: los pedidos tengo que pasarlos a un grupo de WhatsApp (adjuntos o contenido del mail), y también muchos pedidos llegan por WhatsApp, que se deberían cargar en el mismo file para trazabilidad, y también reenviar al grupo. ¿Cómo interactúa todo eso?

**Claude:** Propone una arquitectura con dos entradas (mail y WhatsApp) que confluyen en el mismo pipeline (extracción → agente → JSON), con dos salidas (carga en Excel siempre, reenvío al grupo de WhatsApp siempre), y advierte sobre el riesgo de loop si el bot escucha su propio grupo. Pregunta por el orden de las operaciones y por el tipo de adjuntos esperados.

---

**Usuario:** 1. Lo ideal es tener alguna alerta si falta algún detalle de cantidad para consultar con el cliente. 2. Los adjuntos de WhatsApp suelen ser captura de pantalla del mail o PDF del pedido.

**Claude:** Agrega al diseño un paso de extracción de texto (OCR/visión) para adjuntos, y un branch de "consulta al cliente" cuando algún ítem tiene `motivo_duda`.

---

**Usuario:** No quiero que el agente hable con el cliente, sino que me avise a mí y yo hablar con el cliente.

**Claude:** Saca el branch de respuesta automática al cliente; la alerta pasa a ser solo para el usuario dueño del negocio.

---

**Usuario:** Tiene que ser directo a mí, no al grupo.

**Claude:** Separa el flujo: el grupo recibe siempre el contenido del pedido; la alerta de "necesita revisión" llega aparte, directo al usuario.

---

**Usuario:** El mail es Gmail (con acceso), WhatsApp es personal (no Business), y el Excel está en una carpeta de Drive: https://drive.google.com/drive/folders/1ueFKq7hxZisdWaHHA6JzhHZYWjFPjT2u

**Claude:** Verifica accesos reales: no hay conector de Gmail ni de WhatsApp disponible en la sesión; sí hay acceso a la carpeta de Drive, donde ya existe una estructura previa (`01_Listas_Precios_Semanales`, `02_Documentacion_y_Plantillas`) con una guía que menciona un Sheet maestro `Gestion_Mayorista` (todavía no creado) y otro agente basado en Gemini para listas de precios. Plantea 3 preguntas de arquitectura: dónde viven los pedidos, cómo manejar el riesgo de baneo de WhatsApp no oficial, y en qué plataforma correr la automatización.

---

**Usuario:** (vía selección) Misma planilla maestra / "no quiero ese riesgo, ¿qué otra opción tengo si no quiero pasar a WhatsApp Business? ¿Gemini que tengo integrado en mi celular puede servir?" / Google Apps Script.

**Claude:** Aclara que el Gemini del teléfono no sirve para automatización pasiva (es un asistente por comando puntual, no un bot en segundo plano). Propone un puente manual de bajo riesgo: reenviar el WhatsApp a un mail dedicado (mismo pipeline que el canal Email) y que el bot prepare el mensaje para el grupo/alerta personal listo para copiar y pegar, sin tocar la API de WhatsApp.

---

**Usuario:** Dale.

**Claude:** Crea el Google Sheet `Gestion_Mayorista` en la carpeta de Drive con la pestaña `Pedidos`, y escribe el Apps Script completo (`Codigo_Apps_Script.gs`): lee Gmail por etiquetas, llama a Claude (con soporte multimodal para PDF/imagen), escribe en el Sheet, y manda un mail con el mensaje listo para el grupo + alerta si hay algo dudoso. Da instrucciones de setup y CSV para las pestañas `Clientes` y `Catalogo_Productos`.

---

**Usuario:** El del contrato es un ejemplo, la lista es claramente más amplia.

**Claude:** Cruza el catálogo de 5 productos del contrato original contra la lista de precios real (`Plantilla_Lista_Precios_Semanal`, 12 productos únicos), actualiza `system_prompt.md`, `user_prompt.md` y el `.gs` en consecuencia, y señala un conflicto de unidades (Cajón/Caja/Bolsa vs. Kilo) para confirmar con el usuario.

---

**Usuario:** ¿Cómo venimos con respecto a lo que pide la rúbrica?

**Claude:** Aclara que no tiene el texto de la rúbrica; hace un check inferido contra la estructura del propio `README.md` (6 piezas ✅, iteraciones ⚠️ desactualizadas, evidencia ⚠️ desactualizada, reflexión ✅, nada pusheado ❌) y propone un orden para cerrar la entrega.

---

**Usuario:** ¿Qué está pendiente de confirmar? Se me pasó esa pregunta.

**Claude:** Recuerda la pregunta pendiente: si tomate/cherry/zanahoria se venden realmente por Kilo o si el Kilo es solo la unidad de precio pero se entregan en bulto (cajón/bolsa).

---

**Usuario:** Está bien que sea por Kilo, si bien pueden pedir por bulto (cajón/bolsa) que tienen pesos pre-estimados.

**Claude:** Rediseña la restricción de unidades: en vez de "unidad ambigua si hay más de una oficial", ahora cualquier unidad listada (Kilo o el bulto correspondiente) es válida si el cliente la indica explícitamente. Aclara que no inventa el peso estimado por bulto por no tener el dato real.

---

**Usuario:** Naranja, papa, banana, zapallo, naranja, limón y los tomates (también se piden por bulto).

**Claude:** Suma Cajón como unidad alternativa a esos productos en el catálogo.

---

**Usuario:** Todo se vende por bulto o por Kg/paquete.

**Claude:** Completa el patrón para los productos que quedaban solo en Kilo (pepino, cebollón, lima), y pregunta puntualmente por Rúcula (única con unidad base "Paquete", no Kilo).

---

**Usuario:** No, está bien así, métele.

**Claude:** Confirma el catálogo final (Rúcula solo Paquete, sin conversión automática cajón→kilos por falta de datos reales de peso). Regenera las 3 corridas de evidencia contra el contrato final y agrega las iteraciones 3 y 4 al `README.md` documentando todo el proceso de esta sección.

---

**Usuario:** Pusheá al repo.

**Claude:** Commit y push a `main` (`e58ab49`) con los 6 archivos actualizados.

---

**Usuario:** ¿Qué branch usaste?

**Claude:** `main`, la única branch del repo.

---

**Usuario:** ¿Me subís esta chat a la carpeta del proyecto de GitHub?

**Claude:** Este archivo.
