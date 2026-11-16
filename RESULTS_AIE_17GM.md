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

## Línea base (paso 9)

Corrida contra Foundry real (`gpt-5-nano`) y AI Search real (`idx-team-06`, 29 chunks, `SEARCH_TOP_K=5`), juez local (G-Eval, `gpt-5-mini`). 33 casos: 30 del golden set base + 1 por integrante (06CP, 17GM, 40CP). Detalle: `data/eval/fees-20261116-172231.json`.

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

### Casos con NO

- **Hechos en NO (16 de 33 casos), con recall y fuentes en ok**: no es un problema de recuperación ni de contexto. `gpt-5-nano` casi siempre escribe los montos en palabras ("doscientos cuarenta soles", "cinco milésimas por ciento") en vez del formato exacto del golden set ("S/ 240", "0,005 %"), y el chequeo de hechos busca el texto literal. El apoyo (4.9/5) y la relevancia (4.8/5) confirman que el contenido es correcto; el juez (G-Eval) lo reconoce explícitamente en varios casos (p. ej. en `fees-AIE17GM`: "coinciden exactamente con la fila... que fue citada"). Mejora: pedir en el prompt que los montos se escriban en el formato `S/ X.XX` / `X %` tal como aparecen en el tarifario — ayuda también al cliente real, que escanea el número, no la redacción.
- **Intención mal clasificada, con fuente citada de más (`fees-27`, reproducible en 2 corridas)**: "¿A cuánto está el dólar hoy?" se espera `out_of_scope` (igual que `fees-28`, que sí sale bien), pero el agente responde `product_question` citando `atm-network.html` ("El tipo de cambio vigente se aplica al día de la operación..."). No hay fuga ni fuente prohibida, pero sí una cita que no debería existir y una respuesta vaga que no es un hecho del tarifario. Mejora: reforzar en el prompt que las preguntas de tipo de cambio son `out_of_scope` aunque mencionen "cajero" o "banco".
- **Fuente prohibida citada, de forma intermitente (`fees-25`, tipo `noise`)**: en la primera corrida de esta sesión (antes de corregir mi caso del golden set), "¿Cómo reservo estacionamiento en la sede central?" se clasificó bien como `out_of_scope` pero igual citó `internal-notice-parking.md` (fuente prohibida) y usó su contenido. En la corrida final (la de la tabla de arriba) no volvió a pasar: mismo `.env`, mismo índice, pero `gpt-5-nano` no es determinístico. Riesgo latente, no resuelto: a revisar por qué `internal-notice-parking.md` está en el índice del dominio fees.
- **Recall en NO**: ninguno en ninguna de las corridas (`recall@5 = 1.00`); no aplica esta sesión.
- **FUGA de datos personales**: 0 en ambas corridas; no aplica.

## Mi caso del golden set

Archivo: `evals/golden/fees.AIE_17GM.jsonl` (id `fees-AIE17GM`)

| Campo | Valor |
|---|---|
| Pregunta | ¿Cuál era la comisión por una transferencia interbancaria inmediata en el año 2024? |
| Tipo | `expired` |
| Intención esperada | `product_question` |
| Fuente esperada | `transfer-fees.csv` |
| Fuente prohibida | `transfer-fees-2024.csv` |
| Hecho que debe decir | `S/ 5.00` |

### Por qué lo elegí
El caso que había puesto primero (`fees-AIE17GM` original) resultó ser un **duplicado exacto de `fees-01`** del golden set base (misma pregunta, mismas fuentes, mismo hecho) — no agregaba cobertura nueva, así que lo reemplacé por este.

Lo elegí porque reproduce un hallazgo real de la sesión 4 (ver "Comparación local frente a AI Search" arriba): al preguntar explícitamente por una tarifa "en el 2024", el filtro de vigencia excluye bien `transfer-fees-2024.csv`, pero el agente igual responde con la tarifa **vigente** (`transfer-fees.csv`) presentándola como si fuera la de 2024, sin aclarar que no tiene el histórico. El golden set base tiene casos `expired` parecidos (`fees-22` a `fees-24`), pero ninguno menciona el año explícitamente en la pregunta — esa mención es justo lo que puede tentar al retrieval o al modelo a tratarla como una consulta histórica real.

### Cómo le fue al agente
Recall, intención, fuentes y cita: todos `ok` (citó `transfer-fees.csv`, no el prohibido). Apoyo 5.0/5, relevancia 5.0/5. `facts_ok` salió `NO` por el mismo patrón de formato que domina el resto de la línea base: la respuesta dice "cinco soles" en vez de "S/ 5.00". Más allá de la métrica, la respuesta tampoco aclara que es la tarifa vigente y no un dato histórico de 2024 — el mismo matiz que motivó el caso, confirmado ahora con un test reproducible.

## Antes y después (paso 10)

