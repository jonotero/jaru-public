# JaruQuest

Aventura top-down estilo NES escrita en JARU. Un exterior de 4×4 pantallas,
una mazmorra de 4×3 con llave, puerta cerrada y jefe, cuatro tipos de enemigo,
espada, corazones y objetos. Todo el arte es original y se genera por script.

Está inspirado en los action-RPG de NES, no calcado de ninguno: el héroe viste
de azul, los enemigos son *spitter*, *hopper*, *ogre* y *bat* (nada de octoroks
ni moblins, que son criaturas de Nintendo) y el trofeo final es una gema, no un
triángulo dorado partido en triángulos.

El juego se juega solo cuando nadie toca los botones: es lo que se ve en una
placa sin mando conectado, y en Windows entra a los 12 segundos sin actividad,
como el modo demo de una recreativa.

```
Controles (Windows)   flechas mover · Z atacar · ENTER empezar
```

Los textos que ve el jugador y la traza de consola van en **inglés** (como en
el resto de proyectos JARU); los comentarios del código, en español.

## Estado

Lo que funciona, comprobado ejecutándolo:

- Los dos mundos, el cambio de pantalla con deslizamiento de cámara, y las
  transiciones exterior ↔ mazmorra.
- Movimiento top-down con resolución por ejes y deslizamiento en esquinas.
- Espada (corta arbustos, desvía proyectiles), daño por contacto, retroceso,
  invulnerabilidad y muerte.
- Los cuatro enemigos con su comportamiento propio, el jefe, los proyectiles,
  los objetos, la llave y la puerta cerrada.
- Los cuatro temas de música, compilados a `.jmu` por el Build del IDE.
- Monedas: caen de los enemigos (6 de cada 10; corazón 2 de cada 10) y hay
  algunas repartidas por las pantallas que quedan fuera de la ruta. Las del
  mapa no reaparecen una vez cogidas; las del botín sí, porque los enemigos
  también reaparecen.
- El piloto automático: navega con `GridMap.dMap`/`dir`, esquiva proyectiles,
  pelea con lo que estorba, recoge la llave, abre la puerta cerrada, mata al
  jefe y se lleva el trofeo. **Partida completa observada** de principio a fin.
- ~90-100 fps en Windows.

Lo que falta:

- **No está probado en placa.** Está preparado para ESP32 (perfil por defecto
  WaveShare 2.8 del `.jpr`) pero no se ha flasheado ni medido allí. El número
  que hay que mirar es el frame rate: `draw.scene` repinta los ~336 tiles
  visibles cada frame y en ESP32 no hay caché del tilemap.
- El piloto **gana poco**. Medido sobre una sesión de 400 s (8 vidas): 7 veces
  coge la llave, 3 abre la puerta del jefe, 1 termina el juego. El resto son
  muertes repartidas por la mazmorra. Juega decente, no bien — que era el
  objetivo declarado, pero hay margen de sobra: la esquiva solo reacciona a
  proyectiles ya lanzados y no tiene ninguna noción de "no meterse en una sala
  con poca vida".
- Sin botones físicos mapeados: cuando existan, se añaden en `SetupInput()` y
  se pone `hasInput = true`. El resto del juego no se entera, porque lee
  `Intent`, no el hardware.
- Los enemigos tienen **una sola vista** y se voltean con `flipX`, mientras
  que el héroe tiene tres direcciones dibujadas. Darles frente/espalda/perfil
  es el siguiente salto de calidad gráfica, y triplica su arte.
- Sin usar los dos canales PCM del motor.

## Cómo está montado

```
Code/JaruQuest.aru     el juego entero
Images/                tileset, hojas de sprites y portada (generados)
Sprites/  Maps/        fuentes del editor del IDE (generadas)
Build/Flash/           lo que carga la VM (generado)
Tools/                 los generadores y el script de compilar+ejecutar
```

En runtime son **~390 KB** en total: 230 KB la portada, 68 KB el programa
compilado, ~42 KB los frames de los sprites, ~40 KB el tileset (117 tiles: 24
dibujados y el resto variantes de autotiling) y 14 KB los cuatro `.gmap` de
los dos mundos. La partición LittleFS de la placa son 2,7 MB, así
que el contenido no es la restricción; lo será el frame rate.

