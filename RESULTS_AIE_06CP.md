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

Pendiente: correr `uv run eval` completo y copiar aquí el resumen. Antes conviene revisar los archivos `fees.AIE_17GM.jsonl` y `fees.AIE_40CP.jsonl`: repiten casos del golden set base (el de ChristianPM tiene los 30), y la corrida completa los cuenta dos veces.
