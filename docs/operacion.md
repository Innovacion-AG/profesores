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
| «Alta demanda» o sala de espera frecuente | Reportes de DEA o de alumnos | Revisar las sesiones simultáneas del plan de D-ID. |
| Una URL antigua no carga | Abrir `https://erikv-ag.github.io/profesores/<id>.html` | Revisar que `ErikV-AG/profesores` exista y tenga GitHub Pages activo. |

PENDIENTE: ¿quién recibe los reportes de falla de DEA y por qué canal, además de `#proj-profesores`?

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
| Área usuaria | DEA: asigna los personajes, monta los agentes en Rise y da seguimiento en la cédula |
| Proveedor de video | D-ID. PENDIENTE: ¿canal de soporte y titular de la cuenta? |
| Proveedor de voz | ElevenLabs. PENDIENTE: ¿canal de soporte y titular de la cuenta? |
| Canal | `#proj-profesores` en Slack |
