# Validación FOSS de SimpMusic Echo Brain

## Revisión validada localmente

| Elemento | Resultado |
| --- | --- |
| Base | SimpMusic `dev` 88749c51, con el submódulo `jobjgmailcom/core` fc65fda. |
| Planificador | Pruebas JVM correctas para repetición, variantes, umbrales y gatillos. |
| Android | `:androidApp:assembleDebug` correcto con Java 21, Android API 37, `--no-daemon` y un trabajador. |
| Paquete | `com.maxrave.simpmusic.dev`, versión `2.0.0-dev`/`57`. |
| Firma | Esquema APK v2 correcto mediante la firma debug reproducible de Android. |
| ABI | `arm64-v8a`, `armeabi-v7a`, `x86_64`. |
| Telemetría | No hay Firebase, Google Play Services ni Sentry en DEX. |
| Servicios opcionales | Last.fm usa sólo el stub `lastfm-empty`; no hay endpoint, secreto ni cliente funcional. |

## Garantías de integración

Echo Brain usa el radio relacionado que SimpMusic ya pide para el reproductor. No incorpora un backend, telemetría, claves, extracción de streams ni un bucle de sondeo. La cola original no se borra: las candidatas filtradas se insertan mediante la misma operación `playNext` del reproductor. Crossfade, shuffle, Android Auto y el orden de la cola permanecen en sus capas nativas.

La configuración se guarda en `DataStoreManager` del dispositivo. El historial de inyecciones conserva claves canónicas de canción durante 24 horas para excluir repeticiones. Desactivar Echo Brain impide nuevos ciclos y mantiene la cola de SimpMusic sin alteraciones adicionales.

## Límite de validación

Una compilación y las pruebas no pueden probar las respuestas futuras de YouTube Music ni la reproducción real en cada teléfono. La CI compila y audita el APK sin secretos; la prueba en dispositivo debe confirmar que una canción normal inicia, que Echo Brain agrega candidatas sólo al activarlo y que desactivarlo conserva el comportamiento habitual.
