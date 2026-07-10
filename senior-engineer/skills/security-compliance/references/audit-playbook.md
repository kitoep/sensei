# AUDIT — Playbook de auditoría de seguridad

Entregable: un reporte de hallazgos priorizados con la plantilla exacta de la sección "Plantilla de reporte". No es una charla; es un documento accionable.

## Fase 1 — Reconocimiento del stack (10-15 min de exploración)

Antes de buscar vulnerabilidades, mapea el terreno. Ejecuta y anota:

1. **Lenguajes y frameworks:** lee `package.json`/`requirements.txt`/`go.mod`/`pom.xml`. Anota versiones (una versión vieja ES un hallazgo potencial).
2. **Topología:** ¿monolito, monorepo, microservicios? ¿qué apps existen (api, web, worker, jobs)? Los fixes deben ser simétricos entre apps.
3. **Superficie de entrada:** rutas HTTP/controllers, webhooks, colas/jobs, uploads, WebSockets, CLI. Cada punto donde entra dato externo es un vector.
4. **Datos:** ¿qué DB? ¿hay multi-tenancy? ¿qué PII se almacena? ¿pagos? ¿secretos de terceros?
5. **Auth:** ¿cómo se autentica (JWT/sesión/OAuth)? ¿guard global o por-ruta? ¿RBAC? ¿2FA?
6. **Config de seguridad existente:** busca `helmet`, `cors`, `csrf`, `throttler`/rate-limit, RLS, cifrado. Anota qué existe y qué falta.

Registra el mapa. Si el usuario acotó el alcance (solo el diff, solo un módulo), respétalo pero nota lo que quedó fuera.

## Fase 2 — Threat model rápido

Contesta en 3 líneas cada uno:

- **Activos:** ¿qué es lo más valioso que se puede robar/corromper? (PII de clientes, credenciales, dinero, aislamiento entre tenants).
- **Actores:** ¿quién ataca? Usuario autenticado malicioso (el más importante en SaaS), tenant vecino, atacante anónimo, insider, terceros comprometidos (supply chain).
- **Superficies:** el listado de la Fase 1, punto 3.

Esto enfoca el barrido: prioriza los vectores que tocan los activos de mayor valor.

## Fase 3 — Barrido por dimensiones

Recorre cada dimensión aplicable (según el stack detectado). Para cada una, usa los patrones grep y checks del archivo indicado:

| Dimensión | Referencia | Pregunta rectora |
|---|---|---|
| Aislamiento multi-tenant / RLS | `saas-multitenant.md` | ¿Puede el tenant A ver/tocar datos del tenant B? |
| Control de acceso / RBAC / IDOR | `attack-surface.md` (Broken Access Control) | ¿Puede un usuario acceder a lo que no le toca cambiando un ID? |
| Inyección (SQL/NoSQL/command) | `attack-surface.md` | ¿Hay input que llega a una query/shell sin parametrizar? |
| Auth y sesiones | `guard-rules.md` (auth) + `attack-surface.md` | ¿Default-closed? ¿tokens/sesiones bien manejados? |
| Secretos y cifrado | `guard-rules.md` (secretos) | ¿Hay secretos en claro, en logs, en el repo? |
| Protección de datos / compliance | `data-protection.md` | ¿PII en logs? ¿retención? ¿derechos del titular? |
| Superficie web (XSS/CSRF/CORS/headers) | `attack-surface.md` | ¿Config de headers, CORS y CSRF correcta? |
| Supply chain | `attack-surface.md` (supply chain) | ¿Dependencias vulnerables, lockfile, typosquatting? |
| LLM/bot (si aplica) | `attack-surface.md` (LLM) | ¿Prompt injection, exfiltración vía tools, el bot inventa? |
| Calidad de los tests de seguridad | `adversarial-testing.md` | ¿Los tests prueban lo denegado o solo confirman lo feliz? |

## Fase 4 — Verificación adversarial de hallazgos (regla dura)

Antes de escribir el reporte, cada hallazgo candidato a **Critical o High** debe pasar verificación:

