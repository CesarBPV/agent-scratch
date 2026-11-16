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

## Sesión 6 — Tools del MCP en el chat

Dominio: `<dominio>`. MCP: el mock (`http://127.0.0.1:6274/?MCP_INSPECTOR_API_TOKEN=ee7f99b96a088fe4456b2bcf889411aec5189602b51196220cf7676f7eec3a3e`) y, si ya tienes `MCP_URL`, el compartido (`bank-mcp`).

### Tools de tu dominio

| MCP | Tools que lista `dev/mcp/mcp_tools.py` |
|---|---|
| Mock | _(anotar)_ |
| Compartido | _(anotar)_ |

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

### ¿El modelo inventó algún dato cuando ninguna tool lo resolvía?

no

### Conclusión
 