# Arquetipos por condición dominante

Cuatro perfiles. Elige UNO según la fuerza dominante del intake; los demás sirven como ruta de evolución. Cada perfil da el stack de referencia por capa con criterios de elección (no marcas cerradas: el proveedor concreto se elige con los criterios del final). Costos en USD/mes, rangos de 2025-2026.

---

## A. Mínimo gasto / validación (~$0–25/mes)

**Cuándo aplica:** idea sin usuarios pagando, presupuesto ~$0, un solo desarrollador, lo dominante es descubrir si alguien quiere el producto.
**Cuándo NO:** ya hay clientes que pagan o datos sensibles reales de terceros — súbete al perfil B; el free tier no da backups serios ni SLA.

| Capa | Referencia | Criterio |
|---|---|---|
| Hosting | 1 servicio en PaaS con free tier/hobby (web+API juntos) | Deploy por git push, HTTPS y dominio incluidos, cero config de servidores |
| DB | Postgres gestionado chico (free tier o ~$5) | Backup automático aunque sea diario; NUNCA SQLite en disco efímero de PaaS |
| Backend | Monolito en el lenguaje que el equipo domina | Velocidad de iteración; nada de servicios separados |
| Front | Server-rendered o SPA servida por el mismo servicio | Un solo deploy |
| Jobs | Cron del PaaS o tabla de tareas + tick cada minuto | Sin cola dedicada; los seams (lógica fuera del controller) preparan la cola futura |
| Storage | Bucket S3-compatible free tier | Nunca en el disco del PaaS (efímero) |
| Observabilidad | Logs del PaaS + uptime monitor gratuito | Suficiente para validación |

**Riesgo aceptado (decláralo):** caídas de minutos sin aviso, cold starts, sin soporte.
**Migración a B:** el código no cambia (mismo monolito); cambia el plan del PaaS, la DB a plan con point-in-time recovery, y se separa el worker. Se reaprovecha ~100%.

---

## B. SaaS pequeño en producción (~$25–150/mes)

**Cuándo aplica:** clientes reales pagando, datos de terceros (PII), 1-3 desarrolladores, lo dominante es entregar features sin incendios. **El perfil default cuando hay duda.**
**Cuándo NO:** si nadie paga aún (usa A); si una hora de caída pierde transacciones críticas (usa C).

| Capa | Referencia | Criterio |
|---|---|---|
| Hosting | PaaS tipo Railway/Render/Fly: servicio web + worker separado | Procesos separados API/worker; restart automático; deploy por push con rollback |
| DB | Postgres gestionado ~$10-30 con point-in-time recovery | PITR es la línea que separa "hobby" de "producción"; verifica retención ≥7 días |
| Backend | Monolito modular (capas: controller→service→repo) | Un deployable; los límites de módulo son de código, no de red |
| Front | Mismo repo (monorepo), deploy propio o en el PaaS | Compartir tipos/contratos entre front y API |
| Jobs | Cola respaldada por Redis gestionado (~$5-10) o por la propia DB | Obligatoria si hay webhooks/reintentos/envíos; jobs idempotentes desde el día 1 |
| Storage | Bucket S3-compatible con versionado | Datos de clientes: versionado + política de retención |
| Observabilidad | Logs estructurados (JSON + trace_id) + error tracking (free tier) + uptime + alerta de backup fallido | La alerta que más importa: backup fallido y errores 5xx |

**Riesgo aceptado:** single-instance por servicio → deploys con segundos de blip y caída de región = caída tuya (raro; decláralo).
**Migración a C:** duplicar instancias tras el load balancer del PaaS, réplica de DB. Se reaprovecha todo si los seams existen (sesiones fuera de memoria, jobs idempotentes, sin estado local en disco).

---

## C. Alta disponibilidad requerida (~$150–800/mes)