Un cambio a la vez, siempre desde la línea base (`INGEST_CHUNK_MAX_CHARS=1500`, `SEARCH_TOP_K=5`, `KNOWLEDGE_BASE=azure_search`). 33 casos en todas las corridas.

| Cambio | recall | intención | fuentes | hechos | apoyo | relevancia | cita | tokens ent. / sal. | fuentes prohibidas |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Línea base (K=5, 1500, azure_search) | 1.00 | 0.97 | 0.97 | 0.50 | 4.9 | 4.8 | 1.00 | 45 353 / 51 030 | 0 |
| Chunks más chicos (800) | 1.00 | 0.94 | 0.94 | 0.43 | 5.0 | 4.6 | 1.00 | 45 361 / 50 208 | 1 |
| Menos fragmentos (K=3) | 1.00 | 0.97 | 0.97 | 0.37 | 4.9 | 4.6 | 1.00 | 40 477 / 48 539 | 0 |
| Más fragmentos (K=8) | 1.00 | 0.97 | 0.97 | 0.43 | 4.9 | 4.9 | 1.00 | 52 828 / 47 004 | 0 |
| Base local, sin reranker | 0.90 | 1.00 | 0.91 | 0.43 | 4.7 | 4.6 | 0.90 | 43 973 / 45 260 | 0 |

Lectura:
- **Chunks más chicos (800)**: con este corpus es un cambio que no cambia nada — `uv run ingest build` con 800 sigue dando **29 chunks**, idénticos a los de 1500 (las secciones ya son más cortas que 800 caracteres; ver también sección 6 de `RESULTS.md`). Las diferencias en la tabla (hechos 0.43 vs 0.50, 1 fuente prohibida) son ruido entre corridas de `gpt-5-nano` —el mismo fenómeno intermitente de `fees-25` que ya se vio en la línea base—, no un efecto del cambio.
- **Menos fragmentos (K=3)**: bajó los tokens de entrada (−11 % frente a K=5) pero también `hechos` (0.37, el más bajo de la tabla): con menos contexto, algunos hechos puntuales (como los 3 sub-datos del ITF, repartidos en varios chunks) se pierden.
- **Más fragmentos (K=8)**: no mejoró `apoyo` ni `hechos` de forma clara frente a K=5, y subió los tokens de entrada un 16 %. No trajo más fuentes prohibidas en esta corrida. No se justifica el costo extra.
- **Base local, sin reranker**: es el único cambio que bajó `recall` (0.90, el único por debajo de 1.00 en toda la sesión) y `cita correcta` (0.90): sin el reranking semántico de AI Search, al menos un caso no trajo el documento esperado entre los 5 fragmentos. Confirma lo que ya se había visto en la sesión 4 (sección "Comparación local frente a AI Search"): el reranker de AI Search aporta, no es solo un detalle de infraestructura.

### Qué elige el equipo
Se mantiene la configuración actual: `INGEST_CHUNK_MAX_CHARS=1500`, `SEARCH_TOP_K=5`, `KNOWLEDGE_BASE=azure_search`. Es la única que llega a `recall=1.00` y a la mejor `cita correcta` (1.00); K=3 y K=8 no mejoran `hechos` ni `apoyo` lo suficiente como para justificar bajar cobertura o subir costo, y local pierde recall por no tener reranker. Al cerrar las pruebas se reconstruyó el índice con estos valores (`uv run ingest build` + `uv run ingest publish`, confirmado "29 chunks en ... idx-team-06") y se dejaron las variables de entorno sin overrides, iguales a las del `.env` del equipo.

---

# RESULTADOS AIE 17GM

Evaluación consultada: `data/eval/fees-20261111-204025.json`

## Cumplimiento de metas del tema 2

- [x] Control MetaRecall@k ≥ 0.80 — se cumple (`recall_at_k = 1.00`)
- [ ] Hechos ≥ 0.75 — no se cumple (`facts_ok = 0.5714`)
- [x] Apoyo ≥ 4.0 — se cumple (`groundedness = 4.98`)
- [x] Fugas de datos personales = 0 — se cumple (`personal_data_leaks = 0`)
- [x] Citas de fuentes prohibidas = 0 — se cumple (`forbidden_citations = 0`)

## Qué cambiaría para que sí se cumpla todo

La única meta que no se cumple es la de hechos. Para subir `facts_ok` por encima de 0.75, haría estos cambios:

1. Añadir una validación post-respuesta de hechos antes de devolver la respuesta final.
2. Forzar que cada afirmación responda a una cita directa y verificable en la fuente recuperada.
3. Mejorar la recuperación para evitar mezclar información de varias fuentes o documentos no pertinentes.
4. Rechazar respuestas con afirmaciones ambiguas o parcialmente apoyadas y pedir un reintento con una respuesta más estricta.
5. Incluir un paso de comprobación de “solo hechos respaldados” antes de cerrar la sesión.

Con esto, mantendría el buen nivel actual de recall, apoyo y seguridad, y corregiría la principal brecha: la precisión factual.
