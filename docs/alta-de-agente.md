# Alta de un agente en profesores

Dar de alta un agente de `profesores` toma nueve pasos: siete de producción en Innovación y dos de entrega a DEA. El resultado es una página publicada en `https://innovacion-ag.github.io/profesores/<id>.html`, cuyo iframe queda en la Cédula de DEA para que lo active en Rise. Este manual está escrito para alguien que nunca lo ha hecho.

## Resumen del flujo

| Paso | Qué se hace | Dónde | Resultado |
| --- | --- | --- | --- |
| 1 | Revisar la materia, el personaje y las fechas, y definir el id del agente | Cédula de DEA | Id del agente y fecha límite |
| 2 | Preparar el system prompt y la base de conocimiento | Artefacto para agentes en Claude | Dos documentos en Drive |
| 3 | Producir el video de presentación | HeyGen | Video con subtítulos en Drive |
| 4 | Crear el agente de video | D-ID, cuenta Proyectos Innovación | Id del agente y client key |
| 5 | Crear el agente de voz de respaldo | ElevenLabs, cuenta del área | Id del agente de respaldo |
| 6 | Dar de alta el agente | `agentes.json` en GitHub | Commit en `main` |
| 7 | Validar y publicar | GitHub Actions | Página publicada |
| 8 | Probar el agente | `lista.html` | Agente verificado |
| 9 | Entregar a DEA | Cédula de DEA y `#proj-profesores` | Código de inserción listo para Rise |

## Antes de empezar

Sin estos accesos no se puede completar el alta. Se revisan una sola vez, la primera vez que se da de alta un agente.

