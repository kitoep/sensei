# Esqueleto andante — deploy antes que features

## Qué es

El **esqueleto andante** (walking skeleton) es la versión más delgada posible del sistema que recorre TODA la ruta de punta a punta y está DESPLEGADA: request → auth → lógica mínima → base de datos migrada → respuesta, con CI/CD que la publica automáticamente. Hace casi nada — pero lo hace en el entorno real.

**Por qué funciona:** los problemas de integración (config, secretos, migraciones, red, permisos del hosting) son los que matan calendarios, y solo aparecen al integrar. El esqueleto los fuerza a aparecer la semana 1, cuando arreglarlos cuesta minutos, en lugar de la semana 12, cuando hay 40 features encima.

## Milestone 0 obligatorio (antes de cualquier feature)

Checklist — el milestone 0 se declara terminado cuando TODO esto es verdad:

1. [ ] El repo tiene CI en verde: typecheck + tests + build en cada PR.
2. [ ] `docker compose up` (o equivalente) levanta el entorno local completo con dependencias reales.
3. [ ] Existe UNA migración aplicada (aunque sea una tabla trivial) y el CI migra desde cero.
4. [ ] Hay UN endpoint end-to-end funcionando: pega a la DB y responde (con auth si el sistema la lleva — y la lleva default-closed desde ya).
5. [ ] Ese endpoint está DESPLEGADO en un entorno accesible por URL, publicado por el pipeline (no a mano).
6. [ ] Los secretos del entorno desplegado están en el gestor del hosting, no en el repo.
7. [ ] Hay UN test de integración que ejerce esa ruta contra la DB real y corre en CI.

Si el punto 5 falla, el milestone no está terminado. "Corre en mi máquina" no es un esqueleto andante; es un esqueleto acostado.

## Cómo ordenar los primeros 5 milestones: por riesgo, no por facilidad

Regla de oro: **lo que puede invalidar el proyecto se prueba primero, en su versión más barata.** El orden natural (y equivocado) es empezar por lo fácil y conocido; el orden correcto es empezar por lo que no sabes si funcionará.

Procedimiento:

1. Toma la respuesta a la pregunta 12 del intake ("¿qué invalidaría el proyecto?") y las integraciones de la pregunta 10.
2. Para cada riesgo, define el **spike más barato que lo confirma o descarta** (ej.: "mandar y recibir UN mensaje real por la API de WhatsApp con una cuenta aprobada", "cobrar $1 real con la pasarela", "el modelo extrae el dato correcto en 20 casos reales").
3. Ordena: Milestone 0 = esqueleto andante → Milestones 1-2 = spikes de los riesgos que invalidan → Milestones 3-5 = el corazón del producto sobre lo ya validado.
4. Lo reversible y conocido (CRUDs, pantallas de admin, ajustes visuales) va AL FINAL. Siempre se puede; nunca invalida nada.

Prueba de sanidad del plan: si tu milestone 1 es algo que sabes con certeza que puedes construir, el orden está mal — estás empezando por lo cómodo, no por lo riesgoso.

## Anti-patrones (con su consecuencia real)

| Anti-patrón | Por qué seduce | Cómo termina |
|---|---|---|
| Empezar por el CRUD fácil | Progreso visible rápido | Semana 8: el CRUD perfecto orbita un corazón que nunca se validó y no funciona |
| Deploy "para el final" | "Primero que funcione local" | El deploy revela config/secretos/migraciones rotos con todo el sistema encima; días de arreglos bajo presión |
| Features sin tests "por ahora" | "Voy rápido y luego los agrego" | Nadie los agrega; el primer refactor rompe 3 cosas en silencio y se pierde la confianza en el código |
| "Luego lo conectamos" (integración diferida) | Cada pieza avanza en paralelo | Las piezas no embonan: contratos distintos, supuestos distintos, semanas de pegamento |
| Validar el riesgo con mocks | El mock siempre funciona | La API real tiene cuotas, aprobaciones y latencias que el mock jamás mostró — y el proyecto dependía de eso |
| Construir para escala imaginaria | "Mejor lo dejamos listo para crecer" | Meses en infra para miles de usuarios que aún no existen, cero validación del producto |

## Salida de la fase

Milestone 0 completado (checklist arriba, con evidencia: URL desplegada + CI verde) y tracker con los milestones 1-5 ordenados por riesgo, cada uno con su criterio de terminado medible.
