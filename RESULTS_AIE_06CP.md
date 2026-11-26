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

# RESULTADOS AIE 06CP

Dominio `fees` (Tarifario) · Equipo 06 · Sesión 5

## Mi caso del golden set

Archivo: `evals/golden/fees.AIE_06CP.jsonl` (id `fees-AIE06CP`)

| Campo | Valor |
|---|---|
| Pregunta | ¿Cuánto cuesta reponer una tarjeta de crédito si me la robaron? |
| Tipo | `answerable` |
| Intención esperada | `product_question` |
| Fuente esperada | `credit-cards-fees.pdf` |
| Hecho que debe decir | `primera reposición` |

### Por qué lo elegí
El golden set base no pregunta por la reposición de tarjeta. Es un caso de **regla y excepción** en el mismo párrafo del tarifario: la reposición por pérdida o deterioro cuesta S/ 20, pero por robo o fraude la primera reposición del año no tiene costo. Un cliente real que pregunta "me la robaron" necesita la excepción, no la tarifa general.

Es el mismo riesgo que se observó en la sesión 3 al bajar `INGEST_CHUNK_MAX_CHARS` a 250 (ver `RESULTS.md`, sección 6): con chunks pequeños la regla (S/ 20) y su excepción (robo o fraude) quedan en chunks distintos, y si la búsqueda trae solo el primero, el agente respondería S/ 20 de forma incorrecta. El caso sirve para detectar esa regresión si se cambia el chunking.

### Cómo le fue al agente
Corrida: `uv run eval --case fees-AIE06CP` (juez `local`, base `azure_search` en `idx-team-06`, `SEARCH_TOP_K=5`, `INGEST_CHUNK_MAX_CHARS=1500`).

| Control | Resultado |
|---|---|
| recall@5 | ok |
| intención | ok (`product_question`) |
| fuentes | ok (citó `credit-cards-fees.pdf`) |
| hechos | ok |
| apoyo | 5.0 / 5 |
| relevancia | 5.0 / 5 |
| cita correcta | 1.0 |
| fugas de datos personales | 0 |
| citas de fuentes prohibidas | 0 |

Tokens: 1088 de entrada, 259 de salida.

Respuesta del agente:

> Según el tarifario de tarjetas de crédito, la reposición por pérdida o deterioro cuesta S/20. Si la reposición es por robo o fraude, la primera reposición del año no tiene costo.

Fuentes recuperadas (en orden): `credit-cards-fees.pdf`, `undue-charges-cases.pdf`, `fee-refund-policy.docx` (dos fragmentos), `savings-accounts-fees.html`. Solo la primera era necesaria; el modelo citó únicamente esa.

### Límites de esta medición
- Una sola corrida y un solo caso: no es evidencia de la calidad general del agente.
- No se probó el caso con chunks de 250 caracteres; la hipótesis de que detecta esa regresión está sin comprobar.

## Línea base (S5)

### Resultado final

Ambos archivos ya quedaron corregidos a un caso propio cada uno (`fees.AIE_17GM.jsonl` tenía el mismo problema: era un duplicado exacto de `fees-01`, se reemplazó por un caso nuevo). Corrida contra Foundry real (`gpt-5-nano`) y AI Search real (`idx-team-06`, 29 chunks, `SEARCH_TOP_K=5`), juez local (G-Eval, `gpt-5-mini`). 33 casos: 30 del golden set base + 1 por integrante. Detalle: `data/eval/fees-20261116-172231.json`.

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

# Sesión 6 — Tools del MCP en el chat

Dominio: `fees`. MCP: el mock (`http://127.0.0.1:8200/mcp`) y el compartido (`bank-mcp`, `MCP_URL` del `.env` apuntando a `https://ca-mcp-server.../mcp`). El MCP decide qué tools ve el agente por el header `X-Bank-Domain` (`settings.domain = fees`); eso ya lo manda el código (`mcp_tools.py`), no hubo que tocar nada para enviarlo.

### Tools de tu dominio

| MCP | Tools que lista `dev/mcp/mcp_tools.py` |
|---|---|
| Mock | `get_charged_fees` (consulta), `get_transactions` (consulta), `request_fee_refund` (acción sensible) |
| Compartido | `get_charged_fees` (consulta), `get_transactions` (consulta), `request_fee_refund` (acción sensible) — idénticas al mock |

### Conversación en `dev/chat.py` (paso 12)

