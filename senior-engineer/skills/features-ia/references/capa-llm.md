# Capa LLM — una sola puerta hacia el proveedor

**Regla dura: ningún archivo del proyecto importa el SDK del proveedor excepto la capa LLM.** Si encuentras `import anthropic` / `import openai` fuera de `llm/` (o el módulo equivalente del proyecto), eso es un bug de arquitectura: muévelo antes de seguir.

## Por qué existe la capa

No es (principalmente) para "cambiar de proveedor algún día". Es porque estas 6 cosas deben pasar en **cada** llamada, y solo se garantizan si hay un único punto de paso:

1. Retries con backoff ante 429/5xx/timeouts.
2. Timeout explícito (el default del SDK puede ser minutos).
3. Tracking de tokens, costo y latencia por request.
4. Logging del prompt y la respuesta (con redacción de PII, ver guardrails.md).
5. Selección de modelo por configuración, no hardcodeada en el call-site.
6. Manejo uniforme de errores (rate limit ≠ contenido rechazado ≠ red caída).

## Prohibido inventar — protocolo de verificación

Antes de escribir un nombre de modelo, parámetro o método de SDK:

| Qué vas a escribir | Cómo lo verificas ANTES |
|---|---|
| Nombre de modelo (`claude-...`, `gpt-...`) | Docs actuales del proveedor, o lista de modelos vía API, o el modelo ya usado en el repo (grep). NUNCA de memoria. |
| Parámetro de request (`temperature`, `max_tokens`, `tool_choice`...) | Referencia de API actual o types del SDK instalado (`node_modules/.../types`, `pip show` + código fuente). |
| Método/firma del SDK | El SDK instalado en ESTE repo (lee sus types/código), no la versión que recuerdas. |

Si no puedes verificar (sin red, sin SDK instalado): escribe el código con el valor en configuración + comentario `VERIFICAR:` y decláralo explícitamente en tu reporte: "el nombre del modelo está sin verificar contra docs actuales". Un valor declarado-dudoso es honesto; un valor inventado-seguro es una bomba.

**Default correcto:** si el repo ya llama a un LLM, usa el mismo modelo/versión que ya está en producción salvo instrucción contraria.

## Interfaz mínima de la capa

```typescript
// llm/client.ts — ÚNICO módulo que importa el SDK del proveedor
export interface LlmRequest {
  purpose: string;            // "bot-respuesta", "clasificar-ticket" — para métricas y logs
  system: string;
  messages: Msg[];
  tools?: ToolDef[];
  maxTokens: number;          // SIEMPRE explícito
  temperature?: number;
  model?: ModelAlias;         // alias propio ("fast" | "smart"), mapeado a IDs reales en config
}

export interface LlmResponse {
  text: string | null;
  toolCalls: ToolCall[];
  stopReason: "end" | "tool_use" | "max_tokens" | "refusal";
  usage: { inputTokens: number; outputTokens: number; costUsd: number; latencyMs: number };
}

export async function callLlm(req: LlmRequest): Promise<LlmResponse>
```

Puntos no negociables de la implementación:

- **Alias de modelo en config**, no IDs regados: `{ fast: "<id-verificado>", smart: "<id-verificado>" }`. Cambiar de modelo = 1 línea de config.
- **Retries:** solo para errores transitorios (429, 5xx, timeout de red), backoff exponencial con jitter, máximo 3 intentos. NUNCA reintentar un rechazo de contenido o un 400: el resultado será el mismo y pagas doble.
- **Timeout por request** acorde al caso de uso (un bot de chat no puede esperar 120s).
- **`stopReason: "max_tokens"` es un error de producto**, no un éxito: la respuesta está truncada. Manéjalo (reintentar con más presupuesto, o fallar limpio), no lo entregues al usuario.
- **Costo calculado con precios en config** (también cambian). Registra `purpose`, tokens, costo y latencia en cada llamada — es lo que permite responder "¿por qué la factura subió 3x?" sin arqueología.

## Checklist — PUERTA para dar por buena la capa (o un feature que la usa)

- [ ] `grep` del nombre del SDK en el repo: 0 hits fuera del módulo de la capa.
- [ ] Todos los IDs de modelo viven en config y fueron verificados (o marcados `VERIFICAR:`).
- [ ] `maxTokens` y timeout explícitos en cada `purpose`.
- [ ] Un request que falla con 429 se reintenta; uno que falla con 400 no.
- [ ] Cada llamada deja una línea de log/métrica con purpose, tokens, costo, latencia.
