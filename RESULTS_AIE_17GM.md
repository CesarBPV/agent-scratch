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
