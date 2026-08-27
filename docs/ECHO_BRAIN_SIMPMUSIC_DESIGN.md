# Echo Brain para SimpMusic

## Alcance

Esta adaptación añade una capa **local, determinista y opcional** sobre la cola de SimpMusic. No modifica el extractor de streams, no añade un servidor, no introduce telemetría y no crea una cuenta adicional. Aprovecha `SongRepository.getRelatedData(videoId)`, que SimpMusic ya usa para extender una radio, y su operación `MediaPlayerHandler.playNext(track)`, que inserta una pista inmediatamente detrás de la activa y mantiene la cola compartida.

| Componente SimpMusic | Responsabilidad de Echo Brain |
| --- | --- |
| `MediaServiceHandlerImpl` | Detectar una nueva pista activa, solicitar candidatas sólo cuando corresponde e insertarlas detrás de ella. |
| `QueueData` + reproductor | Conservar la cola original, el índice, shuffle, crossfade, Android Auto y el mecanismo de carga existente. |
| `SongRepository.getRelatedData` | Entregar el radio relacionado que se filtra; Echo Brain no resuelve URLs ni toca la fuente de audio. |
| `DataStoreManager` | Persistir localmente preferencias, diagnóstico y el historial de repetición. |
| `SettingScreen` + `SettingsViewModel` | Mostrar controles visibles siguiendo el diseño nativo de SimpMusic. |

## Ciclo de inyección

1. Cuando cambia la pista activa, Echo Brain toma una instantánea de la cola y del índice.
2. Si está activado y la compuerta no tiene otro ciclo en curso, consulta el radio que SimpMusic ya obtiene para esa pista.
3. El planificador descarta duplicados, una repetición del mismo registro, alternativas cuando están desactivadas y canciones dentro del bloqueo de 24 horas.
4. Ordena únicamente las candidatas válidas por relación con el ancla, contexto local y señales de diversidad. El año no penaliza: una canción relacionada de otra década puede seguir siendo válida.
5. Inserta hasta tres candidatas **en orden inverso** mediante `playNext`, para que el resultado final mantenga el orden elegido inmediatamente después de la pista activa.
6. Al acercarse al fin de una tanda Echo Brain, vuelve a evaluar una vez. Si ningún candidato supera el filtro, la cola original continúa sin ser eliminada.

> Echo Brain nunca obliga una canción sólo para llenar espacio. Si el filtro no encuentra una candidata aceptable, deja pasar la cola de SimpMusic.

## Reglas y preferencias

| Preferencia | Valor inicial | Efecto |
| --- | --- | --- |
| Echo Brain | Activado | Permite la evaluación automática; apagarlo deja SimpMusic sin inserciones adicionales. |
| Similitud mínima | 90 | Muestra las opciones 90, 80, 70 y 60. La relación de radio aporta la base; artista, álbum y sesión reciente pueden sumar afinidad. |
| Versiones alternativas | Desactivado | Rechaza remix, live, acústica, cover, karaoke, slowed, sped up y ediciones equivalentes. |
| Diversidad de artistas | Equilibrada | Evita agotar un único artista: ocho artistas recientes en equilibrada, doce en alta, sin límite en ilimitada. |
| Confirmación de escucha | 60 % | Una escucha significativa mejora las señales locales; un salto antes del 20 % es una señal temporal negativa. |
| Continuidad de cola | Dominante | Tras una pista original que no recibe candidata válida, Echo Brain puede volver a evaluar en la siguiente oportunidad. El modo conservador mantiene mayor separación. |
| Bloqueo de repetición | 24 horas | Impide volver a inyectar el mismo registro o su equivalente canónico durante ese periodo. |

## Protección de la cola

La implementación usa una compuerta atómica para impedir dos inserciones simultáneas. Las candidatas se etiquetan sólo en el estado local de Echo Brain, sin modificar la identidad de `Track`. Se conserva el comportamiento nativo de `QueueData`, y `playNext` realiza las actualizaciones de reproductor, lista y orden de shuffle que SimpMusic ya requiere.

## Validación prevista

Las pruebas puras cubrirán normalización de identidad, duplicados por ID y por título/artista, variantes, umbrales, diversidad, cooldown, gatillos de recarga y exclusión concurrente. Las pruebas de ensamblado usarán la variante FOSS de SimpMusic: sin `isFullBuild=true`, sin Sentry, Last.fm ni claves de firma.

## Licencia y procedencia

SimpMusic y el paquete Echo Brain aportado se distribuyen bajo GNU GPL v3. Esta adaptación conservará avisos de modificación y la atribución a la capa local Flow-compatible incluida por el paquete de referencia, sin copiar la integración de MetroList que depende de sus modelos y servicios propios.
