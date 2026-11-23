# Blueprint: Profesores

Profesores virtuales (Alan Turing, Marie Curie, Adam Smith…) con los que el
alumno conversa por voz o texto dentro de los cursos de Articulate Rise 360
(Rise). Cada agente es una página estática publicada en GitHub Pages que
Rise inserta por iframe. El agente principal es de D-ID; si D-ID no opera,
la página cambia sola a un agente de voz de ElevenLabs.

- Área usuaria: DEA, que asigna los personajes y da seguimiento en la
  Cédula de Seguimiento de materias (ver [Plan de personajes](#4-plan-de-personajes)).
- Patrocinio: DEA.
- Cliente: interno. UCC y GRG son clientes de algunos diplomados (ids
  `diplomadoIA-*-UCC_*` y `diplomadoIA-*-GRG_*`).
- Responsable: @CoriaAlx. Suplente: @rodrig317-cpu. Supervisor y revisor: @ErikV-AG.
- Usuarios: PENDIENTE: ¿cuántos cursos y alumnos usan hoy los agentes?
- Criterio de éxito: PENDIENTE: ¿cómo se mide que el proyecto cumple su objetivo?
- `benito.html` ya no sigue en uso y `content/` (export de Rise «Prueba») no
  tiene uso identificado; los dos se publican tal cual. PENDIENTE: ¿qué es
  `content/` y quién decide si `benito.html` y `content/` se retiran?

## 1. Contexto (C4, nivel 1)

```mermaid
flowchart LR
  alumno([Alumno]) --> rise[Curso en Rise]
  rise -->|iframe| sitio[Sitio de agentes<br/>innovacion-ag.github.io/profesores]
  rise -->|iframe, cursos anteriores| puente[Puente<br/>erikv-ag.github.io/profesores]
  puente -->|iframe con permisos de micrófono| sitio
  sitio --> did[(D-ID<br/>agente de video)]
  sitio --> el[(ElevenLabs<br/>agente de voz de respaldo)]
  editor([Equipo de Innovación]) -->|edita agentes.json| gh[Repositorio Innovacion-AG/profesores]
  gh -->|GitHub Actions| sitio
  dea([DEA]) -->|asigna personaje y fecha de activación| cedula[(Cédula de Seguimiento<br/>hoja «Personajes cod»)]
  editor -->|pega el código de inserción| cedula
  dea -->|activa el agente| rise
```

## 2. Contenedores (C4, nivel 2)

```mermaid
flowchart LR
  subgraph repo[Repositorio Innovacion-AG/profesores]
    lista_json[agentes.json<br/>id, nombre, keys]
    src[src/<br/>plantilla.html, agente.js,<br/>agente.css, tablero.html]
    construir[scripts/construir.py<br/>valida y genera _site/]
    urls[scripts/urls_publicadas.txt]
    flujo[.github/workflows/publicar.yml]
  end
  subgraph pages[GitHub Pages]
    paginas[53 páginas de agente<br/>id.html]
    lista_html[lista.html]
    tablero[tablero.html]
    ids[ids-publicados.json]
    sueltos[benito.html y content/<br/>publicados tal cual]
  end
  flujo --> construir
  lista_json --> construir
  src --> construir
  urls --> construir
  ids -->|compara ids en línea| construir
  construir -->|deploy-pages| pages
  paginas -->|embed v2| did[(D-ID<br/>agent.d-id.com)]
  paginas -->|widget convai| unpkg[(unpkg.com)]
  unpkg --> el[(ElevenLabs)]
  tablero --> fuentes[(Google Fonts)]
  puente[Repositorio ErikV-AG/profesores<br/>index.html y 404.html] -->|incrusta o redirige| paginas
```

## 3. Flujo principal

```mermaid
sequenceDiagram
  actor Alumno
  participant Rise
  participant Pagina as Página del agente
  participant DID as D-ID
  participant EL as ElevenLabs
  Alumno->>Rise: abre la lección
  Rise->>Pagina: carga id.html en un iframe
  Pagina->>DID: carga el embed v2 con agent id y client key
  Pagina-->>Alumno: cargador y botón «Iniciar conversación» en español
  Alumno->>Pagina: pulsa «Iniciar conversación»
  Pagina->>DID: abre la sesión
  alt D-ID opera
    DID-->>Alumno: conversación por video, voz o texto
  else sin créditos, sin sesiones libres, no carga o no conecta
    Pagina->>DID: cuelga («End call» o quita su contenedor) y espera hasta 3 s
    Pagina->>EL: ya colgado D-ID, carga el widget con elevenlabs_agent_id
    EL-->>Alumno: conversación por voz
  end
  Alumno->>Pagina: sale de la página (pagehide) o pulsa «✕ Terminar»
  Pagina->>DID: pulsa «End call» y cierra sus conexiones
  Pagina->>EL: cierra la conversación
```

La página registra cada conexión y micrófono que se abre (script inicial
de `src/plantilla.html`) para cerrar a la fuerza solo las de D-ID,
reconocidas por su destino. El embed de D-ID no expone una función para
colgar. Detalle y límites en [operación › Cierre de sesiones](operacion.md#cierre-de-sesiones)
y en la [decisión 0007](decisiones/0007-se-cuelga-d-id-antes-del-respaldo-y-al-salir-sin-campo-tipo.md).

Si el agente no tiene `elevenlabs_agent_id` (13 de 53), la página muestra
el chat de texto de D-ID o el aviso «El profesor no está disponible por el
momento».

## 4. Plan de personajes

El plan de personajes vive fuera del repositorio, en la
[Cédula de Seguimiento de materias](https://docs.google.com/spreadsheets/d/1I9YYzEdpFYIqBMzy-pgykguDBPYmFZ15uAWOYCB8ObA/edit#gid=764442467)
de DEA, hoja «Personajes cod». Cada fila es una materia con su personaje,
su fecha de activación (columna «ACTIVACIÓN (INICIO EN CAMPUS)») y su
código de inserción (columna «Código de insersión»).

- **La Cédula es la fuente del plan**: qué personaje lleva cada materia y
  cuándo se activa.
- **`agentes.json` es la fuente de lo publicado**: qué agentes existen en el
  sitio y con qué ids de D-ID y ElevenLabs.

Una materia puede estar en la Cédula sin agente publicado todavía. Un agente
publicado sin fila en la Cédula es un hueco en el seguimiento de DEA.
Ver la [decisión 0006](decisiones/0006-la-cedula-de-dea-es-la-fuente-del-plan-de-personajes-y-agentes-json-la-de-lo-publicado.md).

```mermaid
sequenceDiagram
  participant DEA
  participant Cedula as Cédula (hoja «Personajes cod»)
  participant Inn as Innovación
  participant Prov as D-ID y ElevenLabs
  participant Repo as agentes.json
  participant Rise
  DEA->>Cedula: asigna el personaje y la fecha de activación
  Inn->>Inn: produce el video del personaje
  Inn->>Prov: crea el agente en D-ID y en ElevenLabs
  Inn->>Repo: da de alta el agente (commit directo en main)
  Repo-->>Inn: Actions publica id.html y lista.html
  Inn->>Cedula: pega el código de inserción copiado de lista.html
  DEA->>Rise: activa el agente pegando el código en la materia
```

## 5. Dependencias externas

| Servicio | Uso | Cuenta titular | Plan | Ubicación de la credencial |
| --- | --- | --- | --- | --- |
| D-ID | Agentes de video (embed v2). Las client keys solo funcionan desde los dominios permitidos: `innovacion-ag.github.io` y `erikv-ag.github.io`. | Cuenta «Proyectos Innovación». PENDIENTE: ¿quién es el titular? | PENDIENTE: ¿qué plan? Los créditos se renuevan el día 18 de cada mes. | Gestor de contraseñas del área, grupo profesores |
| ElevenLabs | Agente de voz de respaldo (widget `@elevenlabs/convai-widget-embed`). 40 de 53 agentes lo tienen. | Cuenta del área. PENDIENTE: ¿quién es el titular? | PENDIENTE: ¿qué plan y cuándo se renueva? | Gestor de contraseñas del área, grupo profesores |
| HeyGen | Videos de presentación de cada agente. No aparece en el código. | PENDIENTE: ¿cuenta del área titular? | PENDIENTE: ¿qué plan? | Gestor de contraseñas del área, grupo profesores |
| GitHub Pages y Actions | Construcción y publicación del sitio. | Organización Innovacion-AG | PENDIENTE: ¿plan de la organización? | Sin credenciales propias; Actions usa el token del flujo. |
| unpkg.com | Entrega el script del widget de ElevenLabs. | Servicio público | Gratuito | No aplica |
| Google Fonts | Tipografías de `tablero.html`. | Servicio público | Gratuito | No aplica |
| Vimeo | Video de introducción de `benito.html`, página que ya no sigue en uso. | PENDIENTE: ¿cuenta titular del video? | PENDIENTE: ¿qué plan? | Gestor de contraseñas del área, grupo profesores |
| Google Sheets | Cédula de Seguimiento de materias de DEA, hoja «Personajes cod»: plan de personajes y códigos de inserción. Fuera de este repositorio. | DEA | Google Workspace de la institución | Acceso con la cuenta institucional |
| Articulate Rise 360 | Cursos que insertan los agentes por iframe. El montaje lo hace DEA. | PENDIENTE: ¿cuenta de DEA? | PENDIENTE | Fuera de este repositorio |

## 6. Entornos

| Entorno | Dirección | Despliegue |
| --- | --- | --- |
| Producción | https://innovacion-ag.github.io/profesores/ (`lista.html`, `tablero.html`, `<id>.html`) | GitHub Actions (`publicar.yml`) en cada push a `main`, solo si la validación pasa. |
| Puente | https://erikv-ag.github.io/profesores/ | GitHub Pages desde la rama `main` de `ErikV-AG/profesores`. Dentro de un iframe incrusta la página nueva; abierta directo, redirige. |
| Revisión de PR | Checks del PR | `publicar.yml` valida y construye sin publicar. |
| Prueba en el dominio real | https://innovacion-ag.github.io/profesores/prueba/ (`iframes.html` y una página por agente de prueba) | Solo mientras exista la carpeta `prueba/` en `main`: se publica tal cual junto con producción, con un aviso. Se genera con `construir.py --prueba` desde la rama del cambio y se borra al terminar la prueba. |
| Local | http://localhost:8000/lista.html | `python3 scripts/servidor.py`. D-ID no conecta desde `localhost`. |

## 7. Datos

- El sitio no guarda datos: no hay formularios, base de datos ni
  almacenamiento en el navegador.
- La voz y el texto del alumno se envían
  directamente desde su navegador a D-ID y, en el respaldo, a ElevenLabs.
  Son datos personales de alumnos procesados por terceros.
- El seguimiento de la cobertura legal de ese procesamiento está en el
  registro RAID de la bitácora ejecutiva.
- PENDIENTE: ¿D-ID y ElevenLabs guardan las conversaciones según la
  configuración de cada agente?
- `agentes.json` contiene client keys de D-ID. Son llaves de navegador
  protegidas por la lista de dominios permitidos y se publican en cada
  página por diseño.

## 8. Riesgos y deuda técnica

Los riesgos y la deuda técnica del proyecto se registran en el registro
RAID de la bitácora ejecutiva (acceso interno). Para consultarlo o
agregar uno, preguntar en `#proj-profesores`. La deuda técnica que tiene
trabajo concreto se registra además como incidencia con la etiqueta
`deuda-tecnica`.
