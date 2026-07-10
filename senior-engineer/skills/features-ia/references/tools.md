# Tools — contratos estrictos para tool-calling

**Modelo mental obligatorio: el LLM que llama tus tools es un usuario malicioso torpe.** Va a mandar argumentos faltantes, tipos equivocados, IDs inventados, y va a llamar el mismo tool dos veces. El tool no "confía" en el LLM jamás: valida, acota y reporta.

## Contrato de cada tool (checklist — PUERTA)

- [ ] **Schema estricto:** todos los campos tipados, `required` explícito, enums donde el dominio es cerrado, sin `additionalProperties` abiertas. El schema es la primera línea de defensa, no documentación.
- [ ] **Descripción para el LLM, no para humanos:** qué hace, cuándo usarlo, cuándo NO usarlo, qué devuelve, y ejemplos de argumentos válidos si el formato no es obvio. Una descripción ambigua = llamadas basura.
- [ ] **Validación interna completa** aunque el schema ya valide: existencia de IDs contra la DB, rangos, permisos del usuario final (el tool corre con los permisos del USUARIO de la conversación, no con permisos de sistema).
- [ ] **Errores devueltos AL LLM como resultado**, no como excepción que tumba el loop: `{ error: "order_not_found", hint: "verifica el número con el cliente" }`. Un buen mensaje de error permite al LLM corregirse o escalar; un stack trace filtra internals al prompt.
- [ ] **Idempotencia en tools de escritura:** el LLM PUEDE llamar dos veces (retry de red, loop confundido). Usa claves de idempotencia (`conversationId + intent`), o verifica-antes-de-crear, o diseña la operación para que repetirla sea inocua. "Crear pedido" sin idempotencia = pedidos duplicados en producción, garantizado.
- [ ] **El resultado del tool es contenido no confiable** cuando contiene datos externos (correos, descripciones de productos, mensajes previos): ver `guardrails.md`, sección injection.

## Lectura vs escritura

| Tipo | Ejemplos | Regla |
|---|---|---|
| Lectura | consultar pedido, buscar producto, ver horarios | El LLM puede llamarlos libremente (con scoping por usuario). |
| Escritura reversible | agendar nota interna, marcar seguimiento | Permitido con idempotencia + log auditable. |
| Escritura con efecto externo | crear pedido, cobrar, cancelar, enviar correo a terceros | **Confirmación explícita del usuario final antes de ejecutar** ("Confirmo: ¿cancelo el pedido #123?" → sí) o cola de aprobación humana. El bot propone, el humano dispone. |

Si un tool de escritura con efecto externo no tiene paso de confirmación, es un incidente esperando fecha.

## El loop agéntico se acota, siempre

```python
MAX_ITERATIONS = 8          # por turno de conversación
MAX_COST_USD = 0.50         # presupuesto por turno

for i in range(MAX_ITERATIONS):
    resp = call_llm(...)
    if resp.stop_reason != "tool_use":
        return resp                      # respuesta final
    if spent() > MAX_COST_USD:
        return escalate("presupuesto excedido")   # a humano, ver guardrails.md
    results = [run_tool(tc) for tc in resp.tool_calls]
    # cada run_tool atrapa SUS excepciones y devuelve {error: ...} como resultado
return escalate("loop sin converger")    # NUNCA un while True
```

Reglas del loop:

- **Límite de iteraciones y de costo** por turno. Al excederlos: escalar a humano con contexto, no cortar en silencio ni seguir quemando dinero.
- **Detección de bucle tonto:** mismo tool + mismos argumentos 2 veces seguidas → no lo ejecutes de nuevo; devuelve al LLM "ya llamaste esto con estos argumentos, el resultado fue X" o escala.
- **Paralelismo solo en lecturas.** Escrituras: secuenciales y en el orden pedido.
- **Log por iteración:** tool, argumentos, resultado (truncado), duración. Cuando un bot hace algo raro en producción, este log es la única forma de saber qué pasó.

## Errores comunes que vas a evitar

| Error | Consecuencia | Fix |
|---|---|---|
| Tool que lanza excepción no atrapada | Se cae el turno completo del bot | try/catch dentro del tool → `{error}` al LLM |
| Descripción "gets order info" | El LLM lo llama para todo o para nada | Cuándo sí / cuándo no / qué devuelve |
| `orderId: string` sin validar formato | El LLM inventa IDs y el tool consulta basura | Validar formato + existencia; error con hint |
| Sin scoping por usuario | El bot de un cliente lee pedidos de otro | Toda query filtrada por la identidad de la conversación |
| Resultado del tool gigante (JSON de 50KB) | Contexto inflado, costo x10, respuestas peores | El tool resume/selecciona campos; el LLM no necesita el row completo |
