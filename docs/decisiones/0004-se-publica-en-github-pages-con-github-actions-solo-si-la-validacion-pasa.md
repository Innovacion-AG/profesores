# 0004. Se publica en GitHub Pages con GitHub Actions solo si la validación pasa

Estado: propuesto
Fecha: 2026-10-08
Origen: reconstruido desde el historial de git

## Contexto
El sitio se publicaba desde la rama `main` tal cual. Con páginas generadas,
un error en `agentes.json` podía publicar un sitio roto o borrar una página
con iframes en cursos de Rise. El dueño del repositorio edita desde
github.com, sin copia local.

## Decisión
Se usa GitHub Pages con origen «GitHub Actions». El flujo
`.github/workflows/publicar.yml` revisa la sintaxis de `src/agente.js`,
valida `agentes.json`, compara contra la versión anterior y contra
`ids-publicados.json` del sitio en línea, construye `_site/` y solo entonces
lo publica. En un PR solo valida y construye.

## Consecuencias
- Un error de edición deja el sitio como estaba y el mensaje explica qué
  corregir, en español.
- No hay costo de hosting ni servidor que mantener.
- Borrar un agente exige un paso explícito («permitir borrar»).
- Se depende de las acciones de GitHub Pages y de su versión de Node.js
  (incidencia #6).

## Evidencia
- 464c339 Publicación en GitHub Pages con GitHub Actions
- `.github/workflows/publicar.yml`
- Ejecuciones fallidas que no publicaron: 41104859775 (2026-10-30) y 42268804735 (2026-11-06)
