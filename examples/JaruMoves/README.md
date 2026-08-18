# JaruMoves

Homenaje a **Army Moves** (Dinamic Software, 1986) en la linea de su
conversion de Amiga: cuatro fases encadenadas, panel inferior y fondos
de varias capas. Escrito entero en JARU, con los personajes y vehiculos
como sprites y el escenario como tilemap sobre un `GridMap`.

**El juego arranca en modo demostracion y se juega solo.** En Windows,
`D` te da el mando y `D` otra vez se lo devuelve al piloto automatico.
En placas sin botones cableados no hay forma de salir de la demo, que es
justo lo que se quiere en una consola de escaparate.

## Las cuatro fases

| # | Fase | Como se juega |
|---|------|---------------|
| 1 | **El puente** | El jeep avanza con el scroll, salta los huecos de la cubierta y barre a tiros los helicopteros que sueltan bombas y los jeeps que vienen de frente. Caerse al rio es mortal. Sobre los tramos largos hay una pasarela alta con botiquines. |
| 2 | **Helicoptero** | Vuelo rasante al atardecer. Cazas, helicopteros y baterias antiaereas ancladas al suelo. Rozar la copa de los arboles derriba. |
| 3 | **El rio** | El buzo cruza a nado entre minas fondeadas y torpedos humanos. El arpon las revienta a distancia; tocar una mina mata en el acto. |
| 4 | **La jungla** | A pie, con la camara siguiendo al soldado. Fosos, troncos suspendidos, soldados y nidos de mortero, hasta el cuartel enemigo donde estan los documentos. |

## La musica

Seis piezas propias: una por fase en bucle y dos fanfarrias que suenan una
sola vez. Nada de transcribir el tema original de Army Moves: son
composiciones nuevas en el registro que le pega a cada fase.

| Fase | Tema | Duracion | Como suena |
|------|------|----------|------------|
| 1 | `Bridge` | 26,6 s | Marcha militar en re menor: bombo y caja alternos, bajo de fundamental y quinta |
| 2 | `Air` | 21,5 s | Mi menor, mas rapido; bajo a corcheas machaconas y charles constante |
| 3 | `River` | 24,6 s | Lento y con aire, **sin percusion**: seno de ataque largo, bajo grave y una gota suelta |
| 4 | `Jungle` | 27,6 s | Do menor, metal por delante y contracanto que entra a cerrar cada grupo de cuatro |
| — | `Clear` | 2,5 s | Fanfarria de fase superada (1 a 3): triada de metal subiendo y redoble |
| — | `Victory` | 9,0 s | Final de juego: cinco compases en do mayor, metal, pad, arpegio y remate en do agudo |

Las dos fanfarrias van con `loop = false`, y eso importa mas de lo que
parece: `PlayTrack` deja `musicOk` a false en ellas para que la red de
seguridad que relanza la musica caida no las vuelva a arrancar al terminar,
que es justo lo contrario de lo que se quiere.

Los carteles de FASE SUPERADA y de final **esperan a que acabe su
fanfarria** (`MusicBusy`), en vez de durar un numero fijo de fotogramas: los
fps no son los mismos en Windows que en la placa y el cartel se quedaria
corto o largo segun donde corra. El contador de estado es solo el minimo.

Las compone `Tools/build_music.py`, que se ejecuta solo desde
`gen_assets.py`. Escribe los dos formatos, por el mismo motivo que los
sprites: `Music/<N>.jms` es la fuente del editor del IDE y
`Build/Flash/Music/<N>.jmu` es el binario que carga la VM.

Los temas se escriben separando armonia de melodia: la estructura es una
lista de (acorde, figura) por compas, el bajo y el arpegio se derivan del
acorde solos, y la melodia sale de figuras en GRADOS relativos al acorde.
La misma figura sobre otro acorde suena distinta, que es como se saca
variedad sin escribir mil notas a mano.

Solo hay **un canal de ruido**, asi que la bateria es una sola voz que
cambia de periodo: grave para el bombo, medio para la caja, agudo para el
charles. En el canal 4 la "nota" es el PERIODO, no una nota MIDI.

