# Cimientos — barato el día 1, carísimo el mes 6

Cada ítem se decide EXPLÍCITAMENTE: se aplica, o se descarta con razón escrita en `docs/01_decisiones.md`. Nada se omite en silencio. El formato de cada ítem: **Regla → Bien se ve así → Señal de que se hizo mal → Costo de retrofit**.

## 1. Estructura de repo

**Regla:** un solo repo (monorepo con workspaces si hay varias apps) hasta que exista una razón operativa real para separar (equipos distintos deployando independiente).
**Bien:** `apps/` (web, api, worker), `packages/` (código compartido: tipos, validación), un solo lockfile, un solo CI.
**Mal:** dos repos que hay que versionar en pareja; tipos duplicados a mano entre front y back.
**Retrofit:** fusionar repos separados = semanas de historia de git, CI y permisos. Separar un monorepo después es fácil; unir repos no.

## 2. Config por entorno y secretos

**Regla:** desde el commit 1: config solo por variables de entorno, `.env.example` commiteado con TODAS las claves (sin valores), `.env` en `.gitignore`, y validación de env al boot (la app truena al arrancar si falta una variable, no a las 3am cuando se usa).
**Bien:** un módulo `config` único que lee, valida y tipa el env; nadie hace `process.env.X` suelto por el código.
**Mal:** un secreto commiteado "temporalmente" (queda en la historia de git PARA SIEMPRE; rotarlo es la única salida); valores hardcodeados que difieren entre dev y prod.
**Retrofit:** purgar un secreto de la historia de git + rotarlo en todos los servicios. Horas-días, y el riesgo ya corrió.

## 3. CI desde el commit 1

**Regla:** pipeline con typecheck + tests + build en verde ANTES de la primera feature. Verde obligatorio para mergear.
**Bien:** el CI corre en cada PR; levanta las dependencias reales (DB, cola) como servicios; un `npm audit` (o equivalente) donde lo crítico bloquea.
**Mal:** "el CI lo configuro cuando haya más código" — para entonces hay 40 errores de typecheck acumulados y nadie los quiere pagar.
**Retrofit:** cada semana sin CI acumula deuda invisible; instalarlo tarde = días arreglando todo lo que hubiera fallado incremental.

## 4. Tests contra dependencias reales

**Regla:** los tests de integración corren contra la base de datos y la cola REALES (docker compose local y en CI), no contra mocks. Los mocks no prueban constraints, transacciones ni políticas de la DB.
**Bien:** `docker compose up` levanta Postgres+Redis; los tests migran un schema limpio y ejercen el comportamiento real.
**Mal:** suite verde con mocks + producción rota por un constraint o una transacción que ningún mock simuló.
**Retrofit:** migrar una suite mockeada a dependencias reales = reescribir los tests. Empezar bien cuesta una tarde.

## 5. Migraciones de DB versionadas

**Regla:** desde la PRIMERA tabla, todo cambio de schema es una migración versionada y commiteada. Nunca `db push` / cambios a mano en ninguna DB compartida.
**Bien:** carpeta de migraciones en el repo; el CI migra desde cero en cada corrida (eso prueba que la cadena completa funciona); dos roles de DB si hay RLS (runtime sin bypass, migraciones con owner).
**Mal:** el schema de prod difiere del repo y nadie sabe desde cuándo.
**Retrofit:** reconstruir el estado real de un schema divergente es arqueología. Días, con riesgo de pérdida de datos.

## 6. Fechas en UTC

**Regla:** la DB guarda TODO en UTC. La zona horaria del usuario/negocio es dato de presentación y se aplica solo en el borde (UI, mensajes).
**Bien:** columnas `timestamptz`; conversión a zona local únicamente al renderizar; la TZ del negocio guardada como dato.
**Mal:** fechas "naive" que funcionan hasta el primer usuario en otra zona o el primer cambio de horario de verano.
**Retrofit:** migrar timestamps ambiguos ya guardados es de los bugs más caros que existen: no sabes qué zona tenía cada fila.

## 7. Logging estructurado sin PII

