# 0005. Se transfiere a Innovacion-AG y se conserva la dirección anterior con un puente

Estado: aceptado
Fecha: 2026-11-18
Origen: decidido con el dueño del repositorio durante la migración

## Contexto
El repositorio vivía en la cuenta personal `ErikV-AG` y el sitio en
`erikv-ag.github.io/profesores`. Los proyectos del área deben vivir en la
organización Innovacion-AG. Hay iframes en cursos de Rise ya publicados que
apuntan a la dirección anterior con `allow="microphone; camera; autoplay"`.
Ese permiso solo vale para el origen del `src`: una redirección a otro
dominio dejaría al agente sin micrófono.

## Decisión
Se transfiere el repositorio a `Innovacion-AG/profesores` y se publica en
`innovacion-ag.github.io/profesores`. En la cuenta personal se crea el
repositorio `ErikV-AG/profesores` con solo `index.html` y `404.html`: dentro
de un iframe incrustan la página nueva y le delegan micrófono, cámara y
autoplay; abiertos directamente, redirigen. Ambos conservan ruta, parámetros
y ancla. Las client keys de D-ID y los agentes de ElevenLabs permiten los dos
dominios.

## Consecuencias
- Los cursos existentes siguen funcionando sin editar sus iframes, con
  micrófono.
- Los cursos nuevos usan la dirección nueva (`lista.html` ya la entrega).
- El puente queda en una cuenta personal y no debe borrarse mientras haya
  cursos con la dirección anterior.
- Las URL anteriores responden con estado HTTP 404 y el contenido del
  puente, lo normal para un `404.html` de GitHub Pages.

## Evidencia
- 7ccff0f Migra las direcciones a innovacion-ag.github.io (#5)
- Repositorio `ErikV-AG/profesores`, commit d300574
- Prueba en Chrome sin interfaz con tres URL antiguas dentro de un iframe como el de Rise: micrófono y autoplay permitidos en la página nueva