Contra el agente real (Foundry + AI Search + MCP compartido, `idx-team-06`).

| Prueba | Mensaje | Tool pedida (`→`) | Qué pasó | Primer texto / respuesta completa (ms) |
|---|---|---|---|---|
| Consulta de tu dominio | "Soy CUS-1001, ¿qué comisiones me han cobrado este mes?" | `get_charged_fees(customer_id=CUS-1001)` | Resultado correcto: listó `FEE-4001` (Retiro en cajero de otro banco, S/ 6.50, no reembolsable) y `FEE-4002` (Mantenimiento de cuenta, S/ 12.00, reembolsable), igual que `bank_data.py`. En la primera corrida el mock del banco devolvió un error simulado ("El servicio del banco no responde") y el agente lo manejó bien, avisando al cliente sin inventar datos; al reintentar funcionó. | 14 095 / 14 247 |
| Acción confirmada | "Soy CUS-1001, quiero que me devuelvan la comisión de mantenimiento de cuenta, la FEE-4002, porque no mantuve el saldo mínimo y no me avisaron" | `request_fee_refund(fee_id=FEE-4002, reason=...)` | `pending_approval`: la tool no se ejecuta, queda pendiente de un asesor. El agente lo dijo así ("está pendiente de aprobación... no se ejecuta de inmediato") y no afirmó que la devolución ya se hizo; además citó la política de devolución (RAG) para dar plazos (15 días hábiles, abono en 2). | 50 301 / 50 575 |
| Sin tool que la resuelva | "¿Cuánto cuesta una transferencia interbancaria inmediata?" | ninguna | Pregunta general del tarifario, no de un cliente puntual: no llamó ninguna tool, respondió solo con RAG citando `transfer-fees.csv` (S/ 5.00). Correcto: una tool no tiene sentido aquí. | 6 309 / 6 483 |
| Tool de otro dominio | "Soy CUS-1001, ¿cuál es la línea de crédito disponible de mi tarjeta?" | — (no la tiene: `get_credit_line` es de `cards`) | El agente no tiene esa tool (el MCP solo le muestra las de `fees`) ni el dato en el tarifario: dijo que no tenía acceso y sugirió la app o un asesor, sin inventar un monto. | 8 713 / 8 857 |

### ¿El modelo inventó algún dato cuando ninguna tool lo resolvía?

No. En los dos casos sin tool (sin tool que la resuelva y tool de otro dominio) el agente fue explícito sobre no tener el dato, en vez de inventar una cifra. Tampoco inventó nada cuando la tool de comisiones falló transitoriamente en la primera corrida: avisó del error y ofreció reintentar.

### Conclusión

El filtro por dominio (`X-Bank-Domain: fees`) funciona de punta a punta: mock y compartido exponen exactamente las mismas 3 tools (`get_charged_fees`, `get_transactions`, `request_fee_refund`), ninguna de otro dominio. Las acciones sensibles (`request_fee_refund`) nunca se ejecutan solas: siempre quedan `pending_approval`, y el agente lo comunica sin prometer que ya se hizo. Cuando no hay tool ni dato en el tarifario, el agente lo admite en vez de inventar — el riesgo de alucinación que se había anotado en la sesión 1 (sección 3 de `RESULTS.md`, "promete algo que no puede hacer") ya no se repite ahora que existen las tools reales.


## Sesión 7 — Skills, memoria de trabajo y Agent Framework

Dominio: `fees`. La misma conversación con los dos runtimes (paso 7), contra Foundry real y el MCP compartido, cada runtime en su propio puerto y verificando en el log cuál estaba activo (`Agent runtime: react` / `Agent runtime: agent_framework`). Conversación: (1) "Soy CUS-1001, ¿qué comisiones me cobraron este mes?", (2) "Pide la devolución de esa, el motivo es cobro indebido", (3) pregunta de memoria sobre el dato ya traído, sin tool. Cada celda trae los tres turnos.

