---
name: seguridad
description: Use when auditing a codebase or diff for security vulnerabilities, before writing code that touches auth/tenancy/PII/payments/secrets, when reviewing data-protection or compliance posture (LFPDPPP, GDPR, PCI-DSS), when designing multi-tenant isolation or RLS, when writing security tests, or when the user mentions hacking risks, vulnerabilities, permission segregation, data leaks, or "revisión de seguridad".
---

# Seguridad

Playbook de seguridad de nivel experto. Tú (el modelo que lee esto) NO improvisas criterio de seguridad: **ejecutas este playbook**. Donde el playbook diga "regla dura", no hay excepciones ni matices.

## Paso 0 — Detectar el modo (obligatorio, antes de cualquier otra acción)

| El usuario pide... | Modo | Carga |
|---|---|---|
| "audita", "revisa la seguridad", "busca vulnerabilidades", "análisis de riesgos", "¿es seguro?", revisar un diff/PR/proyecto existente | **AUDIT** | `references/audit-playbook.md` |
| Escribir/modificar código nuevo (feature, endpoint, tabla, job, integración) | **GUARD** | `references/guard-rules.md` |
| Ambiguo o ambos | Ambos | ambos archivos |

En modo GUARD el trabajo principal es la feature; las reglas de seguridad se aplican mientras escribes. En modo AUDIT el trabajo ES el reporte de seguridad.

## Paso 1 — Detectar el stack y cargar referencias condicionales

Ejecuta esta detección con Glob/Grep ANTES de auditar o escribir código. Cada señal activa una referencia adicional:

| Señal en el repo | Cómo detectarla | Carga adicional |
|---|---|---|
| SaaS multi-tenant (varias cuentas/negocios/orgs en una DB) | Grep: `tenant_id\|business_id\|organization_id\|account_id\|workspace_id` en schema/modelos | `references/saas-multitenant.md` |
| Postgres/Prisma/ORM con DB relacional | `schema.prisma`, `*.sql`, `knexfile`, `typeorm`, migraciones | `references/saas-multitenant.md` (sección RLS) |
| Maneja datos personales (usuarios finales, clientes, pacientes, teléfonos, emails) | Grep: `email\|phone\|address\|birth\|curp\|rfc` en schema/modelos | `references/data-protection.md` |
| Pagos o datos de tarjeta | Grep: `stripe\|card\|payment\|pan\|cvv\|checkout` | `references/data-protection.md` (sección PCI) |
| Bot, LLM, agente, tool-calling, chat | Grep: `openai\|anthropic\|llm\|prompt\|langchain\|tool_call\|whatsapp.*bot` | `references/attack-surface.md` (sección LLM) |
| API pública / webhooks / frontend web | controllers, rutas, `webhook`, `cors` | `references/attack-surface.md` |
| Se van a escribir o revisar tests de seguridad | siempre que exista suite de tests | `references/adversarial-testing.md` |

Si no puedes determinar una señal, asume que SÍ aplica (default-closed: sobre-cargar una referencia es barato; omitir un vector es un incidente).

## Paso 2 — Ejecutar el modo

- **AUDIT** → sigue las fases de `audit-playbook.md` en orden. El entregable es el reporte con la plantilla exacta de ese archivo. Regla dura: ningún hallazgo Critical/High sin escenario de explotación concreto y verificado en el código.
- **GUARD** → aplica las reglas de `guard-rules.md` a cada línea que escribas. Antes de declarar terminada la feature, ejecuta el "Checklist de cierre" de ese archivo y los tests de `adversarial-testing.md` que apliquen.

## Reglas duras transversales (aplican en ambos modos, siempre)

1. **La identidad y el tenant JAMÁS vienen del cliente.** Ni en body, ni query, ni params, ni headers custom. Solo de token verificado o mapeo server-side.
2. **La UI nunca es enforcement.** Todo control de acceso/validación se valida en el servidor; el frontend solo oculta.
3. **Default-closed.** Auth global por defecto, excepciones explícitas y anotadas. Ante duda entre permitir y denegar: denegar.
4. **Secretos cifrados en reposo, nunca en logs, nunca en el repo.** Tokens de terceros, API keys y credenciales: cifrado AES-256-GCM o vault; en logs, ni PII ni secretos.
5. **No declarar nada "seguro" ni "terminado" sin el test adversarial correspondiente ejecutado** (ver `adversarial-testing.md`).

## Errores comunes del operador (tú)

| Tentación | Corrección |
|---|---|
| "El diff es chico, no hace falta el playbook" | Los incidentes viven en diffs chicos. Ejecuta el playbook proporcional al diff, pero ejecútalo. |
| "Reporto todo lo que encontré" (20 hallazgos Low de lint) | El valor está en priorizar: 3 Critical accionables > 30 triviales. Usa el rubric de severidad. |
| "Este framework ya lo protege por defecto" | Verifica la config real (versión, flags, middleware registrado), no la reputación del framework. |
| "El test pasa, está protegido" | Aplica la prueba de fuego: si borras el enforcement, ¿el test sigue verde? Si sí, no hay test. |
| "No encontré nada, el código está bien" | Un audit sin hallazgos requiere evidencia de qué buscaste y dónde. Lista las verificaciones ejecutadas. |
