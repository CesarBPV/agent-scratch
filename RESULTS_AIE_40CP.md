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
 