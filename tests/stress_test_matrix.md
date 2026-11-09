# Matriz de Pruebas de Estrés y Validación de Resiliencia

**Proyecto:** Ecosistema de Automatización IA Autónomo para Negocios (Empresa Sanmartín)  
**Objetivo:** Demostrar la robustez de la arquitectura bajo 5 escenarios operativos diferenciados (Caminos Felices, Aprobación/Rechazo Humano en HITL y Caminos Infelices de fallo de datos y API).

---

## 1. Cuadro Resumen de Pruebas Ejecutadas

| ID Test | Escenario de Prueba | Entrada Clave | Ruta Activada | Intervención Humana | Salida Final | Estado Airtable |
| :---: | :--- | :--- | :---: | :---: | :--- | :--- |
| **TEST-01** | **Camino Feliz:** Solicitud Estándar | Banda y piñones ($750 USD) | Directa (Auto-Aprobada) | No (Bypass seguro) | Email enviado al cliente | `Completado - Enviado al Cliente` |
| **TEST-02** | **Camino Feliz VIP:** Alto Valor con HITL | Cambio Bomba + Estrellas ($4,400 USD) | Rama HITL | **Sí (Aprobado en Slack)** | Email formal despachado | `Completado - Enviado al Cliente` |
| **TEST-03** | **Camino HITL:** Rechazo por Riesgo | Calibración crítica no homologada ($5,800 USD) | Rama HITL | **Sí (Rechazado en Slack)** | Correo cliente cancelado; alerta a Ventas | `Rechazado por Humano` |
| **TEST-04** | **Camino Infeliz 1:** Payload Incompleto | Email inválido y sin equipo | Router Validación (False) | No | Se aborta llamada a IA; Log en auditoría | `Error de Validación` |
| **TEST-05** | **Camino Infeliz 2:** Falla de API de IA | Caída simulada (504 Timeout) | Error Handler / Try-Catch | No | Captura excepción; Log en auditoría | `Pendiente de Reintento` |

---

## 2. Detalle Exhaustivo por Caso de Prueba

### TEST-01: Cotización Estándar (Auto-Aprobada)
* **Objetivo:** Verificar la autonomía del flujo para pedidos estándar de bajo riesgo que no ameritan frenar la operación.
* **Datos Inyectados:**
  * Cliente: Lucía Fernández (`lfernandez@aguasandinas.com`).
  * Requerimiento: "4 metros de banda modular plástica y 2 piñones motrices".
* **Comportamiento Observado:**
  1. Trigger recibe webhook y valida campos correctamente (`es_valido: true`).
  2. Gobernanza desacopla datos de contacto y genera token anónimo `CLI-SAN-8921`.
  3. El motor GPT-4o-mini calcula un monto de **$750 USD** y clasifica el lead como `Estándar`.
  4. El nodo condicional `¿Requiere Validación Humana (HITL)?` evalúa `$750 > $3000` -> **FALSO**.
  5. Salta la pausa HITL y genera la propuesta HTML automáticamente.
  6. Gmail despacha el correo; Slack registra la confirmación en `#notificaciones-ventas`.
  7. Airtable pasa a `Completado - Enviado al Cliente`.

---

### TEST-02: Cotización de Alto Monto con HITL Aprobado
* **Objetivo:** Probar que el sistema frena la acción crítica y espera interacción humana cuando se superan umbrales de seguridad.
* **Datos Inyectados:**
  * Cliente: Carlos Mendoza (`carlos.mendoza@cerveceriadelsur.com`).
  * Requerimiento: "Cambio de juego de estrellas y bomba pre-enjuague para lavadora".
* **Comportamiento Observado:**
  1. IA tasa los repuestos en **$4,400 USD** y detecta cliente con `Score: 92/100 (VIP)`.
  2. Condición HITL detecta `$4,400 > $3,000` -> **VERDADERO**.
  3. Airtable actualiza el registro a `En Espera HITL`.
  4. Slack recibe mensaje con botones de acción interactiva y resumen técnico.
  5. El nodo `Wait (Webhook)` suspende la ejecución sin consumir recursos de CPU.
  6. El responsable comercial hace clic en `APROBAR`.
  7. El flujo despierta de forma reactiva, marca `Aprobado por Humano` y ejecuta el nodo Gmail con la cotización formal validada.

---

### TEST-03: Solicitud de Alto Riesgo Rechazada por Humano
* **Objetivo:** Demostrar que el Human-in-the-Loop previene comprometer a la empresa ante solicitudes técnicamente inviables.
* **Datos Inyectados:**
  * Requerimiento: Calibración de llenadora fuera de turno y con repuestos no homologados ($5,800 USD).
* **Comportamiento Observado:**
  1. IA califica complejidad `Alta` y activa `requiere_hitl: true`.
  2. Notificación despachada a Slack.
  3. El Director Técnico revisa el resumen y pulsa `RECHAZAR`.
  4. La bifurcación detecta `decision === "RECHAZADO"`.
  5. Se cancela inmediatamente la rama de envío de Gmail (no se emite cotización al cliente).
  6. Airtable se actualiza a `Rechazado por Humano` y Slack avisa a ventas para gestión comercial telefónica.

---

### TEST-04: Camino Infeliz 1 - Payload Incompleto o Datos Corruptos
* **Objetivo:** Comprobar la resiliencia ante usuarios que omiten datos críticos o envían formatos inválidos.
* **Datos Inyectados:**
  * `nombre_cliente: ""`
  * `email_cliente: "correo_invalido_sin_formato"`
  * `equipo_industrial: ""`
  * `mensaje_requerimiento: "ayuda"`
* **Comportamiento Observado:**
  1. El nodo `Data Quality & Input Validation` aplica validación por expresiones regulares y longitud de cadenas.
  2. Marca `es_valido: false` y documenta `motivo_invalido: "Faltan campos obligatorios"`.
  3. El Router deriva a la rama de contingencia.
  4. **Se evita la llamada a la API de IA**, ahorrando costos de inferencia y evitando alucinaciones.
  5. Escribe un registro en `Logs_Auditoria_Errores` con código `VALIDATION_ERROR`.
  6. Envía alerta a soporte técnico. El flujo finaliza sin caída ni loops.

---

### TEST-05: Camino Infeliz 2 - Falla de API de IA (Timeout / Rate Limit)
* **Objetivo:** Garantizar que una caída de OpenAI o Anthropic no provoque una interrupción catastrófica del orquestador.
* **Datos Inyectados:**
  * Payload válido con flag de interrupción simulada de API (`504 Gateway Timeout`).
* **Comportamiento Observado:**
  1. El nodo de OpenAI falla por simulación de error de red.
  2. La configuración `onError: continueErrorOutput` atrapa el error inmediatamente.
  3. El nodo `Parse & Re-asociación PII` detecta la bandera `es_error_ia: true`.
  4. El condicional `¿Falló la API de IA?` conmuta a la tabla de `Logs_Auditoria_Errores` registrando el incidente con estado `Pendiente de Reintento`.
  5. No se envían correos incompletos al cliente. El sistema permanece 100% operativo y en estado limpio.