**Regla:** logs en JSON estructurado con `trace_id` (correlación por request/job) e identificador de contexto (p. ej. tenant); PROHIBIDO loggear PII, tokens o secretos desde el log #1.
**Bien:** un logger central que la app entera usa; los datos personales se referencian por ID, nunca por valor.
**Mal:** `console.log(user)` con email y teléfono — cada log con PII es un incidente de datos esperando su fuga.
**Retrofit:** limpiar PII de logs históricos es imposible (ya se replicaron a la plataforma de logs); solo queda purgar y esperar.

## 8. Manejo de errores global

**Regla:** un handler global de errores desde el esqueleto: toda excepción no controlada termina en (a) respuesta genérica al cliente SIN stack ni detalles internos, (b) log estructurado con el detalle completo y trace_id.
**Bien:** errores de dominio tipados que mapean a códigos HTTP; el stack trace jamás viaja al cliente.
**Mal:** 500 con stack completo expuesto (regalo para atacantes) o, peor, errores tragados en silencio.
**Retrofit:** barato de agregar tarde, pero cada semana sin él son errores de producción invisibles que nadie supo que pasaron.

## 9. Auth default-closed

**Regla:** si hay usuarios, el guard de autenticación es GLOBAL y por defecto TODO endpoint requiere sesión; lo público se marca explícito como excepción (`@Public` o equivalente). Nunca al revés.
**Bien:** endpoint nuevo = protegido sin que nadie recuerde nada; el modelo de roles niega por defecto (deny > allow).
**Mal:** "le agrego auth a los endpoints sensibles" — la lista de sensibles siempre queda incompleta, y el endpoint olvidado es el que aparece en el reporte de vulnerabilidad.
**Retrofit:** auditar cada endpoint existente para invertir el default. Días, con hallazgos vergonzosos garantizados.

## 10. Multi-tenancy (si el intake dijo que sí) — EL irreversible

**Regla:** toda entidad de negocio lleva `tenant_id` (con índice) desde su creación; el tenant JAMÁS viene del cliente (se deriva del token o de un mapeo verificado del canal); y hay una segunda barrera a nivel base de datos (Row-Level Security con rol de app sin bypass, o equivalente) porque la disciplina de aplicación se rompe una vez al año y esa vez basta.
**Bien:** checklist obligatorio para toda tabla nueva: tenant_id + índice → política RLS → queries en contexto de tenant → test de aislamiento cross-tenant (el tenant A jamás ve/afecta datos de B; se espera 404, no 403, para no filtrar existencia).
**Mal:** filtros `where tenant_id = ?` puestos "a mano donde aplica" — el día que un dev lo olvida en un endpoint, dos negocios ven los datos del otro y el producto está muerto.
**Retrofit:** NO SE PUEDE de forma segura. Agregar tenancy después = tocar cada tabla, cada query, cada test, y rezar. Por eso se pregunta en el intake.

## 11. Idempotencia en jobs asíncronos

**Regla:** si hay colas/jobs, todo job es idempotente desde el primero: los retries NO duplican efectos (correos, cobros, reservas). `jobId` determinístico cuando aplique; verificación de existencia ANTES de insertar (no atrapar el error de unique constraint dentro de una transacción — en Postgres la aborta entera).
**Bien:** correr el mismo job 3 veces produce exactamente el mismo estado final; hay un reconciliador de respaldo, pero no es el mecanismo primario.
**Mal:** un retry de la cola manda el mismo correo/cobro dos veces al cliente.
**Retrofit:** medio — pero los duplicados ya enviados a clientes reales no se pueden des-enviar.

## 12. Secretos de terceros cifrados en reposo

**Regla:** tokens/credenciales de servicios externos (APIs, integraciones de clientes) se guardan cifrados (AES-256-GCM con clave en env), NUNCA en claro en la DB. Y al corregir un patrón de manejo de secretos en una app, buscar el MISMO patrón en las demás apps del repo (los bugs de patrón viven duplicados).
**Bien:** helpers `encryptSecret/decryptSecret` compartidos; la clave de cifrado es una env más, rotable.
**Mal:** un dump de DB filtrado expone los tokens de WhatsApp/Stripe de TODOS tus clientes — el incidente deja de ser tuyo y pasa a ser de ellos.
**Retrofit:** medio (migración de cifrado), pero solo si te das cuenta antes que un atacante.

## Salida de la fase

`docs/01_decisiones.md`: tabla con los 12 ítems, columna "aplica/descartado", y la razón de cada descarte. Los descartes sin razón escrita no son válidos.
