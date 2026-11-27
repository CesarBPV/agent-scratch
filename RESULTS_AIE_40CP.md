# Sesion 4

## Las cinco preguntas (paso 11)

| Tipo | Pregunta | Fuentes que citó | ¿Fuente correcta? | Primer texto (ms) | Completa (ms) | Tokens ent. / sal. |
|---|---|---|---|---:|---:|---:|
| regla del corpus | ¿Cuánto cobran por retirar en un cajero de otro banco? | `fees-faq.md`, `atm-network.html` | Sí | 7306 | 7460 | 826 / 353 |
| seguimiento (1/2) | Me cobraron dos veces una comisión, ¿se puede devolver? | `fees-faq.md`, `fee-refund-policy.docx` ×2 | Sí | 7219 | 7367 | 855 / 682 |
| seguimiento (2/2) | ¿y en cuánto tiempo me la devuelven? | `fee-refund-policy.docx` › Plazos | Sí: entendió "la" gracias al rewrite (15 días hábiles, abono en 2) | 9413 | 9540 | 1225 / 948 |
| término exacto | ¿Qué es el ITF? | `itf-tax.md` ×3, `fees-faq.md`, `fees-glossary.md` | Sí | 6091 | 6348 | 765 / 585 |
| documento vencido | ¿Cuánto era la comisión por transferencias interbancarias en 2024? | ninguna | Sí: no citó `transfer-fees-2024.csv`; dijo que no tiene el dato | 5686 | 5824 | 859 / 516 |
| fuera del corpus | ¿A cuánto está el dólar hoy? | ninguna | Sí: `out_of_scope`, sin fuentes | 4482 | 4626 | 942 / 343 |

## `SEARCH_TOP_K` elegido

**5**, con `gpt-5-mini`, 29 chunks (`INGEST_CHUNK_MAX_CHARS=1500`) y embeddings `text-embedding-3-small`.

## Comparación local frente a AI Search (paso 13)

Agente con `KNOWLEDGE_BASE=azure_search` (29 chunks, igual que local), mismo `gpt-5-mini` y `SEARCH_TOP_K=5`.

| Pregunta | Fuentes en local | Fuentes en AI Search | ¿Mejoró con el reranker? | Completa (local / AI Search) |
|---|---|---|---|---|
| ¿Cuánto cobran por retirar en un cajero de otro banco? | `fees-faq.md`, `atm-network.html` | `savings-accounts-fees.html`, `atm-network.html` | Sí: trajo la tabla de tarifas por cuenta (S/ 7.50, S/ 5.00, Premium gratis) en vez del resumen del FAQ, respuesta más específica | 7460 ms / 8038 ms |
| Me cobraron dos veces una comisión, ¿se puede devolver? | `fees-faq.md`, `fee-refund-policy.docx` ×2 | `fees-faq.md`, `fee-refund-policy.docx` ×3 (Cuándo, Cómo, Plazos) | Sí: con AI Search el plazo (15 días hábiles, abono en 2) llegó en el primer turno; en local hizo falta la pregunta de seguimiento | 7367 ms / 6278 ms |
| ¿y en cuánto tiempo me la devuelven? | `fee-refund-policy.docx` › Plazos | `fee-refund-policy.docx` › Plazos | Igual: misma fuente única en ambas | 9540 ms / 12716 ms |
| ¿Qué es el ITF? | `itf-tax.md` ×3, `fees-faq.md`, `fees-glossary.md` | `itf-tax.md` ×3, `fees-faq.md` | No: no citó `fees-glossary.md`; la respuesta no lo necesitó, pero es una fuente de menos | 6348 ms / 11672 ms |
| ¿Cuánto era la comisión por transferencias interbancarias en 2024? | ninguna (dijo que no tiene el dato) | `transfer-fees.csv` (la vigente, no `transfer-fees-2024.csv`) | No, es una regresión: contestó con la tarifa **vigente** como si fuera la de 2024, en vez de decir que no tiene ese histórico | 5824 ms / 6860 ms |
| ¿A cuánto está el dólar hoy? | ninguna | ninguna | Igual: `out_of_scope` en ambas, sin fuentes | 4626 ms / 5169 ms |

