# Auditoría inicial de fuentes

## SimpMusic

| Campo | Hallazgo |
| --- | --- |
| Repositorio | [maxrave-dev/SimpMusic](https://github.com/maxrave-dev/SimpMusic) |
| Rama inspeccionada | Estable `v1.7.0`, revisión `5dc8736bcf478a4c6252a3a9cd403264ea120560` |
| Licencia | GNU GPL v3.0 (`LICENSE` del repositorio) |
| Arquitectura | Compose Multiplatform, módulos Android/desktop y submódulo interno `core` fijado en `dae3ce98f8b220f9d40ec9bc8134583f3a14c64b` |
| Audio y candidatas | El README declara YouTube Music/YouTube como backend. La cola Android se centraliza en `MediaServiceHandlerImpl`, que ya ofrece `playNext(track)` para insertar inmediatamente detrás de la pista activa y `getRelated(videoId)` para obtener pistas relacionadas. |
| Variante FOSS | `isFullBuild=false` queda fijado en `gradle.properties`; `androidApp` usa `crashlytics-empty`, `lastfm-empty` y `cast-empty`. También se retiró el plugin Android de Sentry para que no incorpore su SDK o sus binarios nativos. |

El workflow upstream `.github/workflows/android.yml` no es adecuado como base FOSS directa: forzaba `isFullBuild=true`, escribía secretos de Sentry y Last.fm, y firmaba con secretos de repositorio. En este fork quedó desactivado como compilación de publicación; `.github/workflows/echo-brain-foss-ci.yml` compila `androidApp:assembleDebug`, no recibe secretos y realiza una auditoría del APK.

## Paquete Echo Brain aportado

El usuario proporcionó `Echo-Brain-MetroList-FOSS-Source-030d65e(1).zip`. Se verificó su integridad con `unzip -t` y se extrajo únicamente para inspección en un directorio aislado, sin ejecutar código. El paquete declara procedencia de MetroList FOSS revisión `030d65e71ec9e7b3198175871dd3a481ec201368`, licencia GPL v3 y atribución separada para la capa Flow-compatible.

Los módulos reutilizables a adaptar son el planificador determinista, la compuerta contra inyecciones concurrentes, preferencias locales, diagnóstico, cooldown y pruebas. La integración específica de MetroList no se copiará de forma mecánica: SimpMusic usa `Track`, `QueueData` y `MediaPlayerHandler`, por lo que la adaptación debe utilizar sus propias APIs de cola, reproducción y pistas relacionadas.

## Implementación y comprobación local

El submódulo `core` del fork apunta a `jobjgmailcom/core` revisión `71120594365c063af78f1edc09f8105081a3b189`, que contiene la adaptación estable. El planificador sólo recibe candidatas de `SongRepository.getRelatedData`, no resuelve streams ni efectúa solicitudes propias. `MediaServiceHandlerImpl` usa `playNext` para insertar hasta tres candidatas en orden correcto detrás de la pista activa y reinicia sus marcas en memoria si el usuario carga otra cola. El historial de grabaciones canónicas se guarda localmente durante 24 horas; las líneas de historial vacías se descartan antes de analizarse para no afectar el arranque de audio.

La comprobación local final ejecutó `:domain:jvmTest` y `:androidApp:assembleDebug` con Java 21 y un trabajador Gradle. La APK resultante fue `com.maxrave.simpmusic.dev` `1.7.0-dev`/`56`, con firma v2 y ABIs `arm64-v8a`, `armeabi-v7a` y `x86_64`. Un escaneo de DEX no detectó Firebase, Google Play Services ni Sentry. La coincidencia del nombre `org.simpmusic.lastfm` procede exclusivamente del módulo no operativo `lastfm-empty`; no contiene punto de red `ws.audioscrobbler.com`, claves ni credenciales.
