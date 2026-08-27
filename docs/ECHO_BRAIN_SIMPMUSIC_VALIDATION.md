# Validación FOSS de SimpMusic Echo Brain

## Revisión validada localmente

| Elemento | Resultado |
| --- | --- |
| Base | SimpMusic estable `v1.7.0` (`5dc8736b`), con el submódulo `jobjgmailcom/core` `7112059`. |
| Planificador | Pruebas JVM correctas para repetición, variantes, umbrales y gatillos. |
| Android | `:androidApp:assembleDebug` correcto con Java 21, Android API 37, `--no-daemon` y un trabajador. |
| Paquete | `com.maxrave.simpmusic.dev`, versión `1.7.0-dev`/`56`. |
| Firma | Esquema APK v2 correcto mediante la firma debug reproducible de Android. |
| ABI | `arm64-v8a`, `armeabi-v7a`, `x86_64`. |
| Telemetría | No hay Firebase, Google Play Services ni Sentry en DEX. |
| Servicios opcionales | Last.fm usa sólo el stub `lastfm-empty`; no hay endpoint, secreto ni cliente funcional. |

## Garantías de integración

Echo Brain usa el radio relacionado que SimpMusic ya pide para el reproductor. No incorpora un backend, telemetría, claves, extracción de streams ni un bucle de sondeo. La cola original no se borra: las candidatas filtradas se insertan mediante la misma operación `playNext` del reproductor. Crossfade, shuffle, Android Auto y el orden de la cola permanecen en sus capas nativas.

La configuración se guarda en `DataStoreManager` del dispositivo. El historial de inyecciones conserva claves canónicas de canción durante 24 horas para excluir repeticiones. Echo Brain inicia desactivado y, cuando se activa, añade sólo detrás de la pista actual sin sustituir, borrar ni reordenar la cola original.

## Protección de inicio de reproducción

El cierre reportado al iniciar una canción se aisló en el análisis de un historial local vacío: una línea sin separador podía solicitar una subcadena de longitud negativa. El lector ahora descarta líneas inválidas antes de tratarlas y las rutas de recomendación capturan fallos no cancelables, continúan con la radio normal de SimpMusic y registran el motivo. Por tanto, Echo Brain no puede impedir el arranque ordinario del reproductor si su ciclo falla.

## Ejecución remota verificable

La ejecución [33034392414 de GitHub Actions](https://github.com/jobjgmailcom/SimpMusic/actions/runs/33034392414) terminó correctamente para el commit `87114cef`. Ejecutó las pruebas `:domain:jvmTest`, ensambló `:androidApp:assembleDebug` con Java 21, sin daemon y un trabajador, y publicó el artefacto FOSS sin secretos.

La APK universal descargada de dicha ejecución se comprobó de forma independiente: paquete `com.maxrave.simpmusic.dev`, versión `1.7.0-dev`/`56`, firma v2 válida, ABIs `arm64-v8a`, `armeabi-v7a` y `x86_64`, y SHA-256 `35f8c10e00e544ec5ab3096ecf31ca04c35e2d8fd6ccbf66434071e04870cc3a`. El examen DEX no encontró Firebase, Google Play Services, Sentry ni el endpoint real de Last.fm.

## Límite de validación

Una compilación y las pruebas no pueden probar las respuestas futuras de YouTube Music ni la reproducción real en cada teléfono. La CI compila y audita el APK sin secretos; la prueba en dispositivo debe confirmar que una canción normal inicia, que Echo Brain agrega candidatas sólo al activarlo y que desactivarlo conserva el comportamiento habitual.
