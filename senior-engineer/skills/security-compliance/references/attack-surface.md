# Superficie de ataque — OWASP operativo

No es una lista teórica. Para cada vector: **cómo buscarlo** (patrón concreto) y **cómo se ve el fix**. Recorre los que apliquen al stack detectado.

## A01 — Broken Access Control (el #1 en SaaS: IDOR / BOLA)

**Qué es:** un usuario accede a datos/acciones que no le corresponden cambiando un identificador.

**Cómo buscarlo:**
- Grep rutas con IDs: `:id`, `/{id}`, `req.params`, `req.query.*id`.
- Por cada una, verifica: ¿se comprueba que el recurso pertenece al tenant/usuario del token ANTES de operar? Busca el `findFirst({ where: { id, tenantId } })` vs. el peligroso `findUnique({ where: { id } })` sin tenant.
- Grep acciones de escritura (`update`, `delete`, `destroy`) que reciben un ID del cliente.

**Fix:** todo acceso a recurso por ID filtra también por el tenant/dueño derivado del token. Devolver 404 (no 403) si no pertenece. Ver saas-multitenant.md.

## A03 — Inyección (SQL / NoSQL / Command / Path)

**Cómo buscarlo:**
- SQL crudo con interpolación: grep `` `SELECT ${ `` , `"SELECT " +`, `query(` con concatenación, `raw(`, `$queryRawUnsafe`.
- NoSQL: objetos del cliente pasados directo a filtros (`find(req.body)`), operadores `$where`, `$regex` con input crudo.
- Command: `exec(`, `execSync(`, `spawn(` con strings construidos de input.
- Path traversal: `readFile`/`join` con nombre de archivo del cliente sin sanitizar (`../`).

**Fix:** parámetros/bindings (`$1`, `?`), nunca interpolación. Para NoSQL, whitelistear campos y castear tipos. Para comandos, arrays de args y evitar shell. Para paths, resolver y verificar que quede dentro del directorio permitido.

## A02/A04 — Cripto y secretos

**Cómo buscarlo:**
- Secretos hardcodeados: grep `api[_-]?key`, `secret`, `password`, `token`, `private[_-]?key`, cadenas tipo `sk_live`, `AKIA`, JWT literales. Revisa también el historial de git (`git log -p | grep`).
- Algoritmos débiles: `md5`, `sha1` para contraseñas, `Math.random()` para tokens/IDs de seguridad, `createCipher` (deprecado) en vez de `createCipheriv`.
- Tokens de terceros guardados en claro en DB.

**Fix:** secretos desde env/vault; rotar cualquiera expuesto. Contraseñas con bcrypt/argon2. Tokens/nonces con CSPRNG (`crypto.randomBytes`). Cifrado de campo con AES-256-GCM (`createCipheriv`). Ver guard-rules.md §5.

## A05 — Misconfiguración de seguridad

**Cómo buscarlo:**
- **CORS:** grep `cors(`, `Access-Control-Allow-Origin`. Bandera roja: `origin: '*'` o `origin: true` con `credentials: true`. Debe ser allowlist de orígenes.
- **Headers:** ¿hay `helmet` o equivalente? Verifica CSP, HSTS, X-Content-Type-Options, X-Frame-Options.
- **Errores verbosos:** stack traces o mensajes de DB devueltos al cliente en producción.
- **Debug/flags:** `debug: true`, `NODE_ENV` mal puesto, endpoints de introspección (GraphQL introspection, `/actuator`, `/debug`) abiertos.
- **Defaults:** credenciales por defecto, puertos de admin expuestos.

**Fix:** CORS con allowlist explícita; helmet configurado; errores genéricos al cliente + detalle solo en logs; apagar debug/introspection en prod.

## A07 — Fallas de autenticación

**Cómo buscarlo:**
- ¿El guard de auth es global (default-closed) o hay que recordar ponerlo por ruta? Lo segundo es frágil.
- Grep rutas marcadas públicas/`@Public`/`skipAuth` — ¿cada una está justificada?
- Rate limit en login/registro/reset/OTP: grep `throttle`, `rateLimit`, `RateLimiter`. Ausente en login = fuerza bruta.
- Manejo de sesión: ¿tokens largos sin expiración? ¿refresh no rota? ¿logout no revoca?

**Fix:** guard global + excepciones justificadas; rate limit en superficies de auth; sesiones cortas + refresh rotatorio revocable; 2FA para admins. Ver saas-multitenant.md (segregación).

## CSRF

**Cómo buscarlo:** apps con sesión por cookie. ¿Hay protección CSRF (double-submit token, SameSite)? Las APIs con Bearer token en header no necesitan CSRF (el token no viaja automático), pero las que aceptan cookie de sesión sí.

**Fix:** CSRF double-submit con comparación timing-safe, o SameSite=Strict/Lax + verificación de origen. Nota para tests: un 403 por CSRF puede enmascarar un test de permisos — usa Bearer para aislar (ver adversarial-testing.md).

## XSS

**Cómo buscarlo:** grep `dangerouslySetInnerHTML`, `innerHTML`, `v-html`, `bypassSecurityTrust`, template strings que meten input en HTML sin escapar.

**Fix:** escapar por defecto (los frameworks modernos lo hacen); sanitizar HTML permitido con una librería (DOMPurify); CSP como segunda barrera.

## SSRF

**Cómo buscarlo:** grep `fetch(`, `axios(`, `http.get(`, `request(` donde la URL viene (total o parcialmente) del cliente. Común en: webhooks configurables, "importar desde URL", generación de thumbnails, integraciones.

**Fix:** allowlist de dominios/esquemas; bloquear IPs privadas/loopback/metadata (`169.254.169.254`, `10.*`, `127.*`, `localhost`); resolver DNS y validar la IP resultante; sin redirecciones automáticas a destinos no validados.

## A06 — Supply chain (dependencias)

**Cómo buscarlo:**
- Corre `npm audit` / `pip-audit` / `govulncheck`. Crítico/High = hallazgo.
- ¿Hay lockfile commiteado (`package-lock.json`, `poetry.lock`)? Sin lockfile = builds no reproducibles.
- Typosquatting: revisa nombres de paquetes sospechosos o poco conocidos con pocas descargas.
- Scripts de postinstall en dependencias no confiables.

**Fix:** `npm audit` en CI que bloquea en crítico; lockfile commiteado; fijar versiones; revisar dependencias nuevas antes de agregarlas.

## Riesgos de LLM / bots (si el proyecto usa IA)

Vectores específicos, a menudo ignorados:

1. **Prompt injection:** input del usuario (o contenido recuperado: emails, docs, mensajes) que contiene instrucciones que secuestran al modelo. Busca dónde entra texto no confiable al prompt.
   - **Fix:** separar instrucciones de sistema de datos del usuario; tratar TODO input y contenido recuperado como no confiable; no darle al modelo capacidades que no debería ejecutar sin confirmación; validar/whitelistear las salidas que disparan acciones.
2. **Exfiltración vía tool-calling:** un modelo con acceso a herramientas puede ser inducido a leer datos de un tenant y mandarlos a otro, o a un canal externo.
   - **Fix:** las tools aplican el MISMO scoping de tenant/permiso que el resto del backend (el modelo no es un actor privilegiado); parametrizar las tools por el tenant del contexto, nunca por lo que el modelo "decida"; loggear y limitar las tools que envían datos afuera.
3. **El bot no inventa (grounding):** el modelo responde SOLO con evidencia recuperada (KB, tool-calling a la DB). Datos como precios/servicios/disponibilidad SIEMPRE por tool-calling a la fuente de verdad, jamás generados por el modelo. Sin evidencia → "no tengo esa información" + ofrecer handoff.
4. **Modo humano apaga el bot:** cuando una conversación está en manos de un humano, el bot NO responde. Verifica que exista y se respete ese estado (un test de que en modo HUMANO el bot no emite mensaje).
5. **Guardrails de tema:** el bot solo habla de los temas del negocio; rechaza salirse de alcance.
6. **Límites de gasto/abuso:** rate limit por tenant en llamadas al LLM; el input del cliente no controla el modelo, tokens ni el costo sin tope.

## Salida de esta dimensión

Cada vector encontrado se registra como hallazgo con la plantilla de audit-playbook.md, con su patrón grep como parte de la evidencia y su fix concreto.