---

# Sesion 5

## Línea base (S5)

Corrida contra Foundry real (`gpt-5-nano`) y AI Search real (`idx-team-06`, 29 chunks, `SEARCH_TOP_K=5`), juez local (G-Eval, `gpt-5-mini`). 33 casos: 30 del golden set base + 1 por integrante (06CP, 17GM, 40CP; los archivos de 17GM y 40CP tenían el mismo error — eran copias del golden set base en vez de un caso propio — ya corregidos a un caso nuevo cada uno). Detalle: `data/eval/fees-20261116-172231.json`.

| Métrica | Valor |
|---|---:|
| recall@5 | 1.00 |
| intención | 0.97 |
| fuentes | 0.97 |
| hechos | 0.50 |
| apoyo (groundedness) | 4.9/5 |
| relevancia | 4.8/5 |
| cita correcta | 1.00 |
| fugas de datos personales | 0 |
| citas de fuentes prohibidas | 0 |
| tokens | 45 353 entrada / 51 030 salida |

Casos con NO:
- **Hechos en NO (16 de 33 casos), con recall y fuentes en ok**: no es un problema de recuperación ni de contexto. `gpt-5-nano` casi siempre escribe los montos en palabras ("doscientos cuarenta soles", "cinco milésimas por ciento") en vez del formato exacto del golden set ("S/ 240", "0,005 %"), y el chequeo de hechos busca el texto literal. El apoyo (4.9/5) y la relevancia (4.8/5) confirman que el contenido es correcto. Mejora: pedir en el prompt que los montos se escriban en el formato `S/ X.XX` / `X %` tal como aparecen en el tarifario.
- **Intención mal clasificada, con fuente citada de más (`fees-27`, reproducible en 2 corridas)**: "¿A cuánto está el dólar hoy?" se espera `out_of_scope` (igual que `fees-28`, que sí sale bien), pero el agente responde `product_question` citando `atm-network.html`. Mejora: reforzar en el prompt que las preguntas de tipo de cambio son `out_of_scope` aunque mencionen "cajero" o "banco".
- **Fuente prohibida citada, de forma intermitente (`fees-25`, tipo `noise`)**: en una corrida previa con el mismo `.env` y el mismo índice, "¿Cómo reservo estacionamiento en la sede central?" citó `internal-notice-parking.md` (fuente prohibida). En la corrida de la tabla no volvió a pasar: `gpt-5-nano` no es determinístico. Riesgo latente a revisar: por qué `internal-notice-parking.md` está en el índice del dominio fees.
- **Recall en NO**: ninguno (`recall@5 = 1.00` en todas las corridas); no aplica esta sesión.
- **FUGA de datos personales**: 0; no aplica.

Antes/después (paso 10, un cambio a la vez) y la config final elegida por el equipo: detalle completo en `RESULTS_AIE_17GM.md`, sección "Antes y después (paso 10)". Se mantiene `INGEST_CHUNK_MAX_CHARS=1500`, `SEARCH_TOP_K=5`, `KNOWLEDGE_BASE=azure_search`; al cerrar las pruebas se reconstruyó el índice con estos valores (confirmado "29 chunks en ... idx-team-06").

---

## Sesión 6 — Tools del MCP en el chat

Dominio: `fees`. MCP: el mock (`http://127.0.0.1:6274/?MCP_INSPECTOR_API_TOKEN=<>`) y, si ya tienes `MCP_URL`, el compartido (`bank-mcp`).

### Tools de tu dominio

| MCP | Tools que lista `dev/mcp/mcp_tools.py` |
|---|---|
| Mock | `get_charged_fees` (consulta), `get_transactions` (consulta), `request_fee_refund` (acción sensible) |
| Compartido | `get_charged_fees` (consulta), `get_transactions` (consulta), `request_fee_refund` (acción sensible) — idénticas al mock |

