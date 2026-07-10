# Descripción del PR — auditable, no narrativa del diff

## Principio

El revisor YA VE el diff. La descripción existe para darle lo que el diff no contiene: el contexto del problema, las decisiones tomadas, la evidencia de que funciona y los riesgos. **Re-narrar el diff ("se modificó el archivo X para agregar la función Y") es el anti-patrón #1: ocupa espacio, no informa nada y entierra lo importante.**

Test para cada oración de tu descripción: ¿el revisor puede deducirla leyendo el diff? Si sí, bórrala.

## Plantilla obligatoria (las 5 secciones, en este orden)

```markdown
## Contexto / Problema

<Qué estaba roto o qué falta y a quién afecta. El estado del mundo ANTES
del PR. Referencia al ticket/issue/reporte. Si es un bug: el síntoma
observable y cómo reproducirlo. 2-5 líneas.>

## Qué cambia y por qué

<La decisión, no el diff. Qué enfoque tomaste y POR QUÉ ese y no otro.
Alternativas descartadas y su razón, si las evaluaste. Si hay un cambio
en el diff que sorprendería al revisor ("¿y esto qué hace aquí?"),
explícalo aquí antes de que pregunte. 3-8 líneas.>

## Cómo se verificó

<EVIDENCIA EJECUTADA, no intenciones. Por cada afirmación, la salida real:
- Suite: comando + resumen pegado (`142 passed, 0 failed`).
- Test nuevo: qué captura, y que lo viste FALLAR sin el cambio.
- Verificación manual: pasos exactos + resultado observado (no "probé y
  funciona": qué request/acción, qué respuesta/estado quedó).
- Para UI: screenshot del antes/después.
Lo NO verificado se declara: "no probado en <entorno/caso> porque X".>

## Riesgos y qué NO cubre

<Qué puede romperse y dónde mirarían primero si algo falla. Deuda que
este PR deja a propósito. Casos fuera de alcance. Migraciones o pasos
de deploy requeridos. "Riesgos: ninguno" está prohibido — un cambio sin
ningún riesgo identificable no ameritaba PR; identifica al menos el área
de mayor exposición.>

## Notas para el revisor

<Por dónde empezar a leer el diff. Qué archivo es el corazón del cambio
y cuáles son mecánicos/generados. Dónde quieres segunda opinión.
Si es apilado: sobre qué PR y qué rango revisar.>
```

## Reglas duras

- **Las 5 secciones son obligatorias.** Sin evidencia en "Cómo se verificó" el PR no se abre: se termina la verificación primero. "Lo verifica el CI" no cuenta — el CI verifica que la suite pasa, no que el cambio cumple lo que el PR promete.
- "Cómo se verificó" debe contener salidas REALES pegadas, de comandos corridos en esta sesión después del último commit. Prohibido pegar salidas de memoria o de una corrida anterior al último cambio.
- La descripción se escribe leyendo `git diff main...HEAD` (el rango completo del PR), no el recuerdo de lo que hiciste.
- Longitud objetivo: que el revisor la lea en ≤2 minutos. Si excede ~40 líneas, probablemente el PR excede su alcance (ver `branches-prs.md`), no la descripción su brevedad.

## Ejemplo calibrado

❌ **Re-narración del diff (prohibido):**

> Este PR modifica `discount.py` para cambiar la función `calculate_discount` y que use `price_with_tax` en lugar de `base_price`. También se actualiza `test_discount.py` agregando el test `test_discount_with_tax` y se modifica `models.py` para exponer la propiedad.

Todo eso está en el diff. Cero información nueva.

✅ **Auditable:**

> **Contexto:** Órdenes con cupón cobraban de más: el descuento se aplicaba sobre el precio sin IVA (reporte #482, ~3% de las órdenes de junio).
>
> **Qué cambia y por qué:** El descuento ahora se calcula sobre el precio final con IVA, que es lo que el cliente ve y lo que Legal confirmó como correcto. Descarté recalcular IVA post-descuento: cambia los totales históricos y requiere migración; este enfoque solo afecta órdenes nuevas.
>
> **Cómo se verificó:** `pytest tests/orders/ -q` → `87 passed, 0 failed`. El test nuevo `test_discount_with_tax` falla sin el fix (`AssertionError: 116.0 != 100.0` — salida pegada abajo). Reproduje la orden del reporte #482 en staging: total pasa de $116.00 a $100.00.
>
> **Riesgos:** cualquier reporte financiero que asuma el cálculo viejo mostrará una discontinuidad desde el deploy. No cubre órdenes históricas (a propósito, ver arriba).
>
> **Notas para el revisor:** el corazón es `discount.py:42-58`; `models.py` solo expone una propiedad ya calculada.
