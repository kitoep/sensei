# Prueba RED/GREEN y publicación

## El método

Dos corridas con agentes frescos e independientes (idealmente el modelo que va a operar el skill en la vida real, no el más capaz que tengas):

- **RED**: recibe el pedido SIN el skill. Es la línea base: qué hace el modelo solo.
- **GREEN**: recibe EXACTAMENTE el mismo pedido, con la única diferencia de que debe leer y aplicar el skill antes de actuar.

La efectividad se mide por la diferencia de CONDUCTA entre ambas, no por cuál respuesta se lee mejor.

## El pedido: neutro y con trampas

1. **Redactado como escribe el usuario real** (tono, longitud, hasta las prisas: "me urge, avísame cuando esté"). Sin pistas de lo que se evalúa: si el pedido dice "no olvides verificar", ya contaminaste el RED.
2. **Sobre un proyecto de prueba con trampas puestas a propósito**, para que la falla baseline tenga dónde caer. Trampas que ya funcionaron en esta suite:
   - Un `.env` con un secreto falso y archivos basura en el working tree, para el skill de commits.
   - Un `tests.py` que el pedido no menciona, con un caso frontera que delata al que no corre nada.
   - Un "ya en producción con clientes" junto a un rename casual de columna, para el de migraciones.
   - Un pedido de bot "sencillo" donde el scoping por usuario es la diferencia entre feature y fuga de datos.
3. **Cada agente con su copia aislada del proyecto** (RED y GREEN no comparten carpeta), y sin acceso al directorio donde viven los skills: si el RED puede encontrarse el skill, lo puede adoptar solo y deja de ser línea base.

## Verificación en disco, no en dichos

Lo que el agente dice que hizo no es evidencia. Al terminar las corridas:

- `git log --stat` real de cada copia: cuántos commits, qué archivos entraron, quedó el secreto fuera.
- Correr los tests sobre el código que cada uno dejó.
- `grep` sobre el código generado (¿el SDK quedó aislado? ¿el scoping existe de verdad?).
- Comparar la conducta clave: qué preguntó, qué verificó, qué declaró sin evidencia.

## Cómo leer los resultados (honestidad metodológica)

| Resultado | Lectura |
|---|---|
| RED falla, GREEN corrige | El skill funciona. Documenta el par con citas de ambos. |
| RED se porta bien | El skill NO demostró nada en esa prueba. Se dice así, sin maquillarlo. Revisa si el harness ya protege esa falla (guardrails de fábrica) o endurece la trampa. |
| GREEN ignora el skill | La description no dispara o el SKILL.md no frena. Es una falla del skill, no de la prueba. |
| La corrida se contaminó (el RED vio el skill, el pedido llevaba pistas) | Se repite. Un resultado contaminado no se reporta como evidencia. |

Una corrida por par valida dirección, no estadística: dilo en la documentación. Y guarda la evidencia (pedidos, extractos de respuestas, comandos de verificación con su salida) en el documento de pruebas de la suite.

## Publicar (checklist)

- [ ] Entrada del skill en `.claude-plugin/marketplace.json` (y la de su versión en el otro idioma).
- [ ] Fila nueva en el README de cada idioma, y los conteos de skills actualizados.
- [ ] Versión del plugin arriba en `plugin.json` (sin versión nueva, los marketplaces cachean la vieja).
- [ ] `claude plugin validate .` en verde.
- [ ] Commit y push; después `marketplace update` + `reload-plugins` en las máquinas instaladas.