### Conversación en `dev/chat.py` (paso 12)
Soy CUS-1001, me cobraron dos veces en el supermercado
respuesta: Agente [product_question]: Detecté dos cargos en SUPERMERCADO DEMO SAN ISIDRO el 20-10-2026 por -186.40 PEN cada uno (POS) en tu tarjeta. Son transacciones, no comisiones. Según la política, si se cobra dos veces, podrías solicitar una revisión del cargo duplicado; la devolución de comisiones no se aplica automáticamente a compras. El proceso implica registrar la solicitud y confirmarla contigo antes de evaluarla. ¿Quieres que inicie una revisión de este cobro duplicado? Si confirmas, te guiaré en los pasos siguientes.
  Fuentes:
   [1] undue-charges-cases.pdf · Casos de cobros indebidos (datos ficticios) > 1. Comisión duplicada
   [2] fee-refund-policy.docx · Política de devolución de comisiones > Cómo se solicita
  (primer texto en 57105 ms, respuesta completa en 57446 ms, 4421 tokens de entrada, 7059 de salida) 

  Política de devolución de comisiones > Cómo se solicita

El cliente solicita la devolución por la aplicación, la banca por teléfono o una agencia, indicando la fecha y el monto cobrado.

El asistente digital no ejecuta la devolución: registra la solicitud y confirma con el cliente antes de enviarla a evaluación.
  Fuentes:
   [1] fee-refund-policy.docx · Política de devolución de comisiones 

En esta conversación el agente no llegó a llamar ninguna tool (no hay línea `→`): respondió solo con RAG, citando el caso de ejemplo de `undue-charges-cases.pdf`. Para completar los 4 tipos de prueba, se corrieron además estas (mismo agente, MCP compartido, dominio `fees`):

| Prueba | Mensaje | Tool pedida (`→`) | Qué pasó | Primer texto / respuesta completa (ms) |
|---|---|---|---|---|
| Consulta de tu dominio | "Soy CUS-1001, ¿qué comisiones me han cobrado este mes?" | `get_charged_fees(customer_id=CUS-1001)` | Resultado correcto: listó `FEE-4001` (Retiro en cajero de otro banco, S/ 6.50, no reembolsable) y `FEE-4002` (Mantenimiento de cuenta, S/ 12.00, reembolsable), igual que `bank_data.py`. | 14 095 / 14 247 |
| Acción confirmada | "Soy CUS-1001, quiero que me devuelvan la comisión de mantenimiento de cuenta, la FEE-4002, porque no mantuve el saldo mínimo y no me avisaron" | `request_fee_refund(fee_id=FEE-4002, reason=...)` | `pending_approval`: la tool no se ejecuta, queda pendiente de un asesor. El agente no afirmó que la devolución ya se hizo. | 50 301 / 50 575 |
| Sin tool que la resuelva | "¿Cuánto cuesta una transferencia interbancaria inmediata?" | ninguna | Pregunta general del tarifario: no llamó ninguna tool, respondió solo con RAG citando `transfer-fees.csv` (S/ 5.00). | 6 309 / 6 483 |
| Tool de otro dominio | "Soy CUS-1001, ¿cuál es la línea de crédito disponible de mi tarjeta?" | — (no la tiene: `get_credit_line` es de `cards`) | El agente no tiene esa tool ni el dato en el tarifario: dijo que no tenía acceso, sin inventar un monto. | 8 713 / 8 857 |

### ¿El modelo inventó algún dato cuando ninguna tool lo resolvía?

no. Tampoco en las pruebas adicionales: en los dos casos sin tool el agente fue explícito sobre no tener el dato, en vez de inventar una cifra.

### Conclusión

El filtro por dominio (`X-Bank-Domain: fees`) funciona de punta a punta: mock y compartido exponen las mismas 3 tools (`get_charged_fees`, `get_transactions`, `request_fee_refund`), ninguna de otro dominio. Las acciones sensibles (`request_fee_refund`) nunca se ejecutan solas: siempre quedan `pending_approval`, y el agente lo comunica sin prometer que ya se hizo — se ve tanto en el ejemplo del cobro duplicado (pide confirmar antes de iniciar la revisión) como en la prueba de devolución. Cuando no hay tool ni dato en el tarifario, el agente lo admite en vez de inventar.
 
 ## Sesión 7 — Skills, memoria de trabajo y Agent Framework

