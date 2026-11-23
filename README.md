# Profesores — agentes conversacionales

## Qué es

Cada agente (Alan Turing, Marie Curie, Adam Smith…) es una página que se
inserta por iframe en las materias de Rise:

    https://innovacion-ag.github.io/profesores/<id>.html

Hasta noviembre de 2026 el sitio estuvo en `erikv-ag.github.io/profesores`.
Esas direcciones siguen funcionando: el repositorio `ErikV-AG/profesores`
es un puente que incrusta la página nueva cuando se abre en un iframe (así
conserva el permiso de micrófono de los cursos de Rise) y redirige cuando
se abre directamente. No lo borres mientras haya cursos con la dirección
anterior.

Todas las páginas salen de **una sola lista**, [`agentes.json`](agentes.json),
y comparten la misma lógica (`src/agente.js`) y el mismo estilo
(`src/agente.css`). Una mejora en esos archivos llega a todos los agentes a
la vez. No se edita ninguna página `.html` a mano.

Objetivo: dar a los cursos de Rise profesores virtuales con los que el alumno
conversa por voz o texto.

## Estado y responsables

| | |
| --- | --- |
| Estado | En operación: 53 agentes publicados, con iframes en cursos de Rise |
| Responsable | @CoriaAlx |
| Suplente | @rodrig317-cpu |
| Supervisor y revisor | @ErikV-AG |
| Área usuaria | DEA (asigna los personajes, monta los agentes en Rise y da seguimiento en la cédula) |
| Patrocinio | DEA |
| Credenciales | Gestor de contraseñas del área, grupo profesores |
| Carpeta de Drive | https://drive.google.com/drive/folders/1HXtWZWCgBk6wj0-Q4GHy_we8GcY26FaU |

## Arranque

Requisitos: git, Python 3 (biblioteca estándar) y Node.js para
`node --check`. No se instala nada más. En local D-ID no conecta, porque sus
client keys solo aceptan `innovacion-ag.github.io` y `erikv-ag.github.io`;
se ven el cargador, la compuerta y el aviso.

1. Clonar el repositorio:
   `git clone https://github.com/Innovacion-AG/profesores.git && cd profesores`
2. Validar la lista y generar el sitio: `python3 scripts/construir.py`.
   Debe terminar con «✓ Sitio generado en _site/ … y verificado». Para
   solo validar: `python3 scripts/construir.py --validar`.
3. Revisar la sintaxis de la lógica: `node --check src/agente.js`.
4. Servir el sitio: `python3 scripts/servidor.py`.
5. Abrir http://localhost:8000/lista.html y entrar a un agente.
6. Para ver un agente conectado a D-ID, abrir su página publicada:
   https://innovacion-ag.github.io/profesores/<id>.html

Para editar un agente sin clonar nada, seguir «Agregar o cambiar un agente».

## Estructura del repositorio

| Ruta | Contenido |
| --- | --- |
| `agentes.json` | Lista de agentes: fuente única de datos (nombre, ids y client key de D-ID, id de ElevenLabs). |
| `src/` | `plantilla.html`, `agente.js` y `agente.css` compartidos por todos los agentes; `tablero.html`. |
| `scripts/construir.py` | Valida `agentes.json` y genera `_site/`. |
| `scripts/servidor.py` | Sirve `_site/` en local como GitHub Pages. |
| `scripts/urls_publicadas.txt` | Las 42 páginas que existían al migrar; no pueden desaparecer. |
| `.github/workflows/publicar.yml` | Valida, construye y publica en GitHub Pages. |
| `benito.html`, `content/` | Se publican tal cual. `benito.html` ya no sigue en uso. |
| `_referencia/` | Código de referencia y páginas anteriores; no se publica. |
| `docs/` | Blueprint, operación y decisiones. |
| `catalog-info.yaml` | Ficha del proyecto para el catálogo. |

## Documentación

- [Blueprint: arquitectura, dependencias, datos y riesgos](docs/blueprint.md)
- [Operación: despliegue, reversión, monitoreo y tareas periódicas](docs/operacion.md)
- [Decisiones](docs/decisiones/)

## Dónde pedir ayuda