1. **Escribe el escenario de explotación concreto:** request/input exacto, precondición (rol, estado), y el efecto dañino observable. Si no puedes escribirlo, no es Critical/High — bájalo o descártalo.
2. **Traza el código:** confirma con `archivo:línea` que NO hay un control que lo mitigue aguas arriba o abajo (un guard, un constraint de DB, un middleware). Los falsos positivos vienen de no ver la defensa que sí existe.
3. **Si el proyecto es grande o los hallazgos son muchos, despliega subagentes de verificación:** para cada hallazgo High/Critical, lanza un subagente con el encargo *"Intenta REFUTAR este hallazgo: <hallazgo + archivo:línea>. Busca cualquier control que lo mitigue. Devuelve CONFIRMADO o REFUTADO con evidencia."* Un hallazgo sobrevive solo si el subagente no logra refutarlo. Lanza los subagentes en paralelo (un mensaje con varias llamadas).

Esto evita el modo de falla más común de un audit: reportar plausibles-pero-falsos que queman la credibilidad del reporte entero.

## Rubric de severidad (definiciones operativas)

Asigna la severidad más alta que aplique:

- **Critical:** explotable por un atacante remoto con privilegios bajos o nulos, y el impacto es cruce de datos entre tenants, RCE, robo masivo de credenciales/PII, o pérdida de dinero. Escenario de explotación verificado obligatorio.
- **High:** explotable por un usuario autenticado contra datos/acciones que no le corresponden (IDOR, escalada de privilegios, bypass de auth en una ruta), o secreto expuesto reutilizable. Escenario verificado obligatorio.
- **Medium:** requiere condiciones poco comunes, o el impacto es acotado (fuga de metadatos, falta de rate-limit en endpoint sensible, CSRF en acción no crítica, dependencia vulnerable sin ruta de explotación confirmada).
- **Low:** hardening/defensa en profundidad sin ruta de explotación (header faltante, verbosidad de errores, práctica subóptima). Agrúpalos; no los enumeres uno por uno si son triviales.

Regla dura: **sin escenario de explotación concreto y verificado, un hallazgo no puede ser Critical ni High.** Máximo Medium.

## Plantilla de reporte (formato exacto de salida)

```
# Reporte de auditoría de seguridad — <proyecto/diff> — <fecha>

## Resumen ejecutivo
- Alcance auditado: <qué se revisó / qué quedó fuera>
- Stack: <lenguajes, frameworks, DB, versiones relevantes>
- Hallazgos: <N> Critical, <N> High, <N> Medium, <N> Low
- Riesgo principal en 1 frase: <...>

## Hallazgos (ordenados por severidad, Critical primero)

### [CRITICAL-01] <título corto y específico>
- **Severidad:** Critical
- **Dimensión:** <ej. Aislamiento multi-tenant>
- **Evidencia:** `apps/api/src/x/y.ts:142` (y todas las líneas relevantes)
- **Escenario de explotación:** <request/input exacto → precondición → efecto dañino observable, paso a paso>
- **Verificación:** CONFIRMADO — <cómo se confirmó / resultado del subagente refutador>
- **Fix propuesto:** <cambio concreto; si aplica, snippet>
- **Fixes simétricos:** <otras ubicaciones/apps con el mismo patrón: archivo:línea, o "ninguna">
- **Esfuerzo:** <S/M/L>

[repetir por hallazgo]

## Hardening (Low agrupados)
- <lista compacta de mejoras de defensa en profundidad>

## Gap-analysis de compliance (si aplica)
- LFPDPPP/GDPR/PCI: <controles ausentes, ver data-protection.md>

## Verificaciones ejecutadas (para un audit sin hallazgos, esto es el entregable)
- <lista de qué se buscó, con qué patrón, en qué archivos, y por qué salió limpio>
```

## Regla final del reporte

Si terminas sin hallazgos Critical/High, NO escribas "el código es seguro". Escribe "no encontré vulnerabilidades Critical/High en el alcance auditado" + la sección "Verificaciones ejecutadas". La ausencia de evidencia no es evidencia de ausencia; documenta el esfuerzo para que sea auditable.
