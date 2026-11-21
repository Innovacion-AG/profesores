# 0006. La Cédula de DEA es la fuente del plan de personajes y agentes.json la de lo publicado

Estado: aceptado
Fecha: 2026-11-21
Origen: PR #10

## Contexto
DEA asigna los personajes de cada materia y su fecha de activación en la
[Cédula de Seguimiento de materias](https://docs.google.com/spreadsheets/d/1I9YYzEdpFYIqBMzy-pgykguDBPYmFZ15uAWOYCB8ObA/edit#gid=764442467),
hoja «Personajes cod». Innovación crea los agentes y los publica desde
`agentes.json`. Las dos listas describen a los mismos personajes, pero
ninguna documentación decía cuál manda en cada caso ni cómo pasa el código
de inserción de un lado a otro.

## Decisión
- La Cédula es la fuente del plan: qué personaje lleva cada materia y
  cuándo se activa. La mantiene DEA.
- `agentes.json` es la fuente de lo publicado: qué agentes existen en el
  sitio y con qué ids de D-ID y ElevenLabs. Lo mantiene Innovación.
- El enlace entre las dos es el código de inserción. Al dar de alta un
  agente, Innovación copia el iframe de `lista.html` con el dominio
  `innovacion-ag.github.io` y lo pega en la columna «Código de insersión»
  de la Cédula. DEA lo toma de ahí para activar el agente en Rise.

## Consecuencias
- El repositorio no guarda fechas de activación ni asignaciones de
  materias. Para saber qué falta por producir se consulta la Cédula.
- Una materia puede estar en la Cédula sin agente publicado todavía. Un
  agente publicado sin fila en la Cédula queda fuera del seguimiento de DEA.
- Los códigos de la Cédula con el dominio anterior (`erikv-ag.github.io`)
  siguen funcionando por el puente (ver la decisión 0005). Su actualización
  al dominio nuevo se coordina con DEA, porque cambiarlos en la Cédula no
  cambia los iframes que ya están en Rise.
- El flujo depende de un paso manual: pegar el código en la Cédula. Está
  descrito en `docs/operacion.md`, paso 6 del despliegue.

## Evidencia
- PR #10: `catalog-info.yaml`, `docs/blueprint.md` (sección 4) y
  `docs/operacion.md`.
- Hoja «Personajes cod» revisada el 2026-11-21: columnas «PERSONAJE»,
  «ACTIVACIÓN (INICIO EN CAMPUS)» y «Código de insersión». Las filas
  revisadas tienen iframes con `erikv-ag.github.io`.
