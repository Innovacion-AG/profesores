# Operación: Profesores

Sitio estático en GitHub Pages: https://innovacion-ag.github.io/profesores/.
Los cursos anteriores llegan por el puente https://erikv-ag.github.io/profesores/
(repositorio `ErikV-AG/profesores`).

## No tocar sin consultar

Consultar con el responsable antes de cambiar:

- Los ids (llaves) de `agentes.json`: cada uno es la URL de un iframe en un curso de Rise.
- `scripts/urls_publicadas.txt`.
- `src/`: la plantilla, la lógica y el estilo afectan las 53 páginas a la vez.
- `benito.html` y `content/`.
- El repositorio puente `ErikV-AG/profesores`.
- Los dominios permitidos en D-ID y ElevenLabs.

## Despliegue

1. Según el tipo de cambio:
   - Altas y cambios de agentes en `agentes.json`: commit directo en `main`
     desde github.com, sin pull request (PR). El proceso completo de un
     alta, desde la Cédula hasta la entrega a DEA, está en
     [alta de un agente](alta-de-agente.md).
   - Cambios a `src/`, a `scripts/` o a la publicación (`.github/`): rama
     nueva y PR contra `main`. En el PR, el check «Validar y construir» corre
     `node --check src/agente.js` y `python3 scripts/construir.py`.
2. Si la validación falla, el resumen de la ejecución indica el error y la
   línea de `agentes.json`. No se publica nada.
3. Con el push a `main` (commit directo o PR integrado), `publicar.yml` valida otra vez, construye `_site/` y lo
   publica con `actions/deploy-pages`. La validación compara contra
   `HEAD~1` y contra `ids-publicados.json` del sitio en línea: un agente
   borrado detiene la publicación.
4. Verificar en Actions › Publicar que la ejecución termine en verde. GitHub
   Pages tarda hasta 10 minutos en refrescar su caché.
5. Abrir https://innovacion-ag.github.io/profesores/lista.html y probar el
   agente cambiado.