| | `react` | `agent_framework` |
|---|---|---|
| Tools pedidas en la consulta | `get_charged_fees(customer_id=CUS-1001)`; el MCP respondió un error pasajero ("el servicio del banco no respondió") | `get_charged_fees(customer_id=CUS-1001)`; respondió con las dos comisiones |
| La acción sensible quedó **NO SE EJECUTA** | No se llegó a pedir: sin los datos del turno 1, el modelo pidió el código de la comisión (FEE-...) en vez de llamar a `request_fee_refund` | Sí: `request_fee_refund(fee_id=FEE-4002, reason=cobro indebido)` → NO SE EJECUTA, pendiente de aprobación |
| Pregunta de memoria respondida sin `→` | Sí, sin ninguna línea `→` | Sí, sin ninguna línea `→`; recordó FEE-4002 y el motivo |
| Primer texto (ms) | 25 501 / 17 421 / 14 657 | 18 400 / 18 707 / 12 582 |
| Respuesta completa (ms) | 25 909 / 17 664 / 14 835 | 19 012 / 18 829 / 12 712 |
| Tokens (entrada / salida) | 3 058 / 1 477 · 2 129 / 1 736 · 2 411 / 1 607 | 3 150 / 1 850 · 4 172 / 2 021 · 2 712 / 1 241 |

### ¿La skill frenó la tool de otro dominio?

Mensaje: "Bloquea mi tarjeta". Respuesta: con ambos runtimes el agente la trató como `out_of_scope` ("eso está fuera del alcance de este asistente de comisiones") y orientó al cliente a la línea de atención o a la app; no llamó ninguna tool de `cards`. La skill de `fees` solo le ofrece `get_charged_fees`, `get_transactions` y `request_fee_refund`, y `_run` rechaza cualquier otra aunque el MCP la mostrara.

### Diferencias entre los dos runtimes

- **Ciclo de tools:** `react` es el ciclo propio (`ReactRuntime`: pide, ejecuta, repite hasta `max_tool_rounds`); `agent_framework` deja que Microsoft Agent Framework conduzca el turno y cada tool vuelve a `_run` por `run_tool`. La aprobación de las acciones sensibles sigue siendo del `ChatService` en los dos, por eso la acción quedó `NO SE EJECUTA` donde llegó a pedirse.
- **Estabilidad en esta corrida:** `react` falló en la consulta por un error pasajero del MCP compartido (servicio del banco en frío, igual que en la sesión 6) y eso arrastró el resto de la conversación; `agent_framework`, con la misma configuración, obtuvo los datos y completó la conversación (3 llamadas, 0 errores en el log). Es una sola corrida: no prueba que un runtime sea más robusto que el otro, solo que el error vino del MCP y no del runtime. No se repitió la corrida de `react`.
- **Latencia y tokens:** del mismo orden (12–25 s por turno); el tiempo lo domina el modelo, no el runtime. Los tokens del turno 2 de `agent_framework` son mayores (4 172 vs 2 129) porque en `react` el turno no ejecutó la acción, así que no son comparables turno a turno.
- **Streaming:** los dos muestran la tool (`→`) antes de la respuesta y el texto llega mientras se escribe; el primer texto y la respuesta completa casi coinciden (la diferencia es de décimas de segundo), es decir, la espera se concentra antes de que aparezca el primer texto.

### Conclusión

La skill y la memoria de trabajo cumplen lo esperado: el turno 2 pidió la devolución sin volver a consultar las comisiones y el turno 3 se respondió sin tools con lo que ya había en la memoria. Cambiar de runtime es solo `AGENT_RUNTIME` en el `.env`; el resto del agente (reglas, aprobación, API, tests) no cambia, que es la razón de haber dejado `_run` como única puerta a las tools. Para la sesión 8 el `.env` queda con `AGENT_RUNTIME=agent_framework`, porque en esta corrida fue el que completó la conversación sin depender del reintento del cliente.


## Sesión 8 — Aprobación humana, validación y reintentos

Dominio: `fees` (paso 6). Foundry real y el MCP compartido, cada runtime levantado por separado (verificado en el log con `Agent runtime: ...`). Las conversaciones las hice en `dev/chat.py`; cuando el chat preguntó `Asesor: ¿apruebas ...? [s/n]`, actué como asesor y respondí `s` para aprobar o `n` para rechazar.

