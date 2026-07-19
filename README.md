# SINCRO — desfasaje entre dos televisores

Mide cuánto se adelanta o se atrasa un televisor respecto de otro, aunque estén
en departamentos distintos, usando el micrófono de un aparato en cada lugar.
Sirve para saber si alguien va a escuchar el grito del gol del vecino antes de
ver la jugada.

Publicado en https://pablossg85.github.io/sincro-tv/

## Cómo funciona el estudio

El **dispositivo 1** comanda. El **dispositivo 2** trabaja como esclavo.

1. Los dos televisores en el mismo canal, volumen normal, con juego corrido.
2. Cada aparato abre la página (o escanea su QR) y toca **Activar micrófono**.
   El micrófono queda abierto y escuchando, sin grabar todavía.
3. Cada aparato avisa a la sala que está escuchando. El dispositivo 1 ve el
   estado del 2 y recién entonces se le habilita **Iniciar estudio**.
4. Al iniciar, el dispositivo 1 publica una orden con un instante de arranque
   5 segundos en el futuro. Los dos aparatos hacen la cuenta regresiva y
   empiezan a grabar en ese mismo instante de reloj.
5. Graban 20 segundos, suben su fragmento y cada uno busca el del otro.
   El resultado aparece solo, igual, en las dos pantallas.
6. **Reiniciar estudio** deja todo listo para otra corrida sin volver a pedir
   permiso de micrófono.

El resultado se expresa siempre desde el dispositivo 2:
*"El dispositivo 2 está adelantado / atrasado X ms respecto del dispositivo 1."*

## Cómo se mide

Cada aparato calcula la **envolvente de energía** del audio —el dibujo del
volumen en el tiempo— en bloques de ~21 ms, en dB y normalizada. No guarda el
audio: solo esa curva, junto con la marca de tiempo de su reloj.

Al comparar, las dos curvas se llevan a una grilla común de 20 ms y se calcula
la **correlación cruzada** para todos los desplazamientos entre −3 y +3
segundos. El máximo es el desfasaje. El coeficiente en ese punto se muestra
como "confianza": arriba de 0,45 el número es fiable.

No hace falta que los dos aparatos arranquen exactamente juntos: cada fragmento
viaja con su marca de tiempo absoluta y el cálculo compensa la diferencia. En
las pruebas los aparatos arrancaron con 680 ms de diferencia sin afectar el
resultado.

### Corrección de relojes (imprescindible)

Como el desfasaje se obtiene comparando marcas de tiempo tomadas por dos
aparatos distintos, cualquier diferencia entre sus relojes se suma entera al
resultado. Y los relojes de aparatos de consumo **no** están finamente
sincronizados: en las pruebas, la PC del proyecto estaba 4 segundos corrida
respecto del servidor.

Por eso cada aparato mide su propio desvío contra un reloj común —la función
`hora_servidor()` en Supabase— con el método de NTP: marca la hora antes y
después del pedido, supone latencia simétrica y se queda con la muestra de ida
y vuelta más corta. Todo lo que cruza al otro aparato (el instante de arranque
y el `t0` del fragmento) viaja expresado en hora de servidor.

El panel de resultado muestra el desvío detectado entre los dos relojes, ya
descontado, para poder auditarlo.

Síntoma de que esto falta: con los dos aparatos frente al **mismo** televisor
el resultado da un valor grande en vez de cero. Ese fue el bug que motivó la
corrección; el valor medido era exactamente la diferencia entre relojes.

## Precisión

- Resolución de la grilla: 20 ms.
- Con fragmentos sintéticos limpios: desfasaje sembrado de 1200 ms medido como
  1,20 s con correlación 1,00; también verificado en 750, 1500 y 2600 ms.
- Con audio real capturado por micrófono en el entorno de prueba: dos corridas
  de un desfasaje sembrado de 400 ms dieron 360 y 260 ms. La dirección y el
  orden de magnitud siempre fueron correctos, pero **por debajo de ~100 ms el
  valor es aproximado**.
- Prueba de control con relojes desfasados a propósito 2760 ms y los dos
  aparatos frente al mismo televisor: resultado 0 ms, confianza 0,94, y el
  desvío entre relojes detectado como 2759 ms.
- Fuentes de error no corregidas: latencia de captura del micrófono de cada
  aparato y el error residual de la sincronización de relojes (del orden de la
  mitad del tiempo de ida y vuelta). Ambas muy por debajo del umbral de
  1 segundo.

**Control rápido antes de confiar en una medición:** poner los dos aparatos
frente al mismo televisor. Tiene que dar cerca de 0 ms con confianza alta.

## Infraestructura

- Página estática servida por GitHub Pages. El micrófono exige HTTPS: abierta
  como archivo local no funciona.
- Señalización y traspaso de fragmentos: tabla `sincro_eventos` en el proyecto
  Supabase `fendy-boleteria`. Es una tabla aislada, sin relación con las tablas
  de la boletería. Solo permite lectura anónima de las últimas 2 horas.