La portada merece un aviso aparte: son 320×240, o sea **150 KB en RAM** cuando
está cargada (más que todo el resto del juego junto). Por eso se carga al
entrar en el título y se suelta en `NewGame()`, para que su pico de memoria no
coincida con el de los mundos.

Nada de `Images/`, `Sprites/`, `Maps/`, `Build/` ni `JaruQuest.jpr` se edita a
mano: sale de `Tools/`. Ver [Tools/README.md](Tools/README.md) para regenerar,
para la lista de validaciones que hacen los generadores y para las tres
restricciones del motor que condicionan el arte (lienzo de 8 bits, el negro
como color transparente de los tiles, y el tope de 8 resultados de `hitsGrid`).

### Dos decisiones que explican el resto del código

**Un GridMap por mundo, no uno por pantalla.** La "pantalla" es solo lo que
encuadra la cámara; el mundo es una sola rejilla continua. Así la transición
entre pantallas es interpolar la cámara y no hay carga, ni enlazado de
pantallas que mantener, ni estado que guardar al salir de una sala. Cuesta
4,1 KB el exterior y 3,0 KB la mazmorra, que es un byte por celda.

**La entrada y el piloto producen lo mismo.** El bucle no lee botones: lee un
`Intent` con dirección y ataque. Los botones lo rellenan, o lo rellena el
piloto. Decidirlo desde la primera línea es lo que hace que el modo demo no
sea un parche, y es también la mitad de la IA de los enemigos: perseguir al
héroe y navegar hasta la llave son el mismo problema.

## Música

Cuatro temas: **título**, **partida**, **game over** y **victoria**. Se
escriben en [Tools/songs.py](Tools/songs.py), `gen_music.py` los convierte a
`Music/*.jms` (el formato del editor del IDE, que es JSON de tracker) y el
**Build del IDE los compila a `.jmu`**. El juego solo hace `Sound.loadMusic` /
`playMusic`.

| Tema | Cuándo | Carácter |
|---|---|---|
| Título | portada | lento y abierto sobre `Am F C G`, sin percusión, en bucle |
| Partida | jugando | cinco voces, en bucle |
| Game over | al morir | tres compases que bajan, **una vez** |
| Victoria | al coger la reliquia | fanfarria en **do mayor**, **una vez** |

Los cinco canales del tema de partida siguen el reparto del 2A03 de la NES
más el extra que JARU sí tiene:

| Canal | Voz | Instrumento |
|---|---|---|
| 0 | melodía | pulso duty 25 |
| 1 | contracanto | pulso duty 50 |
| 2 | bajo | **triangular** |
| 3 | arpegio | cuadrada, floja |
| 4 | percusión | ruido (bombo dark / caja / charles) |

La NES tenía que robarle un pulso a la melodía para hacer arpegios; aquí el
cuarto canal de tono los toca a la vez, a una nota por tick (42 ms).

### Por qué no está hecha por código

La primera versión era un secuenciador escrito en el propio `.aru` que
repartía notas desde el bucle del juego. Sonaba, pero tenía un fallo que
**solo aparece en la placa**: el tempo quedaba atado al frame rate. A 90 fps
el error por nota es ±11 ms y no se nota; a los 25-30 fps de un ESP32 sube a
±35 ms, que es casi lo que dura una nota del arpegio — el arreglo se deshace.
El reproductor de `.jmu` corre en la tarea de audio con su propio reloj.

De paso se gana lo que por código no había: **envolventes ADSR por
instrumento** (antes cada nota atacaba y cortaba a saco) y la posibilidad de
abrir las canciones en el tracker del IDE y retocarlas.

El mapeo de tiempos es 1:1 a propósito — `tickMs: 42` y `rowTicks: 1`, así que
una fila del tracker es exactamente un tick de `songs.py`. Sin eso habría que
redondear duraciones y el arreglo dejaría de cuadrar.