Canal `#proj-profesores` en Slack.

## Agregar o cambiar un agente

1. Abre [`agentes.json`](agentes.json) y pulsa el lápiz (**Edit this file**).
2. Copia un bloque existente y cámbialo. Ejemplo:

   ```json
   "calculo_alan": {
     "nombre": "Alan Turing · Cálculo I",
     "did_agent_id": "v2_agt_VRr-daUT",
     "did_client_key": "ck_j4tRwGC0uPBeVPavOeiyA",
     "elevenlabs_agent_id": ""
   },
   ```

   - **La llave** (`calculo_alan`) es el nombre de la página:
     `…/profesores/calculo_alan.html`. Solo letras sin acento, números,
     `_` y `-`. **No cambies la llave de un agente que ya está en un curso**:
     su iframe dejaría de funcionar.
   - `did_agent_id` y `did_client_key`: en D-ID Studio → Agents → Embed
     (`data-agent-id` y `data-client-key`). La key debe permitir los dominios
     `innovacion-ag.github.io` y `erikv-ag.github.io`.
   - `elevenlabs_agent_id`: el id del agente de voz de respaldo en ElevenLabs
     (empieza con `agent_`). Déjalo en `""` si no hay.
   - Cada bloque va separado del siguiente por una coma; el último no lleva
     coma.
3. Pulsa **Commit changes…** y deja **Commit directly to the `main` branch**.
   Las altas y los cambios de agentes en `agentes.json` se hacen directo en
   `main`, sin pull request: la validación de **Actions** impide publicar un
   error.

Los cambios a `src/`, a `scripts/` o a la publicación (`.github/`) van en
una rama nueva con pull request (**Create a new branch… and start a pull
request**), para que la revisión corra antes de publicar.

## Para colaboradores: no subas páginas `.html` a la raíz

Antes cada agente era un archivo `.html` copiado a mano. Ahora cada agente
es una entrada de `agentes.json` y la página se genera sola, con la
compuerta a ElevenLabs y la traducción completa.

Si alguien sube un `.html` directo a la raíz (cualquiera menos
`benito.html`), la publicación **no se detiene**: el archivo se publica tal
cual, pero sin compuerta ni traducción nueva, y en **Actions** aparece un
aviso amarillo: «*archivo*.html se publicó sin pasar por agentes.json…».
Para pasarlo a la lista sin que su URL deje de funcionar:

1. Agrega el agente a `agentes.json` con el mismo id (el nombre del archivo
   sin `.html`) y haz el commit. Esa publicación fallará a propósito,
   porque el archivo y la lista tendrían la misma URL. El sitio sigue como
   estaba.
2. Borra el archivo `.html` de la raíz (abre el archivo → menú **⋯** →
   **Delete file** → **Commit changes**). Ahora sí se publica, y la página
   sale de la lista con la misma URL.

También detiene la publicación un `.html` en la raíz llamado `lista.html`,
`tablero.html` o `404.html`, porque esas páginas las genera la construcción.

## ¿Se publicó?

Abre la pestaña **Actions** del repositorio. Cada cambio en `main` lanza
**Publicar**:

- ✅ verde: el sitio ya está actualizado (GitHub Pages tarda hasta
  10 minutos en refrescar su caché).
- ❌ rojo: **no se publicó nada** y el sitio sigue como estaba. Entra a la
  ejecución: en el resumen aparecen los errores en español, con la línea de
  `agentes.json` donde está el problema (coma faltante, id repetido, key mal
  copiada, agente borrado…). Corrige el archivo y haz otro commit.

En un pull request, la misma revisión aparece abajo del PR, en los checks.

## Copiar el iframe

Abre **https://innovacion-ag.github.io/profesores/lista.html**: tiene todos los
agentes con botones **Copiar URL** y **Copiar iframe**. El iframe copiado ya
tiene el formato que va en Rise:

```html
<iframe src="https://innovacion-ag.github.io/profesores/<id>.html"
  width="100%" height="350" frameborder="0"
  allow="microphone; camera; autoplay" allowfullscreen></iframe>
```

