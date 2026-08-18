# JaruPhantom

Plataformas de pantalla fija en JARU, inspirado en Phantomas (Dinamic, 1986).
No es un remake: los graficos son propios, dibujados desde cero con las reglas
del Spectrum (bloques de 8x8, dos colores por celda, paleta de 15 colores).

Un robot ladron entra por la **azotea** y BAJA planta a planta dando palancas;
con las treinta dadas cede la puerta de la camara acorazada del fondo, donde
esta el botin. Bajar es gratis --te dejas caer-- y volver a subir cuesta la
escalera entera, asi que equivocarse de planta se paga.

Tiene una sola vida, y la bateria **solo** se gasta al tocar un dron o los
pinchos: no hay reloj, hay que no comerse los golpes. Los cuadrados de vida
repartidos por la torre devuelven un trozo.

## Estado

**Torre de ocho plantas, jugable de principio a fin.** ~350 fps en Windows.

- Ocho plantas, cada una con **una idea propia**, de arriba abajo: La Azotea
  (las dos lecciones), Los Pilares, La Chimenea, El Pasillo, Los Forjados,
  El Voladizo, Las Torres y La Cripta.
- Tres biomas: mansion (piedra gris, suelo amarillo), cavernas (roca amarilla)
  y criptas (piedra cian, suelo verde). Es el mismo dibujo con otra tinta,
  como el byte de atributo del Spectrum.
- 30 palancas que **no desaparecen**: cambian de estado y se quedan puestas,
  asi que al volver a una sala se ve lo que llevas hecho. Van montadas en
  paredes o sobre consolas, no tiradas en el suelo, como en el original.
- Mobiliario: consolas, cajas, telaranas en las esquinas y nubes en la azotea.
- 8 cuadrados de vida, un dron por planta, fosos de pinchos.
- El piloto automatico **se pasa la torre entera solo** en un minuto: la ruta la
  calcula `Tools/route.py` fuera del juego y la VM la reproduce tick a tick.
- La puerta de la camara solo cede con todas las piezas; detras esta el botin.
- Una sola vida: diez golpes de bateria, y solo la recuperan los cuadrados
  de vida (`LIVES_ON` en el `.aru` los quita).

Falta el sonido, y mas plantas si se quiere alargar: anadir una es dibujar
otro bloque de arte ASCII en `Tools/build_rooms.py`.

## Controles

| tecla | |
|---|---|
| `O` / `P` o flechas | izquierda / derecha |
| `Z` | salto **alto**: gana altura |
| `SHIFT` o `SPACE` | salto **largo**: gana distancia |
| `D` | vuelve a encender el piloto automatico |
| `C` | banco de calibracion de saltos |
| `R` | reiniciar |

El juego arranca en piloto automatico, como el modo atraccion de un recreativo,
y **cualquier tecla de juego lo apaga**: no hay que saberse ninguna tecla antes
de sentarse a jugar. La `D` lo vuelve a encender.

## Los dos saltos son el juego

Se elige uno en el despegue y no hay control en el aire. Medido por el banco de
calibracion, a paso discreto (que no es lo mismo que la formula continua: el
punto mas alto que se llega a *ocupar* no es el vertice de la parabola, y esa
diferencia es justo la que decide si un escalon se sube):

| salto | altura | alcance |
|---|---|---|
| alto | 47,5 px | 29,25 px |
| largo | 16,5 px | 55,20 px |

Toda la torre esta medida contra esos dos numeros, con dos cotas que se repiten
y que tienen **una sola** solucion cada una:

| obstaculo | | |
|---|---|---|
| foso de 5 celdas = 40 px | largo pasa por 15,2 | alto falla por 10,8 |
| escalon de 4 celdas = 32 px | alto pasa por 15,5 | largo falla por 15,5 |

Y de ahi sale tambien el remate de cada escalera: la ultima repisa tiene que
acabar en la **fila 4**, porque desde ahi el salto alto saca al robot por el
techo (32 px) y desde la fila 8 ya no (harian falta 64).

## `Tools/verify_rooms.py`: lo que mantiene la torre en pie

Lee las constantes del propio `.aru` y comprueba tres cosas sobre los mapas ya
generados, sin abrir el juego:

1. **Que cada sala tenga solucion.** Explora a donde se llega de verdad desde
   la entrada, andando y con los seis saltos posibles, y comprueba que la
   salida de arriba, las piezas y el botin caen dentro. Una repisa dos celdas
   mas alta de la cuenta convierte una planta en un callejon sin salida.
2. **Que cada obstaculo tenga una sola solucion.** Simula el piloto y en cada
   despegue prueba *tambien* el otro salto. Si sirven los dos, avisa: ese
   obstaculo ha dejado de discriminar y el juego se queda sin mecanica.
3. **Que las plantas enganchen.** El hueco del techo de una tiene que caer
   columna a columna sobre el del suelo de la siguiente, o al caer se aterriza
   dentro de un muro.

Los arcos que mide coinciden al pixel con los que imprime el banco de dentro
del juego, que es lo que da confianza en que la simulacion es fiel.

## Compilar y ejecutar

```powershell
python Tools\build_all.py     # regenera tiles, sprites, mapas, .jpr y verifica
.\Tools\run.ps1               # compila y ejecuta
.\Tools\run.ps1 -Seconds 10   # ...y lo mata a los 10 s
.\Tools\shot.ps1              # captura la ventana
```

`build_all.py` es la unica forma de tocar los assets: **nada de `Images/`,
`Sprites/`, `Maps/` ni el `.jpr` se edita a mano**. El arte vive en
`Tools/build_tiles.py` y `Tools/build_sprites.py` como patrones de texto, y las
habitaciones en `Tools/build_rooms.py` como arte ASCII.

