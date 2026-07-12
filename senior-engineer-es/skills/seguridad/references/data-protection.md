# Protección de datos y compliance

Marco operativo. No es teoría legal: cada control se traduce en algo verificable en el código. Aplica el que corresponda según los datos que maneje el proyecto.

## Paso 1 — Clasificar los datos (haz esto primero)

Antes de decidir controles, clasifica cada campo que el sistema almacena:

| Clase | Ejemplos | Control mínimo |
|---|---|---|
| **Público** | nombre comercial del negocio, catálogo | ninguno especial |
| **Interno** | métricas agregadas, config no sensible | acceso autenticado |
| **PII** (datos personales) | nombre, email, teléfono, dirección, IP, ID de cliente | cifrado en tránsito, acceso por rol, fuera de logs, sujeto a derechos del titular |
| **Sensible** | salud, biométricos, orientación, religión, CURP/RFC, geolocalización precisa, credenciales | todo lo de PII + cifrado en reposo + acceso auditado + consentimiento explícito |
| **Secretos** | tokens de terceros, API keys, contraseñas | cifrado en reposo (AES-256-GCM) o hash; nunca en claro, logs ni repo |

Regla: si no sabes en qué clase cae un campo, trátalo como Sensible hasta confirmar lo contrario.

## LFPDPPP (México) — aplica a cualquier SaaS que maneje datos de personas en México

Controles verificables:

1. **Aviso de privacidad:** debe existir y ser accesible al titular en el momento de recabar datos. En código: el flujo de registro/alta enlaza o presenta el aviso; se registra el consentimiento (timestamp, versión del aviso).
2. **Consentimiento:** para datos sensibles, consentimiento expreso (checkbox no premarcado, acción afirmativa). Guardar evidencia.
3. **Derechos ARCO** (Acceso, Rectificación, Cancelación, Oposición): debe existir un mecanismo para que el titular ejerza cada uno. En código: endpoint/proceso para exportar los datos de un titular (Acceso), editarlos (Rectificación), borrarlos/anonimizarlos (Cancelación) y dejar de tratarlos (Oposición).
4. **Finalidad y minimización:** no recabar más datos de los necesarios para la finalidad declarada. Cuestiona cada campo nuevo: ¿la finalidad del aviso lo justifica?
5. **Transferencias:** si los datos se comparten con terceros (proveedores, subprocesadores como el hosting o el proveedor de mensajería), el aviso debe declararlo. Documenta los subprocesadores.
6. **Seguridad:** medidas de seguridad proporcionales (cifrado, control de acceso) — cruza con guard-rules.md y saas-multitenant.md.

## GDPR (UE) — si hay o habrá usuarios europeos (o como estándar más exigente)

1. **Base legal:** todo tratamiento necesita una (consentimiento, contrato, interés legítimo…). Documéntala por finalidad.
2. **DSAR (Data Subject Access Request):** derecho de acceso, rectificación, borrado ("derecho al olvido"), portabilidad y oposición. En código: los mismos mecanismos ARCO, más **exportación en formato estructurado y portable** (JSON/CSV).
3. **Minimización y limitación de finalidad:** igual que LFPDPPP.
4. **Retención:** define y aplica plazos de conservación. En código: proceso que borra o anonimiza datos pasado el plazo. No guardar indefinidamente "por si acaso".
5. **Derecho al olvido / anonimización:** al ejercerse, los datos personales se borran o se anonimizan irreversiblemente (no reversible con una tabla de mapeo). Distinguir borrado (elimina la fila) de anonimización (rompe el vínculo con la persona pero conserva agregados).
6. **Privacy by design & by default:** la configuración más protectora es la de fábrica.
7. **Notificación de brechas:** procedimiento para notificar en 72h. En código: logging/alertas que permitan detectar y acotar una brecha.

## PCI-DSS básico — SOLO si el proyecto toca datos de tarjeta

Regla número uno: **no almacenar datos de tarjeta.** La forma correcta de cumplir PCI es no estar en su alcance.

1. **Nunca almacenar PAN completo ni CVV.** El CVV no se guarda jamás, ni cifrado.
2. **Delegar a un procesador certificado** (Stripe, Mercado Pago, etc.): la tarjeta va del cliente al procesador (tokenización del lado del cliente), tu backend solo maneja el token/referencia. Así el PAN nunca toca tus servidores.
3. Si por algún motivo el PAN pasara por tu sistema, entras en alcance PCI completo — evítalo por diseño.
4. En código: busca `card`, `pan`, `cvv`, `cardNumber` almacenándose en DB o logs → hallazgo Critical.

## Controles transversales de datos (siempre)

- **PII fuera de logs (regla dura):** ni emails, ni teléfonos, ni nombres, ni tokens en logs. Loggea identificadores opacos (`user_id`, `trace_id`) y el `tenant_id`. Revisa también los mensajes de error que se devuelven al cliente y los reportes a servicios de terceros (Sentry, etc.) — configúralos para scrubbing de PII.
- **Cifrado en tránsito:** TLS en todos lados (API, DB, colas, servicios internos). Nada de HTTP plano ni conexiones a DB sin SSL en producción.
- **Cifrado en reposo:** datos Sensibles y Secretos cifrados a nivel de campo (AES-256-GCM); el resto, al menos cifrado de disco/volumen del proveedor.
- **Retención y borrado:** plazo definido por tipo de dato; proceso automatizado que borra/anonimiza al vencer. El "botón de anonimizar" debe ser irreversible.
- **Acceso a datos auditado:** para datos Sensibles, registrar quién accedió a qué (log de acceso, sin exponer el dato en el log).

## Gap-analysis (formato para el reporte de audit)

Al auditar compliance, entrega una tabla:

| Control | Marco | ¿Existe? | Evidencia / Gap |
|---|---|---|---|
| Aviso de privacidad + consentimiento | LFPDPPP | ✅/❌ | archivo:línea o "ausente" |
| Mecanismo ARCO / DSAR | LFPDPPP/GDPR | ✅/❌ | ... |
| PII fuera de logs | ambos | ✅/❌ | ... |
| Retención/borrado automatizado | GDPR | ✅/❌ | ... |
| Sin PAN/CVV almacenado | PCI | ✅/❌ | ... |
| Cifrado en reposo de sensibles/secretos | ambos | ✅/❌ | ... |
