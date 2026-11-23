# 0007. Se cuelga D-ID antes del respaldo y al salir de la página, sin campo «tipo» en agentes.json

Estado: aceptado
Fecha: 2026-11-21
Origen: análisis de sesiones de D-ID abiertas o duplicadas (2026-11-21) y el PR que aplica este cambio

## Contexto
En otro proyecto con agentes de D-ID, las sesiones que quedan abiertas o
que se enciman producían «Alta demanda» y gasto de créditos. El análisis
de este repositorio, con D-ID y ElevenLabs simulados en Chrome, encontró:

- Al cambiar a ElevenLabs, la sesión de D-ID seguía abierta con el
  micrófono activo y se enciman dos sesiones. `cerrarDID()` solo apagaba
  el video que llega de D-ID.
- Una conexión de D-ID que terminaba después del cambio se completaba en
  el contenedor oculto.
- Con el aviso «no disponible», D-ID podía seguir conectando detrás.
- Al quitar Rise el iframe solo llega `pagehide`: `beforeunload` no llega
  y `unload` está bloqueado en Chrome. No salía ningún cierre explícito.

Del SDK público `@d-id/client-sdk` 3.0.9 y del embed v2 (2.2.12, que trae
ese SDK):

- Los agentes anteriores a V4 (`talk`, `clip`) usan streams V1 con WebRTC.
  Al colgar envían `DELETE /streams/{id}`, solo si ya estaban conectados,
  con un `fetch` sin `keepalive`.
- Los agentes de tipo `image` y `expressive` (V4) usan LiveKit. Al colgar
  salen de la sala; no envían DELETE.
- El embed no expone `disconnect()`, ni el estado, ni los ids de sesión.
  Cuelga por su cuenta con su botón «End call», en su `beforeunload` y
  cuando se desmonta. Se desmonta si su contenedor deja de estar en la
  página; lo revisa cada 2 s.

Inventario consultado en D-ID el 2026-11-21 (`GET /agents/{id}/runtime`):
de 54 entradas, 50 son `talk` y 2 son `image`. Las otras 2 respondieron
401. Ninguna es `expressive`.

## Decisión
- Antes de montar ElevenLabs o el aviso, `colgarDID()`:
  1. pulsa «End call» si está a la vista;
  2. quita el contenedor de D-ID para que el embed se desmonte y cuelgue;
  3. espera hasta 3 s;
  4. si sigue abierto, apaga el micrófono de D-ID y deja que cuelgue
     solo, sin cortarle la conexión (ver abajo).
- Con la página abierta, a D-ID nunca se le cortan las conexiones:
  - Si se le pide colgar mientras conecta (desmontaje o «End call»), el
    SDK termina de conectar y cuelga solo.
  - Si se le corta la conexión, la da por fallida y reintenta con una
    sesión nueva: hasta 3 intentos de 45 s. En la prueba real, un corte
    produjo una segunda sesión a los 46 s.
  - Si llega a conectar después del cambio y a los 5 s sigue conectado, se
    le vuelve a pedir colgar con «End call».
  - El cierre forzado de conexiones queda solo para `pagehide`.
- Un registro en `src/plantilla.html` anota cada `RTCPeerConnection`,
  `WebSocket` y pista de `getUserMedia`:
  - Solo registra: la llamada original no cambia, y si registrar falla se
    ignora.
  - Una conexión es de D-ID si va a `*.d-id.com` o a LiveKit Cloud
    (`*.livekit.cloud`, `*.livekit.run`), o si la creó el código del embed
    (`agent.d-id.com`). Lo que va a `*.elevenlabs.io` nunca se toca.
  - El destino solo no basta (prueba real del 2026-11-22): el video de los
    agentes talk va por el TURN de Cloudflare, compartido con otros
    servicios; la conexión de LiveKit se crea sin servidores ICE; y el
    dictado va a `*.stt.speech.microsoft.com`.
  - Una pista de micrófono es de D-ID si la pidió el código de D-ID o si
    se pidió antes del cambio.
