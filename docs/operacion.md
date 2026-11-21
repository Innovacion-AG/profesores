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
     desde github.com, sin pull request (PR).
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
| `cierre forzado de D-ID (tope …)` frecuente, o conexiones de D-ID sin la marca «(D-ID)» | Consola del navegador, mensajes `[agente]` (ver [Cierre de sesiones](#cierre-de-sesiones)) | Revisar si el embed de D-ID cambió y ajustar `DOMINIOS_DID` o `colgarDID()` en `src/agente.js` por PR. |
| «Alta demanda» o sala de espera frecuente | Reportes de DEA o de alumnos | Revisar las sesiones simultáneas del plan de D-ID. |
| Una URL antigua no carga | Abrir `https://erikv-ag.github.io/profesores/<id>.html` | Revisar que `ErikV-AG/profesores` exista y tenga GitHub Pages activo. |

PENDIENTE: ¿quién recibe los reportes de falla de DEA y por qué canal, además de `#proj-profesores`?

## Cierre de sesiones

Una sesión de D-ID abierta ocupa un lugar del plan. El siguiente alumno
cae en la sala de espera o ve «Alta demanda». La página cuelga D-ID en
estos casos (decisión 0007):

| Momento | Qué hace la página |
| --- | --- |
| D-ID no opera y se cambia a ElevenLabs o al aviso | Cuelga D-ID primero: pulsa «End call» si está a la vista, quita el contenedor de D-ID (el embed se desmonta y cuelga solo) y espera hasta 3 s. Lo que siga abierto lo cierra a la fuerza. Después monta ElevenLabs o el aviso. |
| D-ID conecta después del cambio | Cierra esa conexión en cuanto aparece, sin tocar las de ElevenLabs. |
| «✕ Terminar» sin «End call» a la vista, o «Intentar de nuevo» | Cuelga D-ID y, ya colgado, recarga. |
| El alumno sale (cambia de lección, cierra la pestaña, recarga, va atrás) | En `pagehide`: pulsa «End call», cierra lo de D-ID y cierra ElevenLabs. |

Una conexión cuenta como de D-ID por su destino: `*.d-id.com` o LiveKit
Cloud (`*.livekit.cloud`, `*.livekit.run`). Lo que va a `*.elevenlabs.io`
nunca se cierra como D-ID.

ElevenLabs arranca solo únicamente si el alumno acaba de tocar la página;
si no, queda listo y el alumno pulsa «Iniciar conversación».

### Qué buscar en la consola (mensajes `[agente]`)

| Mensaje | Qué significa |
| --- | --- |
| `conexión abierta: ws … (D-ID)` / `pc … (D-ID)` | Conexión registrada y reconocida como de D-ID. Sin «(D-ID)», la página no la cerrará. Si D-ID usa un dominio nuevo, aparece aquí sin la marca: hay que agregarlo a `DOMINIOS_DID` en `src/agente.js`. |
| `colgando D-ID (motivo); conexión: …` | Empieza el cierre previo al respaldo. |
| `se pulsó «End call» de D-ID` | Se usó el cierre propio de D-ID: los agentes anteriores a V4 envían el DELETE. |
| `D-ID colgado en N ms` | D-ID cerró todo dentro del tope. Es lo esperado. |
| `cierre forzado de D-ID (tope de 3 s): …` | D-ID no cerró a tiempo y la página cerró lo que quedaba. Si se repite, revisar si el embed cambió. |
| `conexión tardía de D-ID cerrada: …` o `D-ID avisó «connected» después de colgar` | D-ID conectó después del cambio y se cerró. |
| `salida de la página (pagehide)` | Cierre al salir. Le siguen el cierre forzado y `ElevenLabs cerrado (salida de la página)` si había conversación. |

Los logs muestran solo el host de cada conexión, nunca la URL: la de D-ID
lleva la client key.

### Qué no se puede garantizar

- **Al quitar Rise el iframe** (cambio de lección), la página tiene solo el
  evento `pagehide` y no puede esperar:
  - La página pulsa «End call», pero el DELETE que envían los agentes
    anteriores a V4 sale con un `fetch` del SDK de D-ID que muere con la
    página: en las pruebas simuladas no llegó en ninguno de 4 desmontajes.
    La salida de la sala de los agentes de LiveKit va por su WebSocket y
    llegó en 3 de 4.
  - El navegador sí corta las conexiones de inmediato. D-ID libera la
    sesión según su propio tiempo de expiración, que no controlamos.
  - Lo mismo aplica con el puente de las direcciones anteriores: es un
    nivel más de iframe y se comporta igual.
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