- [ ] Ser miembro de la organización de GitHub `Innovacion-AG` y del equipo `equipo-innovacion`, que da permiso de escritura en este repositorio.
- [ ] Tener acceso al gestor de contraseñas del área, grupo `profesores`, donde están las credenciales de D-ID y ElevenLabs y la client key de incrustación.
- [ ] Tener acceso al perfil de equipo de HeyGen con la plantilla de video de presentación. El acceso se solicita al supervisor del proyecto.
- [ ] Tener acceso a la cuenta de Claude del área, donde está el [artefacto para agentes](https://claude.ai/code/artifact/018e5899-8bd1-4778-9f6c-e4afbd5f5545).
- [ ] Tener permiso de edición en la [Cédula de Seguimiento de materias](https://docs.google.com/spreadsheets/d/1I9YYzEdpFYIqBMzy-pgykguDBPYmFZ15uAWOYCB8ObA/edit#gid=764442467) de DEA.
- [ ] Tener acceso a las carpetas de Drive del proyecto: [videos de presentación](https://drive.google.com/drive/folders/1P_y6_Q8H01367gEzjXniYDfvIN__UT78), [system prompts y bases de conocimiento](https://drive.google.com/drive/folders/1F1HAEUCagzCfwgGhH5OfAU6mMJUN-Sm0) y [video tutorial](https://drive.google.com/drive/folders/1Xv-4605UEGnhqFBJZWG7xOlY7D6_DbS7).
- [ ] Estar en el canal `#proj-profesores` de Slack.

Las credenciales nunca se copian al repositorio, a Slack, a WhatsApp ni a Notion. Siempre se consultan en el gestor de contraseñas del área, grupo `profesores`.

## Paso 1. Revisar la asignación en la Cédula

La Cédula de DEA es la fuente del plan: dice qué personaje va en cada materia y para cuándo se necesita. Nada se produce sin una fila asignada en ella (ver la [decisión 0006](decisiones/0006-la-cedula-de-dea-es-la-fuente-del-plan-de-personajes-y-agentes-json-la-de-lo-publicado.md)).

1. Abrir la [Cédula](https://docs.google.com/spreadsheets/d/1I9YYzEdpFYIqBMzy-pgykguDBPYmFZ15uAWOYCB8ObA/edit#gid=764442467), hoja «Personajes cod», y ubicar la fila de la materia.
2. Anotar los datos de la fila:
   1. MATERIA y PROGRAMA (INI, LAE, LEM o NBACH).
   2. PERSONAJE asignado.
   3. FECHA ENTREGA COMPROMISO, que es la fecha límite para entregar el agente.
   4. ACTIVACIÓN (INICIO EN CAMPUS), que es cuándo empieza la materia. El agente debe estar publicado antes.
3. Revisar en la hoja «concentrado de personajes» si el personaje ya tiene agentes en otras materias. Si los tiene, preguntar en `#proj-profesores` si se reutiliza su video de presentación y su configuración de voz.
4. Definir el id del agente. Es el nombre de su página y no se puede cambiar después de publicarlo, porque los iframes de Rise apuntan a él.
   1. Formato: materia abreviada, guión bajo y apellido o nombre corto del personaje. Ejemplos existentes: `calculo_alan`, `fisica_curie`, `publicidad_ogilvy`.
   2. Solo letras sin acento, números, `_` y `-`; todo en minúsculas.
   3. Comprobar en [`agentes.json`](../agentes.json) que el id no exista.
5. Avisar en `#proj-profesores` que se inicia el agente, con la materia, el personaje, el id y la fecha compromiso.

## Paso 2. Preparar el system prompt y la base de conocimiento

El system prompt define cómo habla y qué sabe el personaje; la base de conocimiento le da el contenido de la materia. Los dos se generan con el artefacto para agentes y se usan igual en D-ID y en ElevenLabs.

1. Ubicar el contenido en crudo de la materia. Está indicado en la hoja «Cont. en crudo» de la Cédula, columna «Liga docs» (archivos `_Curso` y `_Examen`).
2. Abrir el [artefacto para agentes](https://claude.ai/code/artifact/018e5899-8bd1-4778-9f6c-e4afbd5f5545) con la cuenta de Claude del área.
3. Cargar el contenido en crudo e indicar el personaje y la materia. El artefacto genera el system prompt y la base de conocimiento.
4. Revisar el resultado antes de usarlo:
   1. El personaje habla en español y en primera persona.
   2. Se mantiene dentro de la materia y redirige las preguntas ajenas a ella.
   3. Los temas y módulos coinciden con el curso de Rise.
   4. No hay datos inventados sobre el personaje ni sobre la materia.
5. Guardar los dos documentos en la carpeta [system prompts y bases de conocimiento](https://drive.google.com/drive/folders/1F1HAEUCagzCfwgGhH5OfAU6mMJUN-Sm0), nombrados con el id del agente: `<id> - system prompt` y `<id> - base de conocimiento`.

Si el artefacto falla o el resultado no pasa la revisión, avisar en `#proj-profesores` antes de seguir. Un error en este paso se repite en los dos proveedores.

## Paso 3. Producir el video de presentación en HeyGen

El video presenta al personaje al inicio de la materia y se hace antes que el agente, como se acordó en septiembre de 2026. Si el personaje ya tiene un video que se reutiliza (paso 1), este paso se omite.

1. Entrar a HeyGen con el perfil de equipo y abrir la plantilla de video de presentación. El [video tutorial](https://drive.google.com/drive/folders/1Xv-4605UEGnhqFBJZWG7xOlY7D6_DbS7) muestra el proceso completo.
2. Elegir el avatar del personaje y escribir el guion de presentación: quién es, por qué acompaña esta materia y cómo puede ayudar al alumno.
3. Generar el video con subtítulos y descargarlo en 1080p.
4. Nombrar el archivo con la materia y el sufijo de subtítulos, como los existentes; por ejemplo, `quimica_II_1080p_caption.mp4`.
5. Subirlo a la carpeta [videos de presentación](https://drive.google.com/drive/folders/1P_y6_Q8H01367gEzjXniYDfvIN__UT78).
6. Registrar el nombre del archivo en la Cédula, columna «Video del personaje», en la fila de la materia.

PENDIENTE: confirmar con el responsable de dónde sale el guion de presentación y quién lo aprueba.

## Paso 4. Crear el agente en D-ID

D-ID es el agente de video principal. Todos los agentes de `profesores` viven en la cuenta «Proyectos Innovación»; uno creado en otra cuenta tendría que migrarse después y cambiar su id en `agentes.json`.

1. Revisar que la cuenta tenga créditos. Se renuevan el día 18 de cada mes; si están por agotarse, avisar en `#proj-profesores`.
2. Entrar a [D-ID Studio](https://studio.d-id.com) con las credenciales de la cuenta, tomadas del gestor de contraseñas del área, grupo `profesores`.
3. Crear el agente:
   1. Nombre: el personaje y la materia, por ejemplo «Alan Turing · Cálculo I».
   2. Avatar y voz del personaje. Si ya tiene otros agentes, usar la misma voz.
   3. Idioma: español.
   4. Instrucciones: pegar el system prompt del paso 2.
   5. Base de conocimiento: subir el documento del paso 2.
4. Abrir Agents → Embed y revisar los dominios permitidos. Deben estar `innovacion-ag.github.io` y `erikv-ag.github.io`. Sin ellos, la página no conecta.
5. Copiar del código de Embed los dos valores que pide `agentes.json`:
   1. `data-agent-id`, que empieza con `v2_agt_`.
   2. `data-client-key`, que empieza con `ck_`. Debe coincidir con la client key de incrustación guardada en el gestor de contraseñas del área, grupo `profesores`; si es distinta, avisar en `#proj-profesores` antes de seguir.
6. Probar el agente en la vista previa de D-ID: que salude en español y responda una pregunta de la materia.

## Paso 5. Crear el agente de respaldo en ElevenLabs

Si D-ID no opera (sin créditos, sin sesiones libres o sin cargar), la página cambia sola a un agente de voz de ElevenLabs. Sin respaldo, el alumno solo ve un aviso de «no disponible» o el chat de texto, así que todo agente nuevo debe tenerlo.

1. Entrar a [ElevenLabs](https://elevenlabs.io) con las credenciales de la cuenta del área, tomadas del gestor de contraseñas del área, grupo `profesores`.
2. Crear un agente conversacional nuevo con el mismo nombre que en D-ID.
3. Configurarlo con lo mismo del paso 2:
   1. Idioma: español.
   2. Instrucciones: el mismo system prompt.
   3. Base de conocimiento: el mismo documento.
   4. Voz: la más parecida a la del agente de D-ID.
4. Si el agente tiene una lista de dominios permitidos, agregar `innovacion-ag.github.io` y `erikv-ag.github.io`.
5. Copiar el id del agente, que empieza con `agent_`.
6. Probarlo en la vista previa de ElevenLabs con una pregunta de la materia.

Si por alguna razón no se puede crear el respaldo, el alta sigue con el campo vacío y se avisa en `#proj-profesores` para completarlo después.

## Paso 6. Dar de alta el agente en agentes.json

Todas las páginas se generan desde una sola lista, `agentes.json`. El alta se hace desde el navegador, sin clonar nada, y va directo a `main`, sin pull request.

1. Abrir [`agentes.json`](../agentes.json) y pulsar el lápiz (**Edit this file**).
2. Copiar un bloque existente, pegarlo antes del último cierre de llave y cambiar sus valores:

   ```json
   "calculo_alan": {
     "nombre": "Alan Turing · Cálculo I",
     "did_agent_id": "v2_agt_VRr-daUT",
     "did_client_key": "ck_…",
     "elevenlabs_agent_id": "agent_…"
   },
   ```

3. Llenar cada campo:
   1. La llave (`calculo_alan`): el id definido en el paso 1.
   2. `nombre`: el personaje y la materia, separados por «·».
   3. `did_agent_id` y `did_client_key`: los valores del paso 4.
   4. `elevenlabs_agent_id`: el id del paso 5, o `""` si no hay respaldo.
4. Revisar las comas: cada bloque se separa del siguiente con una coma y el último no la lleva.
5. Pulsar **Commit changes…**, escribir un mensaje como «Alta de Alan Turing · Cálculo I» y dejar seleccionado **Commit directly to the `main` branch**.

Nunca se cambia la llave de un agente ya publicado: su iframe en Rise dejaría de funcionar. Tampoco se suben archivos `.html` a la raíz; la página se genera sola. Los cambios a `src/`, `scripts/` o `.github/` no son altas: van por rama y pull request, y los revisa @ErikV-AG.

## Paso 7. Validar y publicar

El commit lanza solo la publicación. Si algo en `agentes.json` está mal, no se publica nada y el sitio queda como estaba, así que un error en este paso no rompe ningún curso.

1. Abrir la pestaña **Actions** del repositorio y entrar a la ejecución más reciente de **Publicar**.
2. Esperar el resultado:
   1. Verde: el sitio ya está actualizado. GitHub Pages tarda hasta 10 minutos en refrescar su caché.
   2. Rojo: no se publicó nada. El resumen de la ejecución explica el error en español y la línea de `agentes.json` donde está: una coma faltante, un id repetido, una key mal copiada.
3. Si salió en rojo, corregir `agentes.json` con otro commit directo en `main` y volver al punto 1.

Un aviso amarillo de «se publicó sin pasar por agentes.json» significa que alguien subió un `.html` a la raíz. Se corrige con el procedimiento del [README](../README.md), sección «Para colaboradores».

## Paso 8. Probar el agente publicado

La prueba se hace en el sitio publicado, porque en local D-ID no conecta: sus llaves solo aceptan los dominios autorizados.

1. Abrir [lista.html](https://innovacion-ag.github.io/profesores/lista.html), buscar el agente y entrar a su página.
2. Pulsar **Iniciar conversación** y aceptar el permiso de micrófono.
3. Comprobar que:
   1. El avatar aparece y saluda en español.
   2. Responde por voz y por texto una pregunta de la materia.
   3. Los textos de la interfaz están en español.
   4. El botón **✕ Terminar** cierra la sesión.
4. Repetir la prueba desde un celular.
5. Si algo falla, abrir la consola del navegador (Cmd+Option+J en Chrome). Los mensajes de la página empiezan con `[agente]`, y `__textosPendientes()` lista lo que quedó en inglés.
6. Revisar que el agente aparezca en [tablero.html](https://innovacion-ag.github.io/profesores/tablero.html) con su personaje y su materia.

## Paso 9. Entregar a DEA

El agente se entrega pegando su código de inserción en la Cédula. Con eso, DEA lo monta en el curso de Rise.

1. En [lista.html](https://innovacion-ag.github.io/profesores/lista.html), pulsar **Copiar iframe** en el agente. No se escribe a mano ni se copia de otra fila.
2. Pegarlo en la [Cédula](https://docs.google.com/spreadsheets/d/1I9YYzEdpFYIqBMzy-pgykguDBPYmFZ15uAWOYCB8ObA/edit#gid=764442467), hoja «Personajes cod», columna «Código de insersión», en la fila de la materia.
3. Comprobar que el `src` empiece con `https://innovacion-ag.github.io/profesores/`. Las filas anteriores a noviembre de 2026 tienen `erikv-ag.github.io`: funcionan por el puente, pero no se usan como modelo.
4. Marcar la columna «Activo» como «Sí» en la misma fila. La marca quien crea el agente.
5. Publicar en `#proj-profesores` que el agente está entregado: materia, personaje, id y enlace a su página.

La bitácora ejecutiva lee la Cédula sola, así que no hace falta reportar la entrega en otro lado.

## Problemas frecuentes

| Síntoma | Causa probable | Qué hacer |
| --- | --- | --- |
| Publicar sale en rojo | Error de formato en `agentes.json`: coma, id repetido o key mal copiada | Leer el resumen de la ejecución en Actions, corregir la línea indicada y hacer otro commit |
| La página muestra «El profesor no está disponible» | D-ID no conecta y el agente no tiene respaldo | Revisar los dominios permitidos y los créditos de D-ID; crear el respaldo del paso 5 |
| La página cambia sola a la voz de ElevenLabs | Sin créditos o sin sesiones libres en D-ID | Revisar el tablero de la cuenta «Proyectos Innovación»; los créditos se renuevan el día 18 |
| Se queda en «Cargando…» o en «Alta demanda» | Sesiones simultáneas agotadas en el plan de D-ID | Esperar o probar más tarde; si se repite, avisar en `#proj-profesores` |
| En Rise no se pide el micrófono | El iframe no tiene `allow="microphone"` | Volver a copiarlo de `lista.html` con **Copiar iframe** |
| Quedan textos en inglés | El widget de D-ID cambió sus textos | Ejecutar `__textosPendientes()` en la consola y reportarlo en `#proj-profesores` |
| El agente quedó en otra cuenta de D-ID | Se creó fuera de «Proyectos Innovación» | Crearlo de nuevo en la cuenta correcta y actualizar sus valores en `agentes.json` |
| Un iframe de Rise dejó de funcionar | Se cambió o borró la llave de un agente publicado | Restaurar la llave anterior; la publicación lo impide salvo que se haya forzado el borrado |

## Lista de verificación

Se copia en el hilo de `#proj-profesores` de cada alta y se marca conforme se avanza.

- [ ] Fila de la materia revisada en la Cédula e id del agente definido
- [ ] System prompt y base de conocimiento revisados y guardados en Drive
- [ ] Video de presentación en Drive y registrado en la Cédula
- [ ] Agente creado en la cuenta «Proyectos Innovación» de D-ID, con los dos dominios permitidos
- [ ] Agente de respaldo creado en ElevenLabs
- [ ] Alta en `agentes.json` con commit directo en `main`
- [ ] Publicar en verde en Actions
- [ ] Prueba completa en computadora y celular
- [ ] Código de inserción con `innovacion-ag.github.io` pegado en la Cédula y «Activo» marcado
- [ ] Entrega avisada en `#proj-profesores`

## Referencias

- [README](../README.md), [operación](operacion.md) y [blueprint](blueprint.md) de este repositorio
- Sitio: [lista.html](https://innovacion-ag.github.io/profesores/lista.html) y [tablero.html](https://innovacion-ag.github.io/profesores/tablero.html)
- [Cédula de Seguimiento de materias](https://docs.google.com/spreadsheets/d/1I9YYzEdpFYIqBMzy-pgykguDBPYmFZ15uAWOYCB8ObA/edit#gid=764442467), hoja «Personajes cod»
- [Carpeta de Drive del proyecto](https://drive.google.com/drive/folders/1HXtWZWCgBk6wj0-Q4GHy_we8GcY26FaU)
- Credenciales: gestor de contraseñas del área, grupo `profesores`
- Dudas: canal `#proj-profesores`. Responsable: @CoriaAlx. Suplente: @rodrig317-cpu. Supervisión: @ErikV-AG.
