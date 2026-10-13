# Ecosistema de Automatización IA Autónomo: triage de turnos para un taller mecánico

**Entrega final** del Curso de Automatización con IA.

## ▶️ Video demo

### [Ver el video demo en Loom](https://www.loom.com/share/3c0fcc980a264de789a8e8e83d85793e)

`https://www.loom.com/share/3c0fcc980a264de789a8e8e83d85793e`

Muestra el trigger (llegada del mail), el procesamiento en Make (Escenario A), el aviso en Slack, la aprobación en Airtable, el envío (Escenario B), la respuesta en el hilo de Gmail y el dashboard de KPIs.

---

Cuando un cliente escribe al taller pidiendo un turno, el sistema lee el mail y lo clasifica con IA según las reglas del taller (severidad ALTA, MEDIA, BAJA o ACLARAR). Después redacta un borrador de respuesta y pide aprobación humana por Slack. Solo cuando una persona aprueba el borrador, responde al cliente en el mismo hilo de Gmail.

| Categoría | Herramienta |
|---|---|
| Orquestador | Make (dos escenarios) |
| Base de datos | Airtable (4 tablas vinculadas) |
| Procesamiento IA | Claude Haiku 4.5, con prompt estructurado y RAG sobre la tabla Reglas |
| Canales de salida | Gmail (cliente) y Slack (aprobación y alertas internas) |

## Enlaces obligatorios

| Recurso | Enlace |
|---|---|
| Dashboard de Control (Shared View pública) | https://airtable.com/appWdTh5wf2wSj32h/shr0ZHcdO8hPqTbP2 |
| Base de datos en modo lectura | https://airtable.com/appWdTh5wf2wSj32h/shrEwVZf53bsbH09a |
| Video demo | https://www.loom.com/share/3c0fcc980a264de789a8e8e83d85793e |
| Escenario A en Make (página pública) | https://us2.make.com/public/shared-scenario/bo3DroxN6fL/taller-mecanico-a-ingesta-y-borrador |
| Escenario B en Make (página pública) | https://us2.make.com/public/shared-scenario/2x5zp5cRCp9/taller-mecanico-b-envio-tras-aprobaci |

## Contenido del repositorio

| Archivo | Qué es |
|---|---|
| [PDF Documento Entrega Final.pdf](PDF%20Documento%20Entrega%20Final.pdf) | Documento completo con los 5 entregables |
| [PDF Diagrama de Arquitectura.pdf](PDF%20Diagrama%20de%20Arquitectura.pdf) | Mapa de arquitectura en PDF |
| [PDF Enlaces de la Entrega.pdf](PDF%20Enlaces%20de%20la%20Entrega.pdf) | Todos los enlaces de la entrega en un solo PDF |
| [Blueprint Escenario A Ingesta HITL.blueprint.json](Blueprint%20Escenario%20A%20Ingesta%20HITL.blueprint.json) | Blueprint del Escenario A de Make (importable) |
| [Blueprint Escenario B Envio.blueprint.json](Blueprint%20Escenario%20B%20Envio.blueprint.json) | Blueprint del Escenario B de Make (importable) |
| `Evidencia Fig*.png` | Capturas de evidencia del flujo funcionando (detalle más abajo) |

## Dónde está cada criterio de la rúbrica

| Criterio (20 % cada uno) | Dónde se ve |
|---|---|
| 1. Mapa de arquitectura | [PDF Diagrama de Arquitectura.pdf](PDF%20Diagrama%20de%20Arquitectura.pdf) y sección 2 del documento |
| 2. Estructuras de datos documentadas | Sección 3 del documento ([PDF Documento Entrega Final.pdf](PDF%20Documento%20Entrega%20Final.pdf)): tablas vinculadas, ciclo de estados, esquemas JSON de cada integración y prompt |
| 3. Optimización de costos | Sección 4: matriz de modelo por tarea, Batches y Prompt Caching, costo de orquestación |
| 4. Seguridad y resiliencia | Sección 5: minimización de datos, rutas de error (Break, Resume, Ignore), puntos de Human-in-the-loop |
| 5. Dashboard de Control | Sección 6 y la Shared View pública con KPIs y tasa de error |

## Cómo funciona

- **Escenario A (ingesta y borrador):** el trigger de Gmail toma solo los mails no leídos enviados a la casilla del taller. El escenario crea el ticket en Airtable en estado *Pendiente* y busca las reglas de triage (RAG). Claude clasifica el mail y redacta el borrador. Un Router separa los casos ALTA/MEDIA/BAJA (*Procesado por IA*), ACLARAR (*Requiere revisión*) y respuestas inválidas (*Error*). Por último, se avisa por Slack.
- **Human-in-the-loop:** Make no tiene una pausa nativa para esperar una aprobación, por eso el flujo se divide en dos escenarios conectados por Airtable. El ticket espera ahí hasta que una persona tilda *Aprobado*.
- **Escenario B (envío tras aprobación):** busca tickets con `AND({Aprobado}, {Estado} = 'Procesado por IA')`. Responde al cliente en el mismo hilo (Thread ID), marca el ticket como *Enviado* y confirma en el hilo de Slack (`thread_ts`).
- **Rutas de error:** si falla la IA, se registra en el Log de errores y se aplica Break con 3 reintentos cada 15 minutos. Si el JSON es inválido, se registra y se aplica Resume hacia la ruta de respaldo. Si falla el envío por Gmail, se registra, el ticket se marca en Error y se aplica Ignore.

## Evidencias

| Archivo | Qué demuestra |
|---|---|
| [Evidencia Fig2 Dashboard Incognito.png](Evidencia%20Fig2%20Dashboard%20Incognito.png) | El dashboard se abre sin iniciar sesión, así que el enlace es público |
| [Evidencia Fig3 Escenario A Make.png](Evidencia%20Fig3%20Escenario%20A%20Make.png) | Escenario A con los módulos nombrados, rutas de error y Router |
| [Evidencia Fig4 Slack Aprobacion HITL.png](Evidencia%20Fig4%20Slack%20Aprobacion%20HITL.png) | Notificación de aprobación en Slack |
| [Evidencia Fig5 Tickets Esperando Aprobacion.png](Evidencia%20Fig5%20Tickets%20Esperando%20Aprobacion.png) | Tickets con borrador, detenidos a la espera de aprobación |
| [Evidencia Fig6 Escenario B Frenado Sin Aprobacion.png](Evidencia%20Fig6%20Escenario%20B%20Frenado%20Sin%20Aprobacion.png) | Sin aprobación, el filtro frena y no se envía nada |
| [Evidencia Fig7 Tickets Aprobados.png](Evidencia%20Fig7%20Tickets%20Aprobados.png) | El paso humano que habilita el envío |
| [Evidencia Fig8 Gmail Respuesta En Hilo.png](Evidencia%20Fig8%20Gmail%20Respuesta%20En%20Hilo.png) | Respuesta al cliente en el mismo hilo de Gmail |

## Check de seguridad

1. **Filtro contra bucles infinitos:** el trigger solo toma mails a la dirección del taller (alias `+taller`) que no estén leídos. Las respuestas salen hacia la dirección del cliente, así que nunca vuelven a disparar el flujo.
2. **Tipos de datos correctos en los filtros:** el filtro HITL compara booleano contra booleano (`Aprobado = true`). El Router compara texto contra texto.
3. **Prompt dinámico:** usa el nombre, el asunto y el mensaje del cliente, más las reglas leídas en vivo desde Airtable. No tiene datos fijos.

> Los blueprints no contienen credenciales: Make referencia las conexiones solo por su número interno.