### Mezcla

Música y efectos van al mismo bus, así que lo que importa es la **relación**,
no el número absoluto. Todos los volúmenes están juntos al principio del
`.aru` (`MUSIC_VOLUME` y `SFXV_*`) para poder ajustarlos de oído sin buscarlos
por el código.

El volumen efectivo de una voz de música es `volumen_de_canal × MUSIC_VOLUME
/ 255` ([JaruAudio.cpp:1983](../../../JARU/vm/common/audio/JaruAudio.cpp)), y
los canales van de 110 a 215 según la voz. Con `MUSIC_VOLUME` a 120 la melodía
sale a ~100 y cualquier efecto (120-200) queda por encima.

:::warning
`Sound.setMusicVolume()` hay que llamarlo **después de cada `playMusic`**, no
una vez al arrancar: reproducir una canción restaura el volumen al de su
cabecera ([JaruAudio.cpp:1391](../../../JARU/vm/common/audio/JaruAudio.cpp)) y
se lleva por delante el ajuste. Puesto al inicio parece funcionar hasta el
primer cambio de tema.
:::

Los efectos comparten el canal 3 con el arpegio. No hace falta silenciar nada:
el mixer descarta un comando de prioridad menor mientras suena uno mayor
([JaruAudio.cpp:856](../../../JARU/vm/common/audio/JaruAudio.cpp)), así que el
efecto manda y el arpegio se salta las notas que caigan encima.

:::warning
**No hay compilador de música por línea de comandos**: `.jms` → `.jmu` lo hace
`uMusicResourceCompiler.pas`, dentro del IDE. Ejecutando con `run.ps1` el
juego avisa por consola (`>>> music not found`) y corre sin música. Para oírla
hay que hacer Build desde el IDE.
:::

## Animación del héroe

Ciclo de andar de **cuatro tiempos con tres dibujos** por dirección: reposo,
pie izquierdo, reposo, pie derecho (`WALK_CYCLE = [0, 1, 0, 2]`). La pose de
reposo aparece dos veces por ciclo, así que dibujarla una sola vez ahorra un
frame por dirección — 1,5 KB de RAM en total — sin que se note.

El ciclo avanza con la **distancia recorrida**, no con el reloj: una pose cada
8 px. Así el paso encaja con el movimiento a cualquier frame rate, que es
justo lo que no pasaría atándolo a un temporizador. Al soltar la dirección
`walkPhase` vuelve a 0, o el héroe se quedaría congelado a media zancada.

Lo que hace que se lea como zancada y no como deslizamiento son **dos** cosas
a la vez: el bloque de piernas se desplaza 1 px lateralmente **y** un pie se
levanta (su contorno termina una fila más arriba). Con solo el desplazamiento
parecía que patinaba.

## Colisión

El juego **no** usa el motor de físicas ni `hitsGrid`: consulta la capa de
celdas directamente y resuelve eje por eje. Para un top-down de cajas
alineadas es más simple, no tiene el tope de 8 tiles por consulta, y evita
traer gravedad, restitución y sueño de cuerpos a un juego que no los quiere.

El detalle que hace que se sienta bien es el deslizamiento en esquinas: si al
avanzar solo choca una de las dos esquinas de cabeza, el cuerpo se aparta un
poco hacia el lado libre en vez de clavarse. Tiene que ser **progresivo** —
la caja es de 12 px sobre tiles de 16, así que librar una esquina puede exigir
apartarse 5 px, y una versión que solo acepta el desplazamiento cuando el
avance ya cabe deja al héroe pegado a la esquina para siempre.

Y una trampa que costó cara: **el piloto mide distancias sobre el eje, no en
Manhattan**. El contacto que hace daño se comprueba por ejes (`Overlap`), así
que sumar los dos miente. Contra el jefe, que ocupa 28×28, una suma de 26 se
cumple con (18, 8) — que está metido dentro de su cuerpo. El piloto se
plantaba ahí creyéndose a salvo y perdía un corazón cada 900 ms; llegaba
entero al jefe y se dejaba la partida siempre en esa sala.