La salida completa de la ultima ejecucion queda en `Build\Flash\run_out.txt`
(`run.ps1` solo enseña las ultimas lineas).

## Dibujar una planta

Una sala es una lista de 24 cadenas de 40 caracteres en `Tools/build_rooms.py`.
El tile concreto lo elige el script mirando los vecinos (la coronacion de un
muro, los extremos de una repisa), asi que en el mapa solo se dice que tipo de
cosa hay:

```
.  vacio                    P  aparicion del jugador
#  muro                     u  donde apareces al llegar subiendo
_  suelo de rejilla         *  pieza
=  repisa volada            E  dron de ida y vuelta (inicio)
I  viga                     e    ...su recorrido
|  tuberia vertical         X  dron de sube y baja (inicio)
-  tuberia horizontal       x    ...su recorrido
:  rejilla de ventilacion   Q  puerta de la camara
o  lampara                  T  el botin
^  pinchos
```

El recorrido de cada bicho se dibuja en el propio mapa: asi no hay una tabla de
coordenadas aparte que se quede desincronizada al mover una repisa.

`build_rooms.py` deja en `Tools/preview_room_NN.png` la sala reconstruida desde
los assets, para poder juzgarla sin arrancar el juego.

### Dos cosas que costaron encontrar al dibujar

- **Las repisas de subida son de UNA fila, no macizas.** Un escalon de cuatro
  celdas se cuelga justo encima de la carrerilla del siguiente y el robot se da
  con su parte de abajo al subir. Con una fila, al pasar rozando solo hay 8 px
  de labio.
- **Subir usa el marcador `u`, bajar no necesita nada.** Al salir por el techo,
  aparecer en el borde de abajo de la sala nueva no vale: ese borde es el hueco
  por el que se acaba de entrar y el robot se cae otra vez. Bajando si vale,
  porque los huecos estan alineados y lo que toca es seguir cayendo.

## El piloto automatico y el verificador hacen trabajos distintos

El piloto **ensena la mecanica**: mira lo que tiene delante y elige el salto que
toca, que es la decision que toma quien juega. El verificador **demuestra que la
torre tiene solucion**, explorando todos los movimientos posibles. Conflarlos
fue un error: exigirle al piloto que resolviese la torre lo convertia en un
problema de IA que no hace falta resolver.

Del piloto, tres cosas que no son obvias:

- **"Salta cuando el muro te pare" no funciona**: pegado a la pared, la primera
  colision horizontal se come la velocidad y el robot sube en vertical sin
  avanzar. Hay que mirar 16 px por delante; y si ya esta pegado, retroceder.
- **Que el suelo se acabe no significa que haya que saltar**: al borde de una
  repisa tambien se acaba. Sin comprobar que hay sitio *pisable* al otro lado
  (solido con dos celdas libres encima, no un muro cualquiera) se tiraba al
  foso de pinchos.
- **`LOOK_LEDGE` vale lo mismo que `LOOK_WALL`, y no por casualidad**: las dos
  preguntan "?salto ya para caer encima de eso?", y la respuesta la fija el
  alcance del salto alto. Con 32 px en vez de 16 despegaba pronto y aterrizaba
  un cuarto de pixel corto del borde, una y otra vez.

## Restricciones de JARU que condicionan el diseno

- Un `.gmap` guarda **un byte por celda**: 256 tiles como maximo por tileset.
  Por eso el decorado son bloques reutilizados y no textura unica.
- Solo es sprite lo que se mueve o cambia de estado (el robot, los drones, las
  piezas, la puerta, el botin). Un tile no se puede animar ni apagar, pero cada
  fotograma de sprite es un BMP suelto en disco.
- `Bitmap.load` solo admite BMP de 16/24/32 bpp. Se escribe 24 bpp con PIL; un
  16 bpp hecho a mano no carga **y no da ningun error**.
- `draw.bitmap` es `(x, y, bitmap)`, al reves de lo que dice la documentacion.
- `draw.sprite` no ancla por el centro sino por el **pivote**, que `Sprite.load`
  fija en `width/2, height/2` del fotograma. Y `flipX` voltea alrededor de ese
  mismo pivote.
- **Los dos formatos `.spr` tienen que declarar el mismo tamano de fotograma.**
  El del editor lista regiones sobre la hoja comun y el de runtime lo lleva en
  `X.json`; si no coinciden, el juego se ve bien lanzado por linea de comandos
  (que usa el runtime) y **descolocado desde el IDE** (que usa el del editor).
  Con el pivote a la mitad de un fotograma mas grande de lo que es, el sprite
  se dibuja desplazado y ademas salta de lado al girarse. `build_sprites.py` lo
  comprueba con un assert.
- `Input.justPressed` no es fiable: los flancos se calculan a mano, una vez por
  tick. Ver el comentario en `ReadInput`.
- El parser rechaza la coma colgante antes de `]`, y un `return` sin valor solo
  se puede poner justo antes de un `end`: si le sigue un `elsif`, intenta leer
  ese `elsif` como el valor devuelto.
- La VM resuelve las rutas relativas contra el directorio del `.jxr`, asi que
  los assets se copian a `Build\Flash`. `draw.text` necesita `font\consola.ttf`
  al lado del programa, y su fuente gasta ~8,4 px por caracter: un rotulo que
  no cabe se sale de la pantalla sin avisar.
- Los binarios del repo no arrancan (son x86 con las DLL x64 del IDE al lado):
  se usan los de `C:\Program Files\JARU IDE\Compiler\`.
