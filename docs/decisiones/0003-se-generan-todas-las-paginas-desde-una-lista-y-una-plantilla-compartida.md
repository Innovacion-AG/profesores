# 0003. Se generan todas las páginas desde una lista y una plantilla compartida

Estado: propuesto
Fecha: 2026-10-08
Origen: reconstruido desde el historial de git

## Contexto
Cada agente era un archivo `.html` copiado a mano. Una mejora (traducción,
compuerta) había que repetirla en decenas de archivos, y los nombres de las
páginas ya estaban en iframes de cursos publicados.

## Decisión
Se usa `agentes.json` como fuente única de datos de cada agente y
`src/plantilla.html`, `src/agente.js` y `src/agente.css` como página, lógica
y estilo compartidos. `scripts/construir.py` valida la lista y genera una
página por agente con el mismo nombre que tenía. Las páginas anteriores se
conservan en `_referencia/paginas_anteriores/` y las 42 URL de entonces
quedan fijas en `scripts/urls_publicadas.txt`.

## Consecuencias
- Una mejora en `src/` llega a todos los agentes en la siguiente
  publicación; también un error, por eso `src/` no se toca sin consultar.
- Agregar un agente es editar un bloque de JSON desde el navegador.
- La construcción impide borrar o renombrar una URL publicada.
- `benito.html` y `content/` quedan fuera de la lista y se publican tal cual.

## Evidencia
- 866631f Fase 1: inventario de agentes en agentes.json
- dec39ff Plantilla, lógica y estilo compartidos por todos los agentes
- d39299b construir.py valida agentes.json y genera _site/; servidor local
- 6343dbf Mueve las páginas viejas a _referencia/paginas_anteriores y agrega README
- 057695a Merge pull request #2 from ErikV-AG/refactor-plantilla