6. Si es un alta, pegar el código de inserción en la
   [Cédula de Seguimiento de materias](https://docs.google.com/spreadsheets/d/1I9YYzEdpFYIqBMzy-pgykguDBPYmFZ15uAWOYCB8ObA/edit#gid=764442467)
   de DEA, hoja «Personajes cod», columna «Código de insersión», en la fila
   de la materia:
   - Copiarlo de `lista.html` con el botón **Copiar iframe** del agente; no
     escribirlo a mano ni copiarlo de otra fila.
   - Comprobar que el `src` empiece con
     `https://innovacion-ag.github.io/profesores/`. Las filas anteriores a
     la migración tienen `erikv-ag.github.io`: siguen funcionando por el
     puente, pero no se usan para altas nuevas.
   - Con el código en la Cédula, DEA activa el agente en Rise.

Para borrar un agente a propósito: quitarlo de `agentes.json`, y luego
Actions › Publicar › Run workflow con la casilla «permitir borrar». Si está
en `scripts/urls_publicadas.txt`, quitar también su línea.

## Probar un cambio en el dominio real (`prueba/`)

D-ID solo conecta desde los dominios permitidos. Un cambio a `src/` se
prueba en https://innovacion-ag.github.io/profesores/prueba/ antes de
llegar a las páginas de los cursos:

1. En la rama del cambio, ejecutar `python3 scripts/construir.py --prueba`.
   Genera `prueba/` con los agentes de `AGENTES_PRUEBA` (en
   `scripts/construir.py`) y el `src/` de esa rama.
2. Llevar solo la carpeta `prueba/` a `main` con un PR aparte. Si `prueba/`
   existe, la construcción la publica tal cual en `/prueba/`, con un aviso
   en Actions. Las páginas de producción no cambian.
3. Probar en `prueba/iframes.html`, con los agentes en un iframe de 350 px
   como en Rise, y en cada página de agente.
4. Al integrar el cambio, borrar `prueba/` en el mismo PR.

## Reversión

- Cambio que entró por PR: abrir el PR integrado, pulsar **Revert** e
  integrar el PR que genera GitHub. Se publica solo.
- Commit directo en `main`: abrir el archivo › History › versión anterior ›
  copiar el contenido, pegarlo en el archivo actual y hacer commit.
- Verificación: Actions › Publicar en verde y la página afectada responde en
  https://innovacion-ag.github.io/profesores/<id>.html.
- Si una publicación falla, el sitio queda como estaba: no hace falta revertir
  para proteger lo publicado.

## Monitoreo

| Señal | Dónde se observa | Qué hacer |
| --- | --- | --- |
| Publicación en rojo | Actions › Publicar; GitHub puede avisar por correo a quien lanzó la ejecución, según su configuración de notificaciones | Leer el error en el resumen de la ejecución y corregir `agentes.json`. |
| Agentes en respaldo de ElevenLabs o con aviso «no disponible» | Consola del navegador, mensajes con prefijo `[agente]` | Revisar créditos y sesiones de D-ID en el tablero de la cuenta «Proyectos Innovación». |
| Cambios a ElevenLabs sin `D-ID colgó solo` después, o conexiones de D-ID sin la marca «(D-ID)» | Consola del navegador, mensajes `[agente]` (ver [Cierre de sesiones](#cierre-de-sesiones)) | Revisar si el embed de D-ID cambió y ajustar `DOMINIOS_DID` o `colgarDID()` en `src/agente.js` por PR. |
| «Alta demanda» o sala de espera frecuente | Reportes de DEA o de alumnos | Revisar las sesiones simultáneas del plan de D-ID. |
| Una URL antigua no carga | Abrir `https://erikv-ag.github.io/profesores/<id>.html` | Revisar que `ErikV-AG/profesores` exista y tenga GitHub Pages activo. |

PENDIENTE: ¿quién recibe los reportes de falla de DEA y por qué canal, además de `#proj-profesores`?

## Cierre de sesiones

Una sesión de D-ID abierta ocupa un lugar del plan. El siguiente alumno
cae en la sala de espera o ve «Alta demanda». La página cuelga D-ID en
estos casos (decisión 0007):

| Momento | Qué hace la página |
| --- | --- |
| D-ID no opera y se cambia a ElevenLabs o al aviso | Cuelga D-ID primero: pulsa «End call» si está a la vista, quita el contenedor de D-ID (el embed se desmonta y cuelga solo) y espera hasta 3 s. Si sigue abierto, apaga su micrófono y deja que cuelgue solo. Después monta ElevenLabs o el aviso. |
| D-ID termina de conectar después del cambio | No le corta la conexión: D-ID reintentaría con una sesión nueva. Apaga su micrófono y espera a que cuelgue solo; si a los 5 s de conectar sigue conectado, le vuelve a pedir colgar. No toca ElevenLabs. |
| «✕ Terminar» sin «End call» a la vista, o «Intentar de nuevo» | Cuelga D-ID y, ya colgado, recarga. |
| El alumno sale (cambia de lección, cierra la pestaña, recarga, va atrás) | En `pagehide`: pulsa «End call», cierra lo de D-ID y cierra ElevenLabs. |

Una conexión cuenta como de D-ID si va a `*.d-id.com` o a LiveKit Cloud
(`*.livekit.cloud`, `*.livekit.run`), o si la creó el código del embed
(`agent.d-id.com`). Lo segundo cubre el video de los agentes talk, que va
por el TURN de Cloudflare; la conexión de LiveKit, que se crea sin
servidores, y el dictado (`*.stt.speech.microsoft.com`). Lo que va a
`*.elevenlabs.io` nunca se cierra como D-ID.

ElevenLabs arranca solo únicamente si el alumno acaba de tocar la página;
si no, queda listo y el alumno pulsa «Iniciar conversación».

### Qué buscar en la consola (mensajes `[agente]`)

| Mensaje | Qué significa |
| --- | --- |
| `conexión abierta: ws … (D-ID)` / `pc … (D-ID)` | Conexión registrada y reconocida como de D-ID. Sin «(D-ID)», la página no la cerrará. Si una conexión de D-ID aparece sin la marca, hay que agregar su dominio a `DOMINIOS_DID` en `src/agente.js`. |
| `colgando D-ID (motivo); conexión: …` | Empieza el cierre previo al respaldo. |
| `se pulsó «End call» de D-ID` | Se usó el cierre propio de D-ID: los agentes anteriores a V4 envían el DELETE (que hoy responde 400; ver abajo). |
| `D-ID colgado en N ms` | D-ID cerró todo dentro del tope. Es lo esperado. |
| `D-ID sigue abierto a los 3 s (…); se deja que cuelgue solo` | D-ID seguía conectando al cambiar. Es normal: le siguen los mensajes de abajo. |
| `conexión tardía de D-ID: …; se deja que D-ID cuelgue` | D-ID abrió una conexión después del cambio. |
| `D-ID conectó después del cambio (N s)` | D-ID terminó de conectar; debe colgar solo en unos segundos. |
| `D-ID sigue conectado 5 s después de conectar; se le vuelve a pedir colgar` | Segunda petición. Con el embed ya desmontado no suele haber «End call» que pulsar (`no hay «End call» de D-ID que pulsar`). |
| `D-ID colgó solo; quedó abierto N s después del cambio` | D-ID cerró todo. En la prueba real fueron de 9 a 12 s. Si no aparece, D-ID quedó abierto hasta que el alumno salga de la página: revisar si el embed cambió. |
| `conexión tardía de D-ID cerrada: …` | Después de salir de la página (`pagehide`), D-ID abrió algo y se cerró. |
| `salida de la página (pagehide)` | Cierre al salir. Le siguen el cierre forzado (el único momento en que la página corta conexiones de D-ID) y `ElevenLabs cerrado (salida de la página)` si había conversación. |

Los logs muestran solo el host de cada conexión, nunca la URL: la de D-ID
lleva la client key.

### Qué no se puede garantizar

- **Al quitar Rise el iframe** (cambio de lección), la página tiene solo el
  evento `pagehide` y no puede esperar:
  - La página pulsa «End call», pero el DELETE que envían los agentes
    anteriores a V4 sale con un `fetch` del SDK de D-ID que muere con la
    página: en las pruebas simuladas no llegó en ninguno de 4 desmontajes,
    y con D-ID real el navegador lo canceló. La salida de la sala de los
    agentes de LiveKit va por su WebSocket: llegó en 3 de 4 desmontajes
    simulados y con D-ID real no se pudo observar.
  - El navegador sí corta las conexiones de inmediato. D-ID libera la
    sesión según su propio tiempo de expiración, que no controlamos.
  - Lo mismo aplica con el puente de las direcciones anteriores: es un
    nivel más de iframe y se comporta igual.
- **El DELETE de D-ID falla** en los agentes talk (embed 2.2.12): aun con
  «End call» y la página abierta, D-ID responde 400 «missing or invalid
  session_id» (prueba real del 2026-11-22, 2 de 2). Esas sesiones se
  liberan por expiración de D-ID. Es un error del embed; hay que
  reportarlo a D-ID.
- **Tras el cambio a ElevenLabs**, D-ID puede quedar abierto unos segundos
  mientras termina de conectar y cuelga (de 9 a 12 s en la prueba real
  del 2026-11-23, sin reintentos). Si no colgara, sigue abierto hasta que
  el alumno sale de la página: el embed no ofrece otro camino para
  colgar.
- **Si D-ID cambia** sus dominios o la forma en que el embed se desmonta,
  el cierre forzado puede dejar de reconocer sus conexiones. Los logs de
  arriba lo muestran.
- **Rise**: lo que hace al cambiar de lección no se puede modificar.

## Tareas periódicas

| Tarea | Frecuencia | Detalle |
| --- | --- | --- |
| Revisar créditos de D-ID | Antes del día 18 de cada mes | Los créditos de la cuenta «Proyectos Innovación» se renuevan el día 18. |
| Revisar plan y saldo de ElevenLabs | PENDIENTE: ¿fecha de renovación? | Cuenta del área. |
| Confirmar la cuenta de HeyGen | Una vez | Los videos de presentación deben producirse con una cuenta del área. |
| Confirmar dominios permitidos | Al crear o rotar una client key de D-ID y al crear un agente de ElevenLabs | Deben incluir `innovacion-ag.github.io` y `erikv-ag.github.io`. |
| Actualizar acciones de GitHub | Antes del retiro de Node.js 20 | Incidencia #6. |
| Revisar la documentación | Trimestral | Actualizar `academiaglobal.mx/verificado-el` en `catalog-info.yaml`. |

No hay respaldos que programar: todo el contenido está en git.

## Contactos y escalamiento

| Rol | Contacto |
| --- | --- |
| Responsable | @CoriaAlx |
| Suplente | @rodrig317-cpu |
| Supervisor y revisor | @ErikV-AG |
| Patrocinio | DEA |
| Área usuaria | DEA: asigna los personajes, monta los agentes en Rise y da seguimiento en la Cédula de Seguimiento de materias, hoja «Personajes cod» (fuente del plan; ver el blueprint, sección 4) |
| Proveedor de video | D-ID. PENDIENTE: ¿canal de soporte y titular de la cuenta? |
| Proveedor de voz | ElevenLabs. PENDIENTE: ¿canal de soporte y titular de la cuenta? |
| Canal | `#proj-profesores` en Slack |