`build_music.py` rehace las comprobaciones del decodificador de la VM sobre
los bytes antes de escribir. Merece la pena: si el binario esta mal,
`Sound.loadMusic` devuelve `false` y el juego se queda mudo sin decir donde.

## Controles (Windows)

| Tecla | Accion |
|-------|--------|
| Flechas | Mover / subir y bajar |
| `Z` | Disparar |
| `X` | Saltar |
| `D` | Tomar el mando / volver a la demo |
| `N` | Saltar a la fase siguiente |
| `R` | Volver al titulo |

En ESP32 generico: GPIO 39 arriba, 38 abajo, 37 disparo, 36 salto,
35 avanzar. En placas sin botones cableados no se mapea
nada: los GPIO 34-39 no llevan pull-up interno y sin boton cableado
flotan, asi que el juego se queda en demo permanente.

## Como esta montado

- `Code/JaruMoves.aru` — motor, entidades, piloto automatico, dibujado y
  maquina de estados.
- `Code/JaruMoves_Phases.aru` — geometria y oleadas de las cuatro fases.
- `Code/JaruMoves_Tiles.aru` — **generado**: indices del tileset.

### Sonido en placa

El juego **no levanta ningun codec de audio**: llama a `Sound.init()` y fija
el volumen, y de la salida se encarga el firmware segun la seccion `audio`
del perfil del dispositivo (`Config/ESP32/device_profiles.json`; en la
Waveshare S3 2.8" es I2S directo por bclk 48 / ws 38 / dout 47).

Esto costo una version muda en placa. El proyecto nacio copiando a
`JaruType`, que **si** arranca a mano el ES8311 de la Freenove FNK0104A, y
ademas hacia depender `soundOk` de que ese arranque saliera bien. En
cualquier otra placa el codec no responde, `soundOk` se quedaba en `false` y
enmudecia todo, musica y efectos. Los ejemplos que suenan en varias placas
--`JaruQuest`, `JaruBunny`-- no tocan el codec.

Otras dos que van en el mismo lote:

- **El volumen maestro en placa tiene que ser alto** (190). Venia en 10, que
  es lo que le pega a una salida con ganancia de codec detras, no a I2S
  directo: aun con el codec arrancando, a ese nivel no se oye nada.
- **`setMusicVolume` va DESPUES de cada `playMusic`**, no una vez al inicio:
  arrancar una cancion resetea el volumen de musica al de su cabecera.
- `Tools/gen_assets.py` — genera todo el arte (ver mas abajo).

Las cuatro fases comparten un unico modelo de escenario: dos arrays con
una entrada por columna de 8 px, `surfY` (superficie solida) y `overY`
(plataforma atravesable desde abajo). Con eso salen el puente con sus
huecos y su pasarela, el perfil de colinas que el helicoptero no puede
tocar, el lecho del rio y el suelo de la jungla con fosos y tablones.

**La colision no pasa por el `GridMap`.** El tilemap es solo la capa de
dibujado; quien decide donde se pisa sigue siendo `surfY`/`overY`, que es
de lo que dependen la parabola del salto y `PredictLanding`. Los tiles
miden 8x8, o sea exactamente el `TCOL` del juego, asi que las dos
rejillas coinciden 1:1 y no hay que traducir nada.

El heroe tambien es uno solo con cuatro modos; jeep y soldado comparten
la fisica de suelo, helicoptero y buzo la de movimiento libre.

## El arte

Todo lo dibuja `Tools/gen_assets.py` desde codigo, no hay ningun fichero
pintado a mano:

```bash
python Tools/gen_assets.py
```

- `Tools/palette.py` — 26 tonos, **ya cuantizados a RGB565**, que es lo
  que sobrevive al viaje a pantalla (un 255 vuelve como 254). Cuantizar
  en origen hace que `Tools/preview.png` sea exactamente lo que se ve.
- `Tools/art_tiles.py` — 28 tiles de 8x8 como bloques de texto.
- `Tools/art_sprites.py` — 14 sprites, 23 fotogramas. No van como texto
  sino pintados sobre un lienzo de caracteres con primitivas: a 32 de
  ancho, un caracter de mas descuadra la figura entera y no hay forma
  comoda de verlo. El contorno se aplica de un tirazo con `outline()`.
- `Tools/preview.png` — todo junto a x4 para revisarlo de un vistazo.

El generador tambien **parchea el `.jpr`** (respetando todo lo demas: las
claves de nivel superior, la carpeta `Code` y cualquier metadato que haya
puesto el IDE). No es un capricho: sin eso el Resource Builder del IDE
no construye. Necesita dos cosas que no se pueden escribir a mano sin que
acaben desincronizandose:

- Cada nodo de imagen tiene que llevar **`"SaveToData": true`** en su
  metadata. Por defecto es `false`, y sin ella el build suelta
  `Skipping image (SaveToData is false)` y luego no encuentra nada.
- Cada `.spr` tiene que llevar en **`imageNodeID` el ID real del nodo de
  su imagen dentro del `.jpr`**: el builder hace `GetNodeByID` con ese
  numero y si falla aborta con `Source image for sprite not found`.

Por eso los `.spr` se escriben DESPUES de resolver los ids de los nodos,
en la misma pasada. Si se anaden o quitan sprites, basta con volver a
ejecutar `gen_assets.py`.

Cinco cosas que conviene saber si se toca esto:

1. **`Sprite.load("X.spr")` no lee el `.spr`.** Le quita la extension y
   abre `X.json` mas un `X_000.bmp` por fotograma (`Sprite_Load` en
   `MSprite.cpp`). Los `.spr` de `Sprites/` son para el editor del IDE;
   lo que consume la VM lo escribe el generador directamente en
   `Build/Flash/Sprites/`. Por eso `Tools/run.ps1` funciona sin abrir el
   IDE ni una vez.
2. **En los tiles, el transparente es el negro PURO** (`0x000000`, el
   `_DRAW_TRANSPARENT_COLOR` de la VM), comprobado igual en el
   renderizador de Windows y en el del ESP32. Por eso el contorno `K` de
   la paleta es un casi-negro `(16,16,24)`: uno negro de verdad seria un
   agujero. En los sprites la clave es el magenta del `.json`.
3. Los sprites son **plantillas compartidas**: todos los helicopteros
   enemigos dibujan el mismo objeto movido de sitio. No se aceleran por
   ello porque el avance de fotograma va por reloj, no por llamada a
   `draw.sprite`.
4. El `.json` que escribe `gen_assets.py` coincide campo a campo con el
   que escribe el Resource Builder del IDE, asi que da igual cual de los
   dos haya construido `Build/flash/Sprites`.
5. El build del IDE **borra `Build\flash\Sprites` entero** antes de
   regenerarlo. Conviene cerrar el juego si esta corriendo, o el borrado
   puede fallar por ficheros abiertos.

### El piloto automatico

No es un jugador aparte: escribe las mismas variables de entrada
(`inLeft`, `inRight`, `inFire`, `inJump`...) que rellenaria el teclado,
asi que de ahi para abajo corre exactamente el mismo codigo que en una
partida humana. Tiene tres piezas que costaron lo suyo de afinar:

- **Salto de huecos.** La distancia se mide al CENTRO del vehiculo, no
  al morro: el jeep empieza a caer cuando su centro pasa el borde, y
  medir desde el parachoques hacia saltar demasiado pronto y aterrizar
  dentro del hueco.
- **`PredictLanding`.** En el aire no decide por instinto: rehace la
  caida fotograma a fotograma con cada marcha y se queda con la que cae
  en suelo firme. Acelerar siempre parece lo prudente y no lo es —al
  bajar de la pasarela el jeep gana 26 px de alcance y se planta justo
  dentro del hueco siguiente.
- **`BestLane`.** En helicoptero y a nado no esquiva "la amenaza mas
  cercana" sino que puntua alturas candidatas cada 8 px y elige la mas
  despejada. Con una mina lo primero funciona; con cuatro te mete en la
  siguiente.

- **Disciplina de fuego.** No dispara sin parar: solo con blanco dentro de
  la LINEA DE TIRO —ni el canon del jeep ni el fusil del soldado apuntan,
  tiran en horizontal—, tras un instante de reaccion y en rafagas de dos a
  cuatro disparos separadas por medio segundo largo de silencio. Medido
  entre 1,5 y 3,1 disparos por segundo segun la fase, frente a los 8 del
  gatillo pegado.

  Lo que hace que una rafaga se LEA como rafaga no es disparar poco, sino
  que los tiros vayan juntos y el silencio de despues sea largo: con el
  mismo intervalo dentro y fuera del grupo sale un chorro irregular. De ahi
  que el enfriamiento entre disparos sea corto (7 fotogramas) y la pausa
  entre rafagas larga (26-46).

- **Saltar para disparar.** Los helicopteros de la fase 1 entran por arriba
  y bajan en picado hasta una de DOS alturas de pasada, alternandose:
  `CHOP_LOW` (138) se barre desde la cubierta, y `CHOP_HIGH` (114) queda
  fuera del canon —que tira en horizontal— y **hay que SALTAR** para
  tirarle, como en el original. El piloto lo busca a proposito: salta con
  el helicoptero todavia por delante (el jeep tarda en subir y la bala en
  llegar) y solo por los altos, porque saltar por uno bajo no aporta nada.
  Nunca si hay un hueco cerca, que los huecos mandan.

  Dos cosas que hubo que afinar para que las alturas signifiquen algo:

  1. El cabeceo se suma a la altura de pasada, no a la Y. Sumarlo a la Y
     INTEGRA el seno, y con paso 0,07 eso no es el vaiven de +-1,4 que
     parece sino uno de +-20, que se comia la diferencia entre las dos
     alturas.
  2. El descenso tiene que ser rapido (~30 fotogramas). Con el picado lento
     del primer intento ninguno llegaba a su altura antes de cruzarse con
     el jeep y los dos tipos volaban igual de altos: medido, 0 derribos de
     5 helicopteros altos.

Ademas hay un vigilante de atasco: la fase a pie la mueve el propio
heroe, asi que si el piloto se planta —subido a un tronco, disparando
por encima del casco enemigo— a los dos segundos tira hacia adelante
pase lo que pase.

Medido en la VM de Windows: la demo completa las cuatro fases con una
o ninguna baja y termina rondando los 21.000 puntos.

## Compilar y ejecutar sin el IDE

```powershell
.\Tools\run.ps1              # compila y ejecuta
.\Tools\run.ps1 -Seconds 60  # lo mata a los 60 s
.\Tools\run.ps1 -BuildOnly   # solo compila
```

## Notas de rendimiento

- 53,6 FPS de media en la VM de Windows a 320x240 (267 muestras de una
  partida completa). El minimo, 35, cae en los cambios de fase, que es
  cuando se repinta el tilemap entero.
- `SetupSpeed()` escala las velocidades por `k` y la gravedad por `k*k`,
  de modo que la PARABOLA del salto es la misma en el mundo aunque en el
  ESP32 haya la mitad de fotogramas para recorrerla.
- En el bucle de juego no se llama a `draw.text`: rasteriza la TTF en
  cada llamada. El marcador va con digitos de siete segmentos hechos de
  rectangulos (siete como mucho por cifra). El texto solo aparece en el
  titulo y en los carteles de fase, donde sobran fotogramas.
- El `GridMap` se reserva UNA vez con el ancho de la fase mas larga y
  despues solo se limpia y se repinta. Crear uno nuevo por fase deja
  12 KB de basura cada vez, y en el monton del ESP32 eso es pedir
  fragmentacion a cambio de nada.
- Topes de entidades fijos (14 enemigos, 12+18 disparos, 10 explosiones)
  para que el tiempo de cuadro del ESP32 no se dispare.

## Pendiente

- **Sin probar en placa.** Ademas del tiempo de cuadro, hay que mirar la
  memoria: el arte ocupa del orden de 46 KB en fotogramas de sprite mas
  12 KB del grid y 4 KB del tileset, y eso en un ESP32 no es despreciable.
- Los cuatro `.jmu` estan verificados contra la VM (cargan, arrancan y
  siguen sonando tras un segundo real de reproduccion). Los `.jms` de al
  lado se escriben segun el esquema del editor pero **solo el IDE puede
  confirmarlos**: el juego no depende de ellos, solo del binario.