**https://innovacion-ag.github.io/profesores/tablero.html** resume los agentes por
personaje y materia, con la fecha de la última publicación. Ninguna de las
dos páginas lleva las keys de D-ID.

## Qué hace la compuerta

Cada página carga el agente de video de D-ID, traducido al español. El
alumno pulsa **Iniciar conversación** (no hay inicio automático: en
celulares el micrófono solo se concede con un toque real, y abrir sesión en
cada visita agota las sesiones del plan).

Si D-ID no opera, la página cambia sola:

| Qué pasa en D-ID | Con `elevenlabs_agent_id` | Sin `elevenlabs_agent_id` |
|---|---|---|
| Sin créditos de video (pasa a solo-texto) | Cambia a ElevenLabs | Se queda el chat de texto de D-ID |
| No carga, se queda en «Cargando…», no conecta, falla la conexión, o la sala de espera / «Alta demanda» dura más de 20 s | Cambia a ElevenLabs | Aviso «El profesor no está disponible por el momento» con botón **Intentar de nuevo** |

Al cambiar, primero se cuelga D-ID y después aparece ElevenLabs o el aviso,
para que no queden dos sesiones abiertas. Si D-ID todavía estaba
conectando, se le apaga el micrófono y cuelga solo unos segundos después. Si el alumno acaba de tocar la
página, la conversación de ElevenLabs arranca sola; si no, la inicia él. La
sesión (D-ID o ElevenLabs) también se cierra al salir de la página y con el
botón **✕ Terminar**, para liberar las sesiones del plan. Detalles y qué no
se puede garantizar: [operación › Cierre de sesiones](docs/operacion.md#cierre-de-sesiones).

Para revisar algo, abre la consola del navegador (Cmd+Option+J en Chrome):
los mensajes de la página empiezan con `[agente]`. `__textosPendientes()`
lista lo que quedó en inglés.

## Borrar un agente a propósito

La publicación se detiene si un agente que estaba en la versión anterior o
en el sitio publicado desaparece de `agentes.json`, porque puede haber
iframes apuntando a él. Si de verdad quieres borrarlo:

1. Quítalo de `agentes.json` y haz el commit (la publicación fallará).
2. Ve a **Actions → Publicar → Run workflow**, marca **permitir borrar** y
   pulsa **Run workflow**.

Las 42 páginas que existían al migrar están además en
`scripts/urls_publicadas.txt`. Para borrar una de esas, también hay que
quitar su línea de ese archivo.

## Revertir un cambio

- **Si el cambio entró por un pull request:** abre el PR ya fusionado y pulsa
  **Revert**. GitHub crea un PR que deshace el cambio; fusiónalo y se
  publica.
- **Si hiciste el commit directo en `main`:** GitHub no tiene botón Revert
  para un commit suelto. Abre el archivo → **History** → abre la versión
  anterior → copia su contenido (botón **Copy raw file**) → edita el archivo
  actual, pega y **Commit changes**.

## Para quien tenga el repositorio clonado

Ver [Arranque](#arranque).

### Nota: si algún día se sirve desde Railway con Caddy

GitHub Pages no envía `Content-Security-Policy`. Si el sitio se mueve a
Railway con Caddy, el Caddyfile **no** debe agregar una CSP (o debe permitir
`blob:` en `media-src`), porque D-ID usa URLs `blob:` para el video y el
audio.

## Migración

**Completada.** El 2026-11-18 el repositorio se transfirió de la cuenta
`ErikV-AG` a la organización `Innovacion-AG`, y el sitio pasó de
`erikv-ag.github.io/profesores` a `innovacion-ag.github.io/profesores`. Las
direcciones anteriores siguen funcionando por el puente `ErikV-AG/profesores`
(ver [docs/decisiones/0005](docs/decisiones/0005-se-transfiere-a-innovacion-ag-y-se-conserva-la-direccion-anterior-con-un-puente.md)).

La migración anterior, de páginas copiadas a mano a una lista y una plantilla
publicadas con GitHub Actions, se completó el 2026-10-08 (ver
[docs/decisiones/0003](docs/decisiones/0003-se-generan-todas-las-paginas-desde-una-lista-y-una-plantilla-compartida.md)).
