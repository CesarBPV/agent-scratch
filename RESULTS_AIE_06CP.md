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
