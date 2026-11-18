# 0002. Se usa ElevenLabs como respaldo de voz con compuerta automática

Estado: propuesto
Fecha: 2026-09-09
Origen: reconstruido desde el historial de git

## Contexto
D-ID deja de operar cuando se agotan los créditos, cuando no hay sesiones
libres («Alta demanda») o cuando el embed no carga. En esos casos el alumno
se quedaba sin profesor.

## Decisión
Se usa un agente de voz de ElevenLabs como respaldo. La página vigila el
estado de D-ID y, si no opera, cierra la sesión de D-ID y carga el widget de
ElevenLabs (`@elevenlabs/convai-widget-embed`) con el `elevenlabs_agent_id`
del agente. Sin ese id, se muestra el chat de texto de D-ID o un aviso.

## Consecuencias
- El alumno conserva una conversación por voz cuando D-ID falla.
- Cada agente requiere un segundo agente en ElevenLabs; hoy 40 de 53 lo
  tienen.
- La detección depende de señales de la interfaz de D-ID (overlay, sala de
  espera, eventos de conexión) que pueden cambiar sin aviso.
- El widget se carga desde unpkg.com sin versión fija.

## Evidencia
- f9c0b82 Enhance D-ID agent with ElevenLabs integration (`tecnologia_transformacion_alan.html`)
- 3b6bee6 Refactor ElevenLabs widget and improve translations
- dec39ff Plantilla, lógica y estilo compartidos por todos los agentes (la compuerta pasa a `src/agente.js`)
- README.md, sección «Qué hace la compuerta»