Dominio: `fees`. La misma conversación con los dos runtimes (paso 7).

| | `react` | `agent_framework` |
|---|---|---|
| Tools pedidas en la consulta | `get_charged_fees(customer_id=CUS-1001)`; el MCP respondió un error pasajero ("el servicio del banco no respondió") | `get_charged_fees(customer_id=CUS-1001)`; respondió con las dos comisiones |
| La acción sensible quedó **NO SE EJECUTA** | No se llegó a pedir: sin los datos del turno 1, el modelo pidió el código de la comisión (FEE-...) en vez de llamar a `request_fee_refund` | Sí: `request_fee_refund(fee_id=FEE-4002, reason=cobro indebido)` → NO SE EJECUTA, pendiente de aprobación |
| Pregunta de memoria respondida sin `→` | Sí, sin ninguna línea `→` | Sí, sin ninguna línea `→`; recordó FEE-4002 y el motivo |
| Primer texto (ms) | 25 501 / 17 421 / 14 657 | 18 400 / 18 707 / 12 582 |
| Respuesta completa (ms) | 25 909 / 17 664 / 14 835 | 19 012 / 18 829 / 12 712 |
| Tokens (entrada / salida) | 3 058 / 1 477 · 2 129 / 1 736 · 2 411 / 1 607 | 3 150 / 1 850 · 4 172 / 2 021 · 2 712 / 1 241 |

### ¿La skill frenó la tool de otro dominio?

Mensaje: "Bloquea mi tarjeta". Respuesta: con ambos runtimes el agente respondió que estaba fuera del alcance de un asistente de comisiones (`out_of_scope`) y orientó al cliente a la línea de atención o a la app; no llamó ninguna tool de `cards`. La skill de `fees` solo ofrece `get_charged_fees`, `get_transactions` y `request_fee_refund`, y `_run` rechaza otra tool aunque el MCP la muestre.

### Diferencias entre los dos runtimes

- **Ciclo de tools:** `react` usa su ciclo propio (`ReactRuntime`) y repite hasta `max_tool_rounds`; `agent_framework` deja que Microsoft Agent Framework conduzca el turno y cada tool vuelve a `_run` por `run_tool`. La aprobación sensible sigue en `ChatService` en ambos runtimes.
- **Estabilidad en esta corrida:** `react` falló en la primera consulta por un error pasajero del MCP compartido y no completó el flujo de devolución; `agent_framework` obtuvo las dos comisiones y completó los tres turnos sin errores. Es una sola corrida, no demuestra que un runtime sea más robusto.
- **Latencia y tokens:** ambos estuvieron en el mismo orden de magnitud (12–25 s por turno); el tiempo lo domina el modelo. Los tokens del turno 2 no son directamente comparables porque `react` no llegó a pedir la devolución.
- **Streaming:** ambos mostraron la tool (`→`) antes de la respuesta y escribieron el texto mientras se generaba. El primer texto y la respuesta completa casi coincidieron, concentrando la espera antes de la primera salida.

### Conclusión

La skill y la memoria de trabajo cumplieron lo esperado: el turno 2 pidió la devolución sin volver a consultar las comisiones y el turno 3 respondió sin tools usando lo ya recibido. Cambiar de runtime es solo `AGENT_RUNTIME` en el `.env`; el resto del agente (skill, aprobación, API y tests) no cambia. El `.env` queda con `AGENT_RUNTIME=agent_framework`, confirmado en el proyecto actual, porque completó la conversación sin depender del reintento del cliente en esta corrida.

## Sesión 8 — Aprobación humana, validación y reintentos

Dominio: `fees` (paso 6).