- Después del cambio, a una conexión de D-ID que llega tarde se le apaga
  el micrófono y se espera a que D-ID cuelgue.
- En `pagehide` se pulsa «End call», se cierran las conexiones y el
  micrófono de D-ID y se cierra ElevenLabs. Todo es síncrono. No se cuelga
  en `visibilitychange`, para no cortar a quien solo cambia de pestaña.
- ElevenLabs arranca solo únicamente con un gesto reciente
  (`navigator.userActivation.isActive`); si no, lo inicia el alumno.
- No se agrega un campo «tipo» a `agentes.json`. El cierre es el mismo
  para los dos tipos. Un dato escrito a mano dejaría de ser cierto sin
  aviso cuando alguien cambie el agente en D-ID Studio. Si hiciera falta
  el tipo, se puede leer en tiempo de ejecución.

## Consecuencias
- El cambio a ElevenLabs tarda hasta 3 s más mientras D-ID cuelga; el
  cargador muestra «Cambiando a modo de voz…».
- Si D-ID seguía conectando al cambiar, queda abierto, sin micrófono,
  hasta que termina de conectar y cuelga: de 9 a 12 s en la prueba real.
  Es preferible a cortarlo, porque cortarlo abría otra sesión.
- El alumno que espera la sala de espera o el tiempo límite tendrá que
  pulsar «Iniciar conversación» en ElevenLabs: su toque ya no es reciente.
- Al quitar Rise el iframe no se puede garantizar el DELETE de los
  agentes anteriores a V4: el `fetch` del SDK puede morir con la página.
  El navegador sí corta las conexiones, y D-ID libera la sesión según su
  propio tiempo de expiración.
- Si D-ID cambia sus dominios o el embed deja de desmontarse al quitar el
  contenedor, hay que revisar `DOMINIOS_DID` y `colgarDID()`. Los logs
  `[agente] conexión abierta: …` muestran los hosts reales.
- El DELETE que envía el propio embed al pulsar «End call» responde 400
  («missing or invalid session_id») en los agentes talk (embed 2.2.12):
  aun con la página abierta, D-ID libera esas sesiones por expiración. Es
  un error del embed que no se corrige desde la página.

## Evidencia
- Banco de pruebas fuera del repositorio: Chrome 158 sin interfaz, D-ID y
  ElevenLabs simulados, página padre como Rise y el puente
  `ErikV-AG/profesores` con su código real. Los resultados están en la
  descripción del PR.
- Conexión tardía con el ajuste (2026-11-23, 4 sesiones autorizadas, 2 por
  tipo, 75 s de observación cada una). D-ID colgó solo en las 4 y no
  hubo reintentos; ElevenLabs siguió hablando:
  - talk: conectó 8.8 y 9.3 s después del cambio, colgó 2.6 s después de
    conectar y quedó abierto 11.5 y 12.0 s en total. Su DELETE respondió
    400.
  - image (LiveKit): conectó 2.8 y 2.6 s después del cambio y quedó
    abierto 9.0 y 10.3 s en total. A los 5 s de conectar seguía conectado,
    pero la segunda petición no encontró «End call»: el embed ya
    desmontado no deja botones. Colgó solo 0.8 y 2.5 s después.
- Prueba con D-ID real en `prueba/iframes.html` (2026-11-22, 10 sesiones
  autorizadas): hosts reales, conexión normal y «✕ Terminar», cambio a
  ElevenLabs con D-ID conectando y desmontaje en 1 y 2 niveles. Resultados
  en el PR.
- `@d-id/client-sdk` 3.0.9: `webrtc-manager.ts` (`disconnect`) y
  `livekit-manager.ts` (`disconnect`).
- Embed `agent.d-id.com/2.2.12`: `DID_AGENTS_API`, el `beforeunload` y el
  desmontaje al quitar el contenedor.