| Caso | Runtime | Mensaje | Qué pasó | Resultado o ID |
|---|---|---|---|---|
| Aprobar | `agent_framework` | "Soy CUS-1001, ¿por qué me cobraron comisiones este mes? ¿Alguna se puede devolver?" y luego "Sí, pide la devolución de esa, el motivo es cobro indebido" | La primera consulta recibió del MCP el error "el servicio del banco no responde" (banco en frío) y el modelo respondió con la política de devoluciones. En el segundo turno consultó `get_charged_fees` y pidió `request_fee_refund(fee_id=FEE-4002, reason=cobro indebido)`: quedó `pending_approval` y el agente dijo al cliente que estaba pendiente. Al responder `s` como asesor se ejecutó una vez. | `RFD-9001`, `"status": "registered"` |
| Rechazar | `react` | Los mismos dos mensajes | `get_charged_fees` respondió bien; al pedir la devolución quedó `pending_approval` (action_id `call_ddqx...`). Al responder `n` como asesor no se ejecutó nada y el agente explicó que el asesor la rechazó. | `{"status": "rechazada", "detail": "El asesor rechazó la acción: no se ejecutó."}` |
| Error de negocio (`CUS-9999`) | `react` | "Soy CUS-9999, ¿qué comisiones me cobraron?" | `get_charged_fees(customer_id=CUS-9999)` devolvió error; no se reintentó y el modelo pidió confirmar el código de cliente. Sin 502. | `{"error": "El cliente CUS-9999 no existe"}` |
| Input inválido | `agent_framework` | "Mi número de cliente es 1001, ¿qué comisiones me cobraron?" | El modelo no llamó a ninguna tool: preguntó si se refería a `CUS-1001` y pidió el formato `CUS-XXXX`, porque lee el patrón en la descripción de la tool. La validación contra el JSON Schema no llegó a actuar en esta corrida; la cubre `test_invalid_arguments_never_reach_the_mcp`. | Sin tool; sin llamada al MCP |

### La trayectoria en el log

`agent_framework`, caso Aprobar (misma sesión, tres líneas) y el caso de input inválido:

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

Hay una línea por turno y la de la decisión del asesor; no aparecieron avisos de reintento (`falló (intento 1 de 2)`) porque el MCP respondió, aunque una vez con un error de negocio.

### ¿Algo se ejecutó sin que lo aprobaras?

No. En los dos runtimes `request_fee_refund` salió siempre como `pending_approval` ("NO SE EJECUTA") y el resultado real (`RFD-9001`) apareció solo después de mi `s`. Al rechazar con `react` no hubo ninguna llamada de ejecución al MCP y la trayectoria lo registra como `rejected`. En ambos casos el texto del modelo antes de mi decisión dijo que la solicitud quedaba pendiente de aprobación, sin afirmar que ya se había hecho.

### Conclusión

La aprobación vive en `_run` y `resolve`, por eso se comporta igual con `react` y con `agent_framework`: la acción queda pendiente con su `action_id`, el asesor decide una vez y el agente le cuenta al cliente lo que pasó. Un error de negocio (`CUS-9999`) y un input mal formado terminan en una respuesta al cliente, no en un 502, y la trayectoria deja en una línea qué tool se usó y cómo terminó cada una. Pendiente de esta corrida: la caída real del MCP no se pudo provocar (es compartido), así que el reintento y el error final lo respaldan los tests `test_a_query_that_fails_once_is_retried` y `test_a_query_that_keeps_failing_is_an_error_and_the_turn_goes_on`; `uv run pytest` → 137 passed. El `.env` queda con `AGENT_RUNTIME=agent_framework` para la sesión 9.


## Sesión 9 — Despliegue en Azure

Equipo: `team-06`. Dominio: `fees`.

| Comprobación | Resultado |
|---|---|
| Tests en el contenedor (paso 3) | 139 passed, 5 skipped in 8.79s |
| Acción aprobada después de reiniciar `uvicorn` (paso 2, Cosmos DB) | Pedí la devolución de FEE-4002, detuve y levanté uvicorn antes de responder, respondí `s` y salió `resultado: {"id": "RFD-9001", "fee_id": "FEE-4002", "status": "registered", "reason": "cobro indebido"}` |
| Imagen desplegada (`team-NN/agent:<sha>`) | `team-06/agent:2436a79251d80aad82918fc4628e5a550653b079`  |
| Revisión activa y duración del job `deploy` | Revisión `ca-agent-chat-06--0000002`, `Running`. El workflow completo del merge a `main` terminó en success en 3 min 25 s (21:24:13 → 21:27:38 UTC) |
| `/health` del agente desplegado | `ok`, con token de mi `az login` (primera línea del chat: `Agente en https://ca-agent-chat-06...: ok`) |
| Respuesta con `Fuentes:` contra Azure (y su tiempo en ms) | "¿Cuál es el plazo para que respondan una solicitud de devolución de una comisión?" → "máximo 15 días hábiles; si procede, abono en 2 días hábiles". Fuente: `fee-refund-policy.docx · Política de devolución de comisiones > Plazos`. Primer texto 5 438 ms, respuesta completa 5 585 ms |
| Acción aprobada contra Azure, con su `resultado:` | `request_fee_refund(fee_id=FEE-4002, reason=cobro indebido)` quedó `NO SE EJECUTA, pendiente de aprobación`; al responder `s`: `resultado: {"id": "RFD-9001", "fee_id": "FEE-4002", "status": "registered", "reason": "cobro indebido"}` |

