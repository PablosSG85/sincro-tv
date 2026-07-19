# SINCRO — ¿están sincronizados los dos televisores?

Mide el desfasaje entre dos televisores que están en lugares distintos, usando
el micrófono de un celular en cada lugar. Sirve para saber si alguien va a
escuchar el grito del gol del vecino antes de ver la jugada.

Página única, sin dependencias, sin servidor: `index.html`.

## Cómo se usa

1. Los dos televisores en el mismo canal, con volumen normal.
2. Cada persona abre la página en su aparato y elige **TV 1** o **TV 2**.
3. Se coordinan por teléfono y aprietan **Grabar** a la vez. Alcanza con
   arrancar dentro de los mismos 8 segundos: no hace falta precisión.
4. Cada aparato devuelve un código corto. Se lo pasan por mensaje.
5. Se pegan los dos códigos en la pestaña **Comparar** y sale el desfasaje.

Conviene grabar con juego corrido, no en el entretiempo: el método necesita
audio con variación (relato, público). Con audio plano la confianza baja, y la
página lo avisa.

## Cómo funciona

Cada aparato graba 20 segundos y calcula la **envolvente de energía** del audio
—el dibujo del volumen en el tiempo— en bloques de ~21 ms, en dB y normalizada.
No guarda el audio: solo esa curva, que va comprimida en el código junto con la
marca de tiempo del reloj del aparato.

Al comparar, las dos curvas se llevan a una grilla común de 20 ms y se calcula
la **correlación cruzada** para todos los desplazamientos entre −3 y +3
segundos. El desplazamiento donde la correlación es máxima es el desfasaje
entre los dos televisores. El coeficiente de correlación en ese punto se
muestra como "confianza".

### Por qué no necesita servidor ni sincronizar relojes

Los relojes del iPhone y de un Android ya están sincronizados por NTP contra
los servidores de Apple y de Google, con un error típico de decenas de
milisegundos. Como el umbral que importa acá es de **1 segundo**, ese error es
unas veinte veces menor que la magnitud a medir, y se puede ignorar.

Si en algún momento se quisiera precisión de milisegundos, habría que sincronizar
los relojes midiendo el tiempo de ida y vuelta contra un servidor común.

### Precisión

- Resolución de la grilla: **20 ms**.
- Verificado con audio sintético de desfasaje conocido: pidiendo 750 ms midió
  740; con 2600 midió 2600; con 1500 al revés midió 1500 y atribuyó
  correctamente qué televisor iba adelantado; sincronizados dio 0,00.
- Fuentes de error no corregidas: latencia de captura del micrófono de cada
  aparato (~10-40 ms) y deriva entre los relojes. Ambas muy por debajo del
  umbral de 1 segundo.

## Requisitos

El navegador solo da acceso al micrófono en un contexto seguro, así que la
página **tiene que servirse por HTTPS** (GitHub Pages sirve). Abierta como
archivo local desde el celular, el micrófono queda bloqueado.
