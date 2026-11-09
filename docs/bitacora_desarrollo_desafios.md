# 🛠️ Bitácora de Ingeniería: Decisiones y Desafíos Técnicos Resueltos
**Autor:** Miqueas Silva  
**Proyecto:** Ecosistema Autónomo de Cotizaciones Industriales con IA (Empresa Sanmartín S.A.)  
**Diplomatura:** IA Automation — Coderhouse

---

## 1. Contexto y Objetivos del Desarrollo
Durante el diseño y puesta en marcha del ecosistema en entorno Docker local, surgieron desafíos técnicos reales de integración, concurrencia, rendimiento de hardware y compatibilidad de versiones. A continuación documento las decisiones de ingeniería adoptadas para estabilizar el sistema a nivel de producción.

---

## 2. Registro de Desafíos Técnicos y Soluciones Implementadas

### Desafío 1: Offloading de Inferencia LLM a GPU NVIDIA RTX 4060 y Liberación de VRAM
* **Problema:** En la primera ejecución, el contenedor de Ollama procesaba el modelo `qwen2.5:7b` exclusivamente sobre CPU y memoria RAM del sistema. Esto provocó un pico de uso de CPU del 1500% y una retención de 4.8 GB en la memoria RAM del host a través de WSL2 (`vmmemWSL`), afectando la fluidez del entorno.
* **Diagnóstico:** El contenedor Docker original carecía de la directiva de acceso a hardware de video (`--gpus all`), por lo que Ollama no detectaba los controladores CUDA disponibles en Windows 11.
* **Solución de Ingeniería:**
  1. Recreé el contenedor vinculando el runtime de NVIDIA:  
     `docker run -d --name n8n_ollama --gpus all --network n8n_red_n8n -p 11434:11434 -v n8n_ollama_data:/root/.ollama ollama/ollama:latest`
  2. Verifiqué mediante `nvidia-smi` que el modelo se aloje directamente en los 8 GB de VRAM de la RTX 4060.
  3. Inyecté el parámetro `"keep_alive": "0s"` en el body de la petición HTTP desde n8n. Esto fuerza a Ollama a descargar el modelo de la memoria de video inmediatamente después de emitir la cotización, dejando el sistema en 0% de uso residual.

---

### Desafío 2: Resolución de DNS y Comunicación Interna en Red Docker
* **Problema:** El nodo de inferencia en n8n arrojaba el error:  
  `Problem in node '4. IA': The connection cannot be established due to an incorrect host (domain) value`.
* **Diagnóstico:** La petición intentaba resolver el host genérico `http://ollama:11434`. Al tratarse de una red puente personalizada (`n8n_red_n8n`), el nombre de host asignado por Docker al contenedor es exactamente `n8n_ollama`.
* **Solución de Ingeniería:**
  - Actualicé la URL del nodo a `http://n8n_ollama:11434/api/chat`.
  - Adicionalmente configuré un alias de red en Docker para permitir ambas resoluciones, logrando una latencia de comunicación interna de 0.14 ms.

---

### Desafío 3: Migración de Métodos en Airtable Node (v2)
* **Problema:** Al ejecutar los nodos de lectura y escritura de Airtable, n8n arrojaba:  
  `Cannot read properties of undefined (reading 'execute')`.
* **Diagnóstico:** En las versiones modernas de n8n, los endpoints y selectores de operación del nodo de Airtable sufrieron una deprecación de términos: la acción histórica `append` fue reemplazada por `Create`, y `list` por `Search`.
* **Solución de Ingeniería:**
  - Refactoricé el nodo de consulta de catálogo a `Operation: Search` con selección dinámica de base y tabla.
  - Ajusté el nodo de guardado transaccional y la ruta de error a `Operation: Create`, mapeando explícitamente las columnas de destino (`ID_Solicitud`, `Nombre_Cliente`, `Monto_Total_USD`, `Estado`, etc.).

---

### Desafío 4: Arquitectura HITL Resiliente sin Dependencias Frágiles
* **Problema:** El diseño original contemplaba una notificación interactiva en Slack para la aprobación humana. Sin embargo, depender de la creación de Slack Apps, OAuth y tokens `xoxb-` introducía puntos únicos de fallo externos innecesarios y dependencias frágiles de red.
* **Solución de Ingeniería:**
  - Rediseñé el Human-in-the-Loop aprovechando la capacidad nativa de n8n con el nodo **`Wait (Webhook Resume)`**.
  - Cuando una cotización supera los $3,000 USD, el flujo suspende la ejecución de forma reactiva sin consumir CPU.
  - El sistema espera una llamada autenticada al endpoint de aprobación antes de disparar el correo comercial, cumpliendo la consigna de erradicar el "efecto metralleta" de forma robusta.

---

### Desafío 5: Interpolación de Expresiones y Entrega de Propuestas Formales en Gmail
* **Problema:** El correo de cotización llegaba al destinatario con el HTML formateado pero con las etiquetas crudas `{{ $('5. Procesar Respuesta IA1')... }}` sin evaluar.
* **Diagnóstico:** El campo `Message` del nodo de Gmail se encontraba en modo de texto estático (`Fixed`), enviando el string literal en vez de evaluar la expresión JavaScript. Además, n8n había sufijado el nodo precedente con un identificador numérico (`5. Procesar Respuesta IA1`).
* **Solución de Ingeniería:**
  - Conmuté el campo a modo `Expression` (anteponiendo `=`) y ajusté la referencia del nodo origen a `$('5. Procesar Respuesta IA1')`.
  - El correo se despacha con diseño corporativo en HTML, con asunto parametrizado (`[Ref #COT-XXXX]`) y montos calculados dinámicamente.

---

## 3. Conclusión
El ecosistema final quedó validado en un entorno 100% operativo, local y reproducible, cumpliendo con los estándares de costo cero, alta velocidad de inferencia por GPU, manejo seguro de errores y trazabilidad relacional en base de datos.