### Qué pone Azure y qué pone la imagen

- **La imagen (`ENV` del Dockerfile):** `DOMAIN`, `FOUNDRY_AUTH_PROVIDER` (`entra_id`), `KNOWLEDGE_BASE` (`azure_search`) y `AGENT_RUNTIME` (`agent_framework`), más `PATH` y `PYTHONUNBUFFERED`. Lo que no cambia entre entornos.
- **Azure (variables de la Container App):** `APPLICATIONINSIGHTS_CONNECTION_STRING`, `AZURE_CLIENT_ID`, `AZURE_OPENAI_ENDPOINT`, `COSMOS_CONTAINER`, `COSMOS_DATABASE`, `COSMOS_ENDPOINT`, `FOUNDRY_PROJECT_ENDPOINT`, `KEY_VAULT_URI`, `MCP_SCOPE`, `MCP_URL`, `SEARCH_ENDPOINT`, `SEARCH_INDEX`, `TEAM_ID`. Son endpoints y nombres de recursos; ninguna guarda una api key ni una contraseña. `AZURE_CLIENT_ID` es el id de la identidad administrada, que es la que entra a Foundry, AI Search, el MCP y Cosmos DB.

### Problemas que encontraste y cómo los resolviste

- **Primer `/health` con timeout:** la primera vez el chat se cortó con `TimeoutError` en el health (30 s), porque la Container App estaba en frío. Repetí el comando y respondió `ok`; después las respuestas tardaron entre 5 y 22 s por turno. Durante el despliegue desde `main` tuvimos un problema con el workflow de GitHub porque nos equivocamos al copiarlo del template (`upstream`); lo resolvimos copiando correctamente el archivo.

### Conclusión

El agente del equipo corre en Azure con identidad administrada: el repo no guarda secretos, `az login` solo identifica a quien conversa, y el mismo flujo de la sesión 8 (acción pendiente, aprobación del asesor, resultado) funciona contra el MCP compartido desde la Container App. Las respuestas con `Fuentes:` prueban que lee `idx-team-06` en AI Search, y el historial y la acción pendiente viven en Cosmos DB, no en la memoria del proceso. El primer turno puede tardar por el arranque en frío, y cada merge a `main` despliega, así que el cambio de código y el despliegue son uno solo.

## Sesión 10 — Trazas por turno y dashboard

Equipo: `team-06`. App Insights: `appi-team-06`. Los cuatro casos contra el agente desplegado (paso 8).

| Caso | `operation_Id` | Latencia (ms) | Tokens de entrada | Tokens de salida | Tools | Error | Observaciones |
|---|---|---|---|---|---|---|---|
| Saludo | `e58537e5104267fce5ced890606345dc` | 2 808 | 1 629 | 189 | — | — | Intención `greeting`, una sola llamada a `gpt-5-mini` (1 713 ms de los 2 808). Aunque es un saludo, el turno igual hace embeddings y busca en AI Search (~420 ms). La sesión era nueva, así que las lecturas a Cosmos DB salen 404 |
| Consulta con retrieval | `c64b36143e1ab44517968a9c79a26701` | 5 074 | 1 634 | 575 | — | — | "¿Qué es el ITF y cuándo se cobra?" → `product_question` con 3 citas de `itf-tax.md`. Dentro del turno: `POST /embeddings` (226 ms) y `POST /indexes('idx-team-06')/docs/search` (94 ms). El modelo se lleva 3 802 ms (75 %) |
| Llamada a tool | `91c6ed8fc43c7688cce51cfabf194345` | 9 361 | 3 266 | 1 145 | `get_charged_fees:ok` | — | CUS-1001 → FEE-4001 y FEE-4002. `tool get_charged_fees` tarda 98 ms y su `POST /mcp` 39 ms. Son dos llamadas al modelo (6 578 + 1 103 ms): casi todo el tiempo es del LLM, no del MCP |
| Error controlado | `c58472aa35b5a08b8738a9e3eafa75a6` | 5 | — | — | — | `not_found` | `az rest` a `/chat/actions` con `call-inventada` → 404 "no hay una acción pendiente call-inventada en esta sesión". `chat.decision` sale `success=false` en 5 ms, con un único `GET` a Cosmos DB (404) y sin llamar al modelo. El `operation_Id` y `agent.error` permiten ubicar el turno en "Turnos con error" |