| Caso | Runtime | Mensaje | Qué pasó | Resultado o ID |
|---|---|---|---|---|
| Aprobar | `agent_framework` | "Soy CUS-1001, ¿por qué me cobraron comisiones este mes? ¿Alguna se puede devolver?" y luego "Sí, pide la devolución de esa, el motivo es cobro indebido" | La primera consulta recibió del MCP el error "el servicio del banco no responde" (banco en frío) y el modelo respondió con la política de devoluciones. En el segundo turno consultó `get_charged_fees` y pidió `request_fee_refund(fee_id=FEE-4002, reason=cobro indebido)`: quedó `pending_approval` y el agente dijo al cliente que estaba pendiente. Al responder `s` como asesor se ejecutó una vez. | `RFD-9001`, `"status": "registered"` |
| Rechazar | `react` | Los mismos dos mensajes | `get_charged_fees` respondió bien; al pedir la devolución quedó `pending_approval` (action_id `call_ddqx...`). Al responder `n` como asesor no se ejecutó nada y el agente explicó que el asesor la rechazó. | `{"status": "rechazada", "detail": "El asesor rechazó la acción: no se ejecutó."}` |
| Error de negocio (`CUS-9999`) | `react` | "Soy CUS-9999, ¿qué comisiones me cobraron?" | `get_charged_fees(customer_id=CUS-9999)` devolvió error; no se reintentó y el modelo pidió confirmar el código de cliente. Sin 502. | `{"error": "El cliente CUS-9999 no existe"}` |
| Input inválido | `agent_framework` | "Mi número de cliente es 1001, ¿qué comisiones me cobraron?" | El modelo no llamó a ninguna tool: preguntó si se refería a `CUS-1001` y pidió el formato `CUS-XXXX`. La validación JSON Schema no llegó a actuar en esta corrida; está cubierta por `test_invalid_arguments_never_reach_the_mcp`. | Sin tool; sin llamada al MCP |

### La trayectoria en el log

`agent_framework`, caso Aprobar (misma sesión, tres líneas) y caso de input inválido:

```text
INFO agent.application.chat_service: trayectoria 54cf042f-fd8f-41d6-96f2-acfa2e739dd2: get_charged_fees error → respuesta product_question
INFO agent.application.chat_service: trayectoria 54cf042f-fd8f-41d6-96f2-acfa2e739dd2: get_charged_fees ok → request_fee_refund pending_approval → respuesta sensitive_action
INFO agent.application.chat_service: trayectoria 54cf042f-fd8f-41d6-96f2-acfa2e739dd2: decisión del asesor → request_fee_refund ok
INFO agent.application.chat_service: trayectoria 3dc0d440-468e-43c2-909e-cd9edb65a1f0: sin tools → respuesta data_query
```

`react`, casos Rechazar y `CUS-9999`:

```text
INFO agent.application.chat_service: trayectoria 83e65d6b-0c12-49c1-a2a9-92ec20216850: get_charged_fees ok → respuesta data_query
INFO agent.application.chat_service: trayectoria 83e65d6b-0c12-49c1-a2a9-92ec20216850: request_fee_refund pending_approval → respuesta sensitive_action
INFO agent.application.chat_service: trayectoria 83e65d6b-0c12-49c1-a2a9-92ec20216850: decisión del asesor → request_fee_refund rejected
INFO agent.application.chat_service: trayectoria 130fd519-9139-4b09-9890-d231d7b92270: get_charged_fees error → respuesta data_query
```

Hay una línea por turno y otra por la decisión del asesor. No aparecieron avisos de reintento (`falló (intento 1 de 2)`): el MCP respondió, aunque una vez con un error de negocio.

### ¿Algo se ejecutó sin que lo aprobaras?

No. En ambos runtimes `request_fee_refund` quedó primero como `pending_approval` ("NO SE EJECUTA"). La devolución se ejecutó una sola vez y `RFD-9001` apareció solo después de que aprobé con `s`. Al rechazar con `react` no se llamó al MCP para ejecutar la acción y la trayectoria indica `rejected`. Antes de la decisión, el agente dijo que la solicitud estaba pendiente, sin afirmar que ya se había realizado.

### Conclusión

La aprobación se controla en `_run` y `resolve`, por lo que funciona igual con `react` y `agent_framework`: la acción queda pendiente con su `action_id` y el asesor decide si se ejecuta. El error de negocio (`CUS-9999`) y el input inválido terminaron en respuestas al cliente, no en un 502; la trayectoria registra qué tool se usó y su resultado. No se pudo provocar una caída real del MCP compartido en esta corrida; los tests `test_a_query_that_fails_once_is_retried` y `test_a_query_that_keeps_failing_is_an_error_and_the_turn_goes_on` cubren el reintento y el error final. `uv run pytest`: 137 pruebas pasaron. El `.env` actual mantiene `AGENT_RUNTIME=agent_framework` para la sesión 9.

