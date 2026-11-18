# 0001. Se usa D-ID como proveedor de los agentes de video

Estado: propuesto
Fecha: 2026-04-09
Origen: reconstruido desde el historial de git

## Contexto
Los cursos de Rise necesitan profesores virtuales con rostro y voz con los
que el alumno pueda conversar. Desde el primer commit, las páginas del
repositorio cargan agentes de D-ID.

## Decisión
Se usa el embed v2 de D-ID (`https://agent.d-id.com/v2/index.js`) para cada
agente. Cada agente se identifica con un `did_agent_id` y una
`did_client_key`, que solo funciona desde los dominios permitidos en la
cuenta «Proyectos Innovación».

## Consecuencias
- Se obtiene un agente de video conversacional sin servidor propio: basta
  una página estática.
- El servicio depende de los créditos y de las sesiones simultáneas del plan
  de D-ID; sin ellos el agente no opera (ver 0002).
- La interfaz de D-ID viene en inglés y se traduce en el navegador
  (`src/agente.js`), lo que exige mantener las cadenas cuando D-ID cambia.
- Las client keys quedan públicas en cada página; su protección es la lista
  de dominios permitidos.

## Evidencia
- 8dbb51b Sustitución de archivos de profesores (`did-agent.html` con el embed v2)
- 94c46ef Add Alan_Matematicas.html for D-ID agent integration
- `agentes.json`: campos `did_agent_id` y `did_client_key` en los 53 agentes
- `src/agente.js`: constante `URL_DID`