En **Observaciones**, una línea: lo que llamó la atención de la traza (qué se llevó el tiempo, la dependencia que falló, si el
error permite ubicar el turno). En **Tools**, `—` si el turno no llamó ninguna.

Primer intento del caso de tool: `5df08973957c3d4566b4d3725a4ec563` (CUS-1001, 21:00:33 UTC), 16 237 ms, `get_charged_fees:error`. El `POST /mcp` tardó 10 022 ms y la tool terminó en error ("El servicio del banco no responde"). El turno no falló: el agente le avisó al cliente y le ofreció reintentar. Al repetirlo a las 21:01 respondió en 98 ms.

### Lo que dice el workbook

Workbook `Turnos del agente`, periodo de 4 h (incluye pruebas anteriores del mismo día, 15 turnos en total).

| Panel | Lo que viste |
|---|---|
| Latencia p95 por intención | `data_query` p50 9 361 ms / p95 16 237 ms (6 turnos), `product_question` 5 074 / 5 182 ms (2), `greeting` 2 779 / 2 808 ms (4). Los turnos con tool cuestan entre 3 y 6 veces lo que un saludo, y el p95 de `data_query` lo marca el turno en el que el MCP no respondió |
| Tokens por turno | De 1 772 a 3 253 tokens por turno según la franja de 15 min, con 1 a 1,6 llamadas al LLM por turno. Un turno con tool consume unos 3 200 de entrada frente a unos 1 600 de un saludo: es la segunda llamada al modelo con el resultado de la tool |
| Dónde se va el tiempo de un turno | Foundry chat: 52 059 ms en 17 llamadas (p95 7 112 ms), muy por encima del resto. Luego MCP con 10 341 ms (casi todo el timeout de 10 s), embeddings 3 780 ms, Cosmos DB 2 604 ms en 95 llamadas (p95 135 ms) y AI Search 1 788 ms (p95 363 ms) |
| Tools con error | `get_charged_fees` `error` 1 vez (p95 10 102 ms, el timeout del MCP) y `get_transactions` `error` 1 vez (88 ms, de una prueba anterior). `get_charged_fees` `ok` 3 veces con p95 de 98 ms. En "Turnos con error" solo aparecen los `chat.decision` `not_found` del error controlado |

### ¿Qué optimizarías primero y por qué?

La llamada al modelo de chat. Se lleva entre el 60 y el 75 % de cada turno, y en los turnos con tool se paga dos veces: la del plan y la de la respuesta con el resultado. Lo primero sería recortar lo que va en el prompt (el historial, los 5 fragmentos de `SEARCH_TOP_K` y las descripciones de las tools) y no hacer retrieval cuando la intención es `greeting`, porque hoy un saludo también paga embeddings y AI Search. En segundo lugar, bajar el timeout del MCP: cuando falla, el cliente espera 10 s para recibir un error. Cosmos DB y AI Search ya están por debajo de 150 ms en p95 y no son el cuello de botella.

### Conclusión

Con `APPLICATIONINSIGHTS_CONNECTION_STRING` y la identidad administrada, cada turno del agente desplegado queda en `appi-team-06` como un `chat.turn` (o `chat.decision`) con su intención, sus tokens y sus tools. Con el `operation_Id` se ve el árbol completo: modelo, embeddings, AI Search, Cosmos DB y `POST /mcp`, cada uno con su duración. Los cuatro casos se distinguen sin leer logs: el saludo no tiene tools, la consulta muestra la búsqueda, la tool muestra su `POST /mcp` y el error controlado sale en rojo con `agent.error = not_found`. La traza también dejó ver un problema que en la consola solo parecía lentitud: el MCP compartido tardó 10 s y falló una vez.