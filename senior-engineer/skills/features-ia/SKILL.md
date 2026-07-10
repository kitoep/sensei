---
name: features-ia
description: Use when building any feature powered by an LLM — chatbots, WhatsApp/customer-support bots, tool-calling/function-calling, agents, RAG, classification/extraction with AI, prompt changes, integrating AI provider APIs (Anthropic, OpenAI, etc.), or writing/updating evals; when the user says "agrega un bot", "conecta la IA", "mejora el prompt", or asks to add AI to a product.
---

# Features-IA — construir con LLMs sin inventar

## Principio

Un feature con LLM no es "llamar la API y cruzar los dedos". Es: una capa propia que aísla al proveedor, tools con contratos estrictos, guardrails que asumen input hostil, y evals que deciden si un prompt puede cambiar. El LLM es un componente **no determinístico y no confiable por diseño**: todo lo que lo rodea debe compensar eso.

**Regla cero — prohibido inventar:** ningún nombre de modelo, parámetro de API o método de SDK sale de tu memoria directo al código. O lo verificaste contra la documentación/SDK actual del proveedor en esta sesión, o lo declaras como no verificado. Los modelos y APIs cambian más rápido que tu conocimiento.

**Violar la letra de estas reglas es violar su espíritu.** No hay prototipos "temporales" exentos de la capa, ni cambios de prompt "demasiado chicos" para el eval.

## Protocolo (carga el reference ANTES de trabajar en su área)

| Área de trabajo | Qué cubre | PUERTA para avanzar | Reference |
|---|---|---|---|
| 1. Capa LLM | Interfaz propia agnóstica: retries, timeouts, costo, logging | Ninguna llamada al SDK fuera de la capa; modelos/params verificados | `references/capa-llm.md` |
| 2. Tools | Tool-calling: schemas estrictos, errores, idempotencia, loop acotado | Cada tool pasa el checklist de contrato | `references/tools.md` |
| 3. Guardrails | Prompt injection, alcance, escalamiento, PII, validación de salida | Checklist de guardrails cubierto o brechas declaradas | `references/guardrails.md` |
| 4. Evals | Golden set, regresión como puerta de cambio de prompt, LLM-judge | Ningún prompt cambia en prod sin eval de regresión corrido | `references/evals.md` |

Un feature nuevo toca las 4 áreas. Un cambio de prompt toca mínimo la 4. Un tool nuevo toca la 2 y la 3.

## Red flags — DETENTE si te descubres pensando esto

- "El modelo se llama `gpt-5-turbo` / `claude-4-sonnet`, lo recuerdo" → no lo recuerdas, lo estás generando. Verifica contra docs/SDK actual.
- "Importo el SDK del proveedor aquí mismo y listo" → toda llamada pasa por la capa propia. Sin excepciones "temporales".
- "El LLM va a llamar el tool con los argumentos correctos" → el LLM es un usuario malicioso torpe. El tool valida todo.
- "Es un bot interno, no necesita guardrails" → todo texto que entra al prompt es hostil hasta demostrar lo contrario.
- "El prompt nuevo se ve mucho mejor" → "se ve mejor" no es una métrica. Corre el golden set.
- "Parseo la respuesta con un regex y ya" → la salida del LLM se valida contra schema, con fallback definido para cuando venga basura.
- "Le pongo `JSON.parse` directo a la respuesta" → ¿y cuando devuelva markdown con el JSON adentro, o una disculpa? Fallback obligatorio.

## Racionalizaciones conocidas

| Excusa | Realidad |
|---|---|
| "Verificar el nombre del modelo es paranoia, siempre uso este" | Los proveedores deprecan modelos cada pocos meses. Un modelo inventado/viejo falla en runtime o (peor) responde peor y nadie sabe por qué. |
| "La capa de abstracción es sobre-ingeniería para un solo proveedor" | La capa no es (solo) para cambiar de proveedor: es donde viven retries, timeouts, costo y logging. Sin ella, cada bug de producción es arqueología. |
| "Los evals son para después, primero que funcione" | Sin golden set no sabes si "funciona". El primer prompt define el baseline; sin baseline, cada cambio es una apuesta ciega. |
| "Prompt injection es teórico, mis usuarios no son hackers" | El injection más común no es un hacker: es un cliente pegando un correo/documento que contiene instrucciones. Y basta un caso viral para quemar el producto. |
| "20 casos de eval no prueban nada" | 20 casos reales detectan las regresiones groseras que hoy detecta nadie. Es el mínimo viable, no el estado final. |
| "El loop del agente termina solo cuando resuelve" | Un loop sin límite de iteraciones ni presupuesto es una factura infinita esperando un caso raro. |