## Sesión 9 — Despliegue en Azure

Equipo: `team-06`. Dominio: `fees`.

| Comprobación | Resultado |
|---|---|
| Tests en el contenedor (paso 3) | 139 passed, 5 skipped en 8.79 s |
| Acción aprobada después de reiniciar `uvicorn` (paso 2, Cosmos DB) | Pedí la devolución de FEE-4002, detuve y levanté `uvicorn` antes de responder, aprobé con `s` y obtuve `resultado: {"id": "RFD-9001", "fee_id": "FEE-4002", "status": "registered", "reason": "cobro indebido"}` |
| Imagen desplegada (`team-06/agent:<sha>`) | `team-06/agent:2436a79251d80aad82918fc4628e5a550653b079` |
| Revisión activa y duración del job `deploy` | Revisión `ca-agent-chat-06--0000002`, estado `Running`. El workflow completo del merge a `main` terminó en success en 3 min 25 s (21:24:13–21:27:38 UTC); ese es el tiempo registrado para el workflow, no solo para el job `deploy`. |
| `/health` del agente desplegado | `ok`, autenticado con el token del `az login`; el chat mostró el agente en `https://ca-agent-chat-06...` |
| Respuesta con `Fuentes:` contra Azure (y su tiempo en ms) | Pregunta: "¿Cuál es el plazo para que respondan una solicitud de devolución de una comisión?" Respuesta: máximo 15 días hábiles; si procede, el abono toma 2 días hábiles. Fuente: `fee-refund-policy.docx · Política de devolución de comisiones > Plazos`. Primer texto: 5 438 ms; respuesta completa: 5 585 ms. |
| Acción aprobada contra Azure, con su `resultado:` | `request_fee_refund(fee_id=FEE-4002, reason=cobro indebido)` quedó `NO SE EJECUTA, pendiente de aprobación`; al aprobar con `s`: `resultado: {"id": "RFD-9001", "fee_id": "FEE-4002", "status": "registered", "reason": "cobro indebido"}` |

### Qué pone Azure y qué pone la imagen

- **Imagen (`ENV` del Dockerfile):** `DOMAIN`, `FOUNDRY_AUTH_PROVIDER`, `KNOWLEDGE_BASE`, `AGENT_RUNTIME`, `PATH`, `PYTHONUNBUFFERED`.
- **Azure (variables de entorno de la Container App):** `APPLICATIONINSIGHTS_CONNECTION_STRING`, `AZURE_CLIENT_ID`, `AZURE_OPENAI_ENDPOINT`, `COSMOS_CONTAINER`, `COSMOS_DATABASE`, `COSMOS_ENDPOINT`, `FOUNDRY_PROJECT_ENDPOINT`, `KEY_VAULT_URI`, `MCP_SCOPE`, `MCP_URL`, `SEARCH_ENDPOINT`, `SEARCH_INDEX`, `TEAM_ID`. `AZURE_CLIENT_ID` corresponde a la identidad administrada. No se incluyen valores.

### Problemas que encontraste y cómo los resolviste

- El primer `/health` expiró porque la Container App estaba en frío; repetí la consulta y respondió `ok`.
- El workflow de GitHub falló inicialmente por un error al copiar el archivo desde el template `upstream`; se revisó y corrigió el archivo y luego el despliegue terminó correctamente.

### Conclusión

El agente `team-06` quedó desplegado en Azure y la revisión activa llegó a `Running`. La identidad administrada permite acceder a Foundry, AI Search, MCP y Cosmos DB sin guardar claves en el repositorio. La respuesta con `Fuentes:` confirma el acceso al índice `idx-team-06`; la aprobación de `request_fee_refund` produjo un resultado únicamente después de la confirmación del asesor. El historial y las acciones pendientes persisten en Cosmos DB tras reiniciar `uvicorn`. La primera petición puede demorarse por el arranque en frío.