**Cuándo aplica:** una hora de caída pierde dinero o clientes de forma medible (intake #4 con consecuencia concreta), o hay SLA contractual.
**Cuándo NO (paranoia común):** "se vería mal si se cae" no es HA; el perfil B con buen restart da ~99.5% que cubre a la mayoría. Exige la consecuencia medible antes de aceptar este perfil.

| Capa | Referencia | Criterio |
|---|---|---|
| Hosting | ≥2 instancias por servicio tras load balancer, health checks reales (que toquen DB, no solo `200 OK`) | Deploy rolling sin downtime; una instancia muere y nadie lo nota |
| DB | Postgres gestionado con réplica y failover automático | Prueba el failover ANTES de necesitarlo (game day) |
| Backend/Front | Igual que B — HA no cambia el código, cambia la topología | Stateless obligatorio: sesiones en DB/Redis, cero estado en disco local |
| Jobs | Cola con Redis gestionado con réplica; workers ≥2 | Idempotencia deja de ser buena práctica y pasa a ser correctness |
| Storage | Bucket con replicación del proveedor | — |
| Observabilidad | Métricas con alertas (p95, error rate, queue depth), on-call definido, status page | HA sin alertas es HA de adorno: nadie se entera de que el failover ocurrió |

**Multi-región: casi siempre paranoia.** Solo se justifica con usuarios en continentes distintos sufriendo latencia medida, o requisito contractual/regulatorio de residencia. La caída de una región completa de un proveedor grande es tan rara que un plan de restauración documentado (DR pasivo: backups en otra región + runbook) cubre el riesgo a 1/10 del costo del multi-región activo.
**Migración a D:** ya casi está — D agrega elasticidad, no redundancia.

---

## D. Picos de tráfico (~variable, $100–1000+/mes)

**Cuándo aplica:** tráfico con picos de ≥10× lo normal y predecibles o súbitos (campañas, temporadas, eventos, viralidad). La fuerza dominante es absorber el pico sin pagar el pico las 24h.
**Cuándo NO:** crecimiento sostenido y gradual no es "picos" — es el perfil B/C escalando el plan.

| Capa | Referencia | Criterio |
|---|---|---|
| Hosting | Autoscaling horizontal (PaaS con autoscale o contenedores gestionados) | Escalar por métrica (CPU/RPS/queue depth), con mínimo bajo y máximo acotado (protege la cartera) |
| DB | La pieza que NO autoescala: connection pooler (pgbouncer o el del proveedor) obligatorio + plan dimensionado al pico de escrituras | Las conexiones matan Postgres antes que el CPU; el pooler es lo primero |
| Backend | Igual, pero TODO lo diferible se desacopla a la cola | El request síncrono hace lo mínimo; la cola absorbe el pico y los workers drenan a su ritmo |
| Front | Assets tras CDN, páginas públicas cacheadas/estáticas | El pico de curiosos no debe tocar tu API |
| Jobs | La cola ES el amortiguador: workers autoescalan por queue depth | Backpressure explícito: qué se degrada primero (ej. recordatorios se retrasan, bookings no) |
| Observabilidad | Igual que C + dashboards de saturación en vivo + límites de gasto/alertas de billing | El autoscaling sin tope convierte un ataque/bug en una factura |

**Decisión clave — qué desacoplar:** clasifica cada operación en (1) síncrona intocable (login, booking, pago), (2) diferible segundos (notificaciones, webhooks salientes), (3) diferible minutos (reportes, emails). Solo (1) dimensiona el API; (2) y (3) viven en la cola.

---

## Criterios para elegir proveedor concreto (cualquier perfil)

1. **Precio predecible** — factura mensual estimable; cuidado con serverless por-request para cargas constantes (barato de entrada, caro sostenido).
2. **Salida barata** — Postgres estándar + contenedores + S3-compatible = te mueves en un fin de semana. Servicios propietarios (DB exclusiva, funciones con SDK propio) = lock-in que se paga con condición que lo justifique.
3. **PITR y backups verificables** en DB gestionada — restaura uno de prueba antes de confiar.
4. **Región cercana a tus usuarios** (y compatible con residencia de datos si el intake #5 lo impone).
5. **Track record** — status page pública con historial; sin historial, no hospedes producción ahí.
6. **El que el equipo ya conoce gana empates.**
