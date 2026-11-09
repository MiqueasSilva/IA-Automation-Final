# Guion Técnico para Grabación del Video Demo (3 Minutos)

**Objetivo:** Demostrar con máxima solvencia técnica el funcionamiento en vivo del ecosistema de automatización IA para la Empresa Sanmartín, cumpliendo con cada segundo de la pauta de evaluación.

> ⚠️ **Recordatorio de Seguridad:** En la grabación, asegúrate de tener las credenciales y API keys ocultas en n8n/Make (usar variables de entorno o vista minimizada de credenciales).

---

## Estructura Temporal del Video (180 Segundos)

```
00:00 - 00:35 ──► Bloque 1: Presentación del Negocio y Arquitectura
00:35 - 01:25 ──► Bloque 2: Trigger y Procesamiento Inteligente con IA
01:25 - 02:15 ──► Bloque 3: Human-in-the-Loop (HITL) en Acción y Salida Multicanal
02:15 - 02:45 ──► Bloque 4: Resiliencia y Camino Infeliz (Error Handling)
02:45 - 03:00 ──► Bloque 5: Conclusión y Checklist de Seguridad
```

---

## Guion Segundo a Segundo

### Bloque 1: Presentación del Negocio y Arquitectura (00:00 - 00:35)
* **Qué mostrar en pantalla:** Mostrar la vista general del lienzo de n8n con el flujo completo desplegado y la pestaña abierta de Airtable con las 3 tablas relacionales.
* **Qué decir:**
  > *"Hola a todos. Para esta entrega final del curso de IA Automation, presento el Ecosistema Autónomo de Cotizaciones Industriales y Triaje de Leads para la Empresa Sanmartín, fabricante líder de maquinaria de envasado.*  
  > *La arquitectura conecta 4 tecnologías clave: n8n como orquestador central, Airtable como memoria relacional y base de auditoría, GPT-4o-mini con prompts estructurados en JSON como motor de tasación técnica, y una salida multicanal combinando Gmail para propuestas comerciales y Slack para notificaciones internas y validación humana."*

---

### Bloque 2: Trigger y Procesamiento Inteligente con IA (00:35 - 01:25)
* **Qué mostrar en pantalla:** Cambiar a Postman / cURL o cargar un nuevo registro en Airtable (Caso: Carlos Mendoza - Cervecería del Sur - Lavadora de botellas Línea 2). Hacer clic en "Execute Workflow" en n8n para ver las luces de ejecución avanzar.
* **Qué decir:**
  > *"Comenzamos disparando el flujo con una solicitud de cotización real. El nodo de 'Data Quality' valida inmediatamente que los tipos de datos sean correctos y el email cumpla la expresión regular.*  
  > *Inmediatamente después, el módulo de Gobernanza desacopla los datos personales (PII) creando un token anónimo antes de invocar la IA, resguardando la privacidad del cliente.*  
  > *El motor de IA analiza el requerimiento técnico, consulta los valores de referencia del catálogo, desglosa repuestos y mano de obra, y retorna un JSON estructurado con un monto estimado de 4,400 dólares, un score de lead de 92 puntos y clasificación VIP."*

---

### Bloque 3: Human-in-the-Loop (HITL) en Acción y Salida (01:25 - 02:15)
* **Qué mostrar en pantalla:** Mostrar el nodo `IF: ¿Requiere HITL?` tomando la rama superior. Abrir Slack y mostrar el mensaje con botones de "Aprobar" / "Rechazar". Luego hacer clic en "Aprobar" y mostrar cómo el nodo `Wait` de n8n se reanuda al instante. Mostrar la bandeja de entrada de Gmail con el correo recibido.
* **Qué decir:**
  > *"Al superar nuestro umbral de seguridad de 3,000 dólares, el sistema evita el 'efecto metralleta' y se detiene en un nodo de espera reactivo. En lugar de disparar el correo al cliente, envía una alerta al canal comercial de Slack.*  
  > *Aquí vemos la notificación con el resumen del equipo y los repuestos cotizados. Como gerente comercial, reviso la propuesta y presiono 'APROBAR'.*  
  > *Al instante, n8n reanuda la ejecución, actualiza Airtable a 'Aprobado por Humano' y despacha este correo formal por Gmail con un diseño HTML profesional, desglosando la cotización y mapeando el ID de seguimiento."*

---

### Bloque 4: Resiliencia y Camino Infeliz (02:15 - 02:45)
* **Qué mostrar en pantalla:** Disparar una solicitud corrupta (sin equipo y con email inválido). Mostrar cómo el nodo condicional deriva a la rama de error sin llamar a la IA y cómo se crea una fila en la tabla `Logs_Auditoria_Errores` de Airtable.
* **Qué decir:**
  > *"Para verificar la resiliencia, sometemos el sistema a estrés con el 'Camino Infeliz'. Al enviar datos incompletos, el router de validación detecta la anomalía, aborta la llamada a la IA ahorrando costos y escribe de forma autónoma el incidente en nuestra tabla de auditoría con código VALIDATION_ERROR y alerta en Slack.*  
  > *Asimismo, el nodo de OpenAI cuenta con captura de excepciones para que ningún timeout o error de red cuelgue el orquestador."*

---

### Bloque 5: Conclusión y Checklist de Seguridad (02:45 - 03:00)
* **Qué mostrar en pantalla:** Volver al diagrama general y mostrar el repositorio de GitHub con los entregables (JSON, PDF de arquitectura y CSVs).
* **Qué decir:**
  > *"El ecosistema cumple con todos los controles: previene bucles infinitos con filtros de estado, valida tipos estrictos de datos, utiliza prompts 100% dinámicos y protege los datos sensibles de la compañía.*  
  > *Todos los blueprints, esquemas y la documentación técnica completa se encuentran disponibles en el repositorio de GitHub. Muchas gracias."*