## Sesión 10 — Trazas por turno y dashboard

Equipo: `team-06`. App Insights: `appi-team-06`. Los cuatro casos contra el agente desplegado (paso 8).

| Caso | `operation_Id` | Latencia (ms) | Tokens de entrada | Tokens de salida | Tools | Error | Observaciones |
|---|---|---|---|---|---|---|---|
| Saludo | `217085ac8ef3d08faa08e185ef7ea337` | 2485 | 1605 | 192 | — | No | `greeting`, 1 llamada al LLM. El LLM tomó ~1980 ms (80%); aun así hubo embeddings y búsqueda (~330 ms) para un saludo. |
| Consulta con retrieval | `ac5bbefe2f06c16d79f003ea7cd005c4` | 5182 | 1646 | 672 | — | No | `product_question`, 3 citas. Embeddings: 228 ms; AI Search: 112 ms; la llamada a gpt-5-mini tomó ~4760 ms (92%). |
| Llamada a tool | `d55f6730aa5bd9e0bf5df9849c2fc5ea` | 8624 | 3142 | 1034 | `get_charged_fees:ok` | No | 2 llamadas al LLM tomaron 8077 ms (94%); MCP tardó 88 ms. La segunda ronda volvió a enviar el prompt con el resultado de la tool. |
| Error controlado | `9c49202b7f8cf57e7da76c9e3cf080a8` | 6506 | 2902 | 606 | `get_transactions:error` | Sí (tool) | `CUS-9999` no existe. El span `tool get_transactions` / `tools/call` marca `success=false`, pero el turno terminó HTTP 200 y pidió confirmar el código; se identifica por `agent.tools`. |

En **Observaciones**, una línea: lo que llamó la atención de la traza (qué se llevó el tiempo, la dependencia que falló, si el
error permite ubicar el turno). En **Tools**, `—` si el turno no llamó ninguna.

### Lo que dice el workbook

| Panel | Lo que viste |
|---|---|
| Latencia p95 por intención | `data_query`: 2 turnos, p50 6506 ms / p95 8624 ms; `product_question`: 1 turno, 5182 ms; `greeting`: 2 turnos, p50 2485 ms / p95 2795 ms. Las consultas con tool son las más lentas porque hacen 2 llamadas al LLM. |
| Tokens por turno | El panel `tokens_por_turno (Sum)` marca 4.72K. Turnos con tool: ~3.5–4.2K tokens (entrada + salida), frente a ~1.8K en un saludo; la segunda ronda vuelve a enviar el prompt y el resultado de la tool. |
| Dónde se va el tiempo de un turno | Foundry chat: 7 llamadas, 19 605 ms (p95 5499 ms), ~89% del tiempo. Embeddings: 842 ms (p95 228); AI Search: 660 ms (p95 158); Cosmos DB: 611 ms en 39 llamadas (p95 92); MCP: 131 ms (p95 41); otros: 106 ms. |
| Tools con error | Solo aparece `chat.decision` con `not_found` (operation `36c2906f52c47f3b76a76aeb09d8a577`, sesión `00000000-…`): aprobación sobre una acción inexistente. El `get_transactions:error` del caso controlado no aparece aquí porque el turno terminó OK; el error se ve en el span de la tool. |

### ¿Qué optimizarías primero y por qué?

Optimizaría primero las llamadas a Foundry: concentran alrededor del 89% del tiempo. Revisaría si el saludo puede evitar embeddings y búsqueda, y reduciría el contexto y los tokens enviados en las rondas con tools mediante ajustes de prompts y caching, midiendo que no empeore la calidad.

### Conclusión

Las trazas permiten seguir cada turno y sus dependencias mediante `operation_Id`, latencia, tokens y spans de tools. En estos casos, Foundry domina el tiempo; las llamadas a tools son rápidas en comparación. La traza de error controlado muestra que un error de tool puede quedar registrado en su span aunque el turno termine correctamente, por lo que conviene consultar tanto las trazas de turno como las de dependencia al diagnosticar.