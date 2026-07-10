# Las preguntas antes de planear

Cinco capas, en orden. Cada pregunta trae su porqué y el error que previene — si al leerla ya conoces la respuesta (por el código, los docs o la conversación), NO la hagas: preguntar lo ya respondido quema la paciencia del usuario y diluye las preguntas que sí importan.

**Mecánica:** una pregunta por mensaje. Opción múltiple siempre que el espacio de respuestas sea enumerable (2-4 opciones + tu recomendación marcada). Abierta solo cuando genuinamente no puedes anticipar la respuesta. Detente cuando una respuesta más ya no cambiaría el plan: el objetivo no es completar el cuestionario, es eliminar las incógnitas que cambian decisiones.

---

## Capa A — Intención real (qué problema es, no qué feature es)

El usuario suele llegar con una solución ("quiero un botón que exporte a Excel"), no con el problema ("contabilidad me pide estos datos cada viernes"). Planear la solución literal sin entender el problema produce features correctas que no sirven.

| Pregunta | Por qué | Error que previene |
|---|---|---|
| ¿Qué problema resuelve esto y para quién? | La solución propuesta puede no ser la mejor para ese problema. | Construir lo pedido en vez de lo necesitado. |
| ¿Cómo se ve el éxito? ¿Qué podrá hacer alguien que hoy no puede? | Convierte deseo en criterio verificable. | "Terminado" sin definición → discusión al entregar. |
| ¿Qué pasa si NO se hace? ¿Qué duele hoy? | Calibra urgencia y tamaño justificable de la solución. | Sobre-ingeniería para un dolor menor (o al revés). |
| ¿Esto es para producción, un experimento, o una demo? | El estándar de calidad, tests y robustez cambia radicalmente. | Aplicar rigor de producción a un prototipo desechable, o viceversa. |

## Capa B — Alcance (el borde importa más que el centro)

| Pregunta | Por qué | Error que previene |
|---|---|---|
| ¿Qué queda explícitamente FUERA de esta versión? | Lo no-dicho se asume dentro; el alcance solo crece si no se acota por escrito. | Scope creep silencioso; features fantasma que el usuario asumió incluidas. |
| ¿Cuál es la versión mínima que ya sirve para algo? | Casi siempre existe un corte intermedio entregable que valida la dirección. | Big-bang de semanas sin feedback intermedio. |
| ¿Qué decisiones de aquí son reversibles y cuáles no? | Las irreversibles (schema público, formato de API, IDs, contratos externos) merecen 10x más análisis; las reversibles se deciden rápido y se corrigen barato. | Gastar el análisis en lo reversible y decidir a la ligera lo irreversible. |
| ¿Hay casos borde que ya sabes que existen? (usuarios sin X, datos viejos, concurrencia) | El usuario suele conocer los casos raros de su dominio y no mencionarlos hasta que truenan. | Diseñar solo para el happy path. |

## Capa C — Contexto existente (el plan vive en un repo, no en el vacío)

Esta capa se responde mayormente EXPLORANDO, no preguntando. Pregunta solo lo que el repo no puede decirte.

| Pregunta / verificación | Por qué | Error que previene |
|---|---|---|
| ¿Qué código existente toca o reemplaza esto? (explorar) | Todo cambio aterriza sobre patrones, convenciones y deuda ya existentes. | Reinventar algo que ya existe; romper un patrón establecido. |
| ¿Hay spec, tracker o backlog contra el cual reconciliar ANTES de escribir el plan? | Si existe una lista previa de tareas/requisitos, cada ítem se incluye o se re-planifica explícitamente con el usuario — nunca se omite en silencio. | Features acordadas que desaparecen del plan porque nadie cruzó las listas (error real y caro). |
| ¿Qué patrón existente debe seguir esto? (capas, validación, tests, naming) | La consistencia vale más que la preferencia personal del que escribe. | Un módulo "isla" con convenciones propias que nadie más entiende. |
| ¿Este cambio altera contratos que otros consumen? (API, schema, eventos, jobs) | Los consumidores rotos aparecen en producción, no en tus tests. | Breaking changes accidentales. |

## Capa D — Riesgos (encuentra la suposición que tumba todo)

| Pregunta | Por qué | Error que previene |
|---|---|---|
| ¿Cuál es LA suposición que, si es falsa, invalida el plan entero? | Todo plan descansa en 1-2 suposiciones estructurales (la API externa lo permite, el dato existe, el volumen cabe). Se verifican PRIMERO, con un spike si hace falta. | Descubrir en la tarea 8 de 10 que la premisa de la tarea 1 era falsa. |
| ¿Qué parte tiene más incertidumbre técnica? | Esa parte va al inicio del orden de ejecución, no al final. | Dejar lo difícil para el final y estrellarse con todo lo demás ya construido encima. |
| ¿Hay datos/estados legacy que no cumplen las reglas nuevas? | El código nuevo asume invariantes que los datos viejos violan (nulls, duplicados, estados imposibles). | Migraciones que truenan en producción con datos reales. |
| ¿Qué puede fallar en la frontera con sistemas externos? (timeouts, webhooks duplicados, rate limits) | Lo externo falla de formas que lo local no; la idempotencia y los reintentos se diseñan, no se parchan. | Efectos duplicados y estados inconsistentes bajo fallas parciales. |
| ¿Esto toca seguridad, datos personales o dinero? | Si sí, el plan necesita sus controles y tests adversariales como tareas de primera clase, no como "pendiente". | Seguridad como afterthought. |

## Capa E — Restricciones (la realidad del proyecto)

| Pregunta | Por qué | Error que previene |
|---|---|---|
| ¿Hay plazo o evento que fija fecha? | Con fecha dura, el alcance es la variable; sin fecha, la calidad manda. | Recortar calidad cuando había que recortar alcance. |
| ¿Presupuesto de infra/servicios? (¿optimizamos costo o disponibilidad?) | Cambia decisiones de arquitectura enteras. | Diseñar para una escala/presupuesto que no es el real. |
| ¿Qué compatibilidad hay que preservar? (versiones, clientes viejos, URLs públicas) | Lo público es contrato aunque no esté documentado. | Romper consumidores desconocidos. |
| ¿La migración puede requerir downtime o debe ser en caliente? | Define si el plan necesita estrategia expand/contract y pasos de despliegue propios. | Planear como si la DB estuviera vacía. |

---

## Anti-patrones al preguntar

- **Interrogatorio completo:** hacer las 20 preguntas. Haz solo las que cambian el plan de ESTE proyecto; 4-6 suele bastar.
- **Pregunta-relleno:** preguntar lo que el código ya responde. Explora primero.
- **Preguntar sin recomendar:** en opción múltiple, marca siempre tu recomendación y su porqué — el usuario decide mejor contra una postura que contra un menú neutro.
- **Aceptar respuestas vagas en lo irreversible:** si la respuesta a una decisión irreversible es "como veas", presenta las opciones con consecuencias concretas y pide decisión explícita. En lo reversible, "como veas" es luz verde: decide tú y documenta.
