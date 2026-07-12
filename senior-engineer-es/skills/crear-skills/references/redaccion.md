# Redacción y validación: convenciones de la suite

## Convenciones no negociables

| Regla | Detalle |
|---|---|
| SKILL.md ligero, references con el detalle | El modelo carga el SKILL.md al activar y abre cada reference solo en su fase. Detalle en el SKILL.md = contexto quemado en vano. |
| Description en inglés | Así indexa Claude Code. El contenido va en el idioma nativo de la suite. |
| Doctrina firma | Incluir "Violar la letra de estas reglas es violar su espíritu" (o su equivalente en inglés) cerca del principio. Cierra la salida de "esta vez no aplica". |
| Referencias a otros skills SIEMPRE condicionales | "usa el skill `reparador` si está instalado; si no, ..." Los skills se venden sueltos: ninguno puede depender de otro para funcionar. |
| Cero guiones medios (—) | Delatan texto de IA. Punto, coma, dos puntos o paréntesis. Verifícalo con grep antes de dar por terminado. |
| Español natural, no calco | "Hard gate" no es "puerta dura": es "regla que no se salta". Si una frase suena a traducción, se reescribe como la diría un dev. |
| Nombres de archivo sin acentos ni eñes | `diseno.md`, no `diseño.md`. Compatibilidad entre sistemas. |

## Dos idiomas: versiones nativas, no traducciones

Cada versión de la suite se escribe como si fuera la original de su idioma:

- Las tablas de frases prohibidas atrapan lo que el modelo escribe EN ESE idioma. "Debería funcionar" y "Should work" son entradas distintas escritas por separado, no traducciones.
- La terminología es la del oficio en ese idioma: en inglés walking skeleton, expand/contract, blind `git add .`; en español la forma en que un dev hispanohablante lo dice de verdad (que muchas veces conserva el término en inglés: eso está bien, calcarlo no).
- Los nombres de los skills son nativos por versión (reparador / fixer) y el contenido de código, SQL y comandos queda idéntico e intacto en ambas.

## Si delegas la redacción

Delegar a subagentes o a otro modelo es válido y escala bien, con tres condiciones en el encargo:

1. **Rol de experto del dominio** ("actúa como un DBA senior con años en producción"), no "escribe un documento sobre bases de datos".
2. **El propósito del paquete completo**: para qué existe la suite y contra qué modelo-falla se escribe. Sin ese contexto, el redactor produce documentación genérica.
3. **Las fallas baseline del paso de diseño**, literales. Son la especificación real del skill.

## Validación personal: la puerta que no se delega

Cuando el redactor termina, su reporte NO es la validación. La validación es:

1. Leer TODO lo escrito, archivo por archivo, completo. Sin excepciones por longitud.
2. Revisar contra tu propio criterio de experto: ¿el contenido técnico es correcto? ¿tú lo dirías así? ¿las puertas son verificables? Lo que no esté a tu nivel, se corrige ahí mismo.
3. Checar la mecánica: frontmatter válido, references citados que existen con ese nombre exacto, referencias cruzadas condicionales, grep de guiones medios en cero, grep de calcos conocidos.
4. Consistencia con la suite: mismo esqueleto de secciones, doctrina firma presente, tono parejo con los skills vecinos.

Los errores del redactor viven en lo que su resumen no menciona. Por eso se lee el archivo, no el resumen.
