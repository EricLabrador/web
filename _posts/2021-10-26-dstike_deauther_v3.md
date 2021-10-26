---
layout: single
title: DSTIKE deauther V3
excerpt: "En este artículo, explico la utilidad del aparato DSTIKE Deauther v3 y todas sus funciones, también dejo el enlace de compra."
date: 2021-10-26
classes: wide
header:
  teaser_home_page: true
categories:
  - script
tags:
  - hardware hacking
  - wireless
---

## ¿Qué es DSTIKE Deauther?

**DSTIKE Deauther** es una herramienta, entre otras, que sale en la serie de Mr Robot, este dispositivo se utiliza para bloquear y denegar conexiones de redes wifi.

Para acceder a la web de compra, pulsa [aquí](https://www.amazon.es/Seamuing-desarrollo-programable-incorporada-impresi%C3%B3n/dp/B08XJ8W9TT/ref=sr_1_1?__mk_es_ES=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=dstike&qid=1635271572&qsid=261-8620237-3578240&sr=8-1&sres=B08XJ8W9TT%2CB089R5XQ87%2CB082TNGSRN%2CB09BLSWT2G%2CB08RRVBZKM%2CB092ST6LQW%2CB088LV6ZBG%2CB092R63HXC%2CB092TPJDY1%2CB09377CQ7P%2CB08TX3451X%2CB06Y1RVHBP%2CB09HG9CC7B%2CB07DKD79Y9%2CB06XFTLN43%2CB078J7LDLY%2CB06Y1LZLLY%2CB07QPN951P%2CB06Y1ZPNMS%2CB09573BPY8)

<h1>Vista rápida del dispositivo</h1>

<p align="center">
  <img width="350" height="500" src="/assets/images/dstike/partes.png">
</p>

* 1 - Este botón, en caso de pulsarse 3 segundos, se encienden unas luces a modo de linterna, para apagarlo, se tiene que mantener pulsado otros 3 segundos.
* 2 - En esta parte, hay un láser.
* 3 - Esta es la parte donde se puede flashear el firmware del dispositivo, tiene un puerto USB y si lo conectamos a nuestra máquina Linux, siguiendo esta guía se puede hacer sin problema: [Falshear Firmware](https://github.com/SpacehuhnTech/esp8266_deauther/wiki/Installation#flashing-the-firmware-bin-file)
* 4 - En caso de pulsar este botón, se restablecerá el dispositivo a los valores de fábrica.
* 5 - Es la pantalla donde se puede elegir lo que se quiere hacer con el DSTIKE

En el lateral derecho, hay una rueda, con la cual se puede seleccionar lo que queremos hacer, lo veremos en el siguiente apartado del artículo.
En el lateral izquierdo, se puede ver un puerto micro-usb y la batería del dispositivo, la batería se reparte en 4 pequeñas rayas, cada raya corresponde a un 25% de la batería.

<h1>Opciones del dispositivo</h1>

En el apartado de "<b>SCAN</b>", se pueden encontrar las siguientes opciones (Hay que tener en cuenta que este dispositivo, por ahora, solo puede escanear redes 2.4G):

- Escaneo de Puntos de Acceso y clientes conectados a la red [<b>SCAN AP + ST</b>]
- Escaneo únicamene de Puntos de Acceso [<b>SCAN APs</b>]
- Escaneo únicamente de clientes conectados a la red [<b>SCAN Stations</b>]
<p align="center">
  <img width="350" height="500" src="/assets/images/dstike/scan_options.jpg">
</p>

En el apartado de "<b>SELECT</b>", se puede seleccionar sobre que item router/cliente que se ha obtenido en la fase de escaneo se quiere realizar un ataque.

- Selección de Puntos de Acceso [<b>APs</b>]
- Selección de Clientes [<b>Stations</b>]
- <b>Names</b> (He estado investigando y no he encontrado el uso de esta opción, después de realizar varios escaneos esta opción sigue a 0.)
- <b>SSIDs</b> (El uso de esta opción es igual a la primera de este apartado, he hecho pruebas y he creado redes con el DSTIKE y en caso de crearlo si que salen para seleccionar.)

Únicamente con las 2 primeras opciones es suficiente para efectuar ataques, también en esta versión de DSTIKE está la opción de seleccionar todos los clientes y todos los puntos de acceso que se hayan escaneado.

<p align="center">
  <img width="350" height="500" src="/assets/images/dstike/scan_options_1.jpg">
</p>

En el apartado de "<b>ATTACK</b>", se puede elegir que tipo de ataque se va a efectuar.

* Ataque de deautenticación (En caso que no se seleccione ningún cliente, se utilizará la MAC Broadcast address -> FF:FF:FF:FF:FF:FF) [<b>DEAUTH</b>]
* Beacon Flood Attack , este ataque consiste en crear diferentes puntos de acceso en un canal en concreto para saturar dicho canal y hacer que o bien la conexión sea lenta o bien que se desconecten los clientes. [<b>BEACON</b>]
* Esta opción se supone que es para generar paquetes probe request/response, pero para dengar una conexión, con los 2 anteriores es suficinete, no le veo el uso a esta opción [<b>PROBE</b>]

<p align="center">
  <img width="350" height="500" src="/assets/images/dstike/attack_mode.jpg">
</p>

En el apartado "<b>PACKET MONITOR</b>" se pueden escanear los distintos canales, del 1 al 14, para ver si en la zona que se esta escaneando hay tráfico wifi o no. Hay que tener en cuena que DSTIKE no hace channel hopping de forma automática, si giramos la rueda, se cambiará de canal.

<p align="center">
  <img width="350" height="500" src="/assets/images/dstike/packet_monitor.jpg">
</p>

En el apartado "<b>CLOCK</b>" se puede cambiar la hora y poner el modo reloj, para que se vea la hora.

<p align="center">
  <img width="350" height="500" src="/assets/images/dstike/clock.jpg">
</p>

Finamente, en el apartado "<b>LED</b>", en esta versión del DSTIKE, simplemente es un láser, según el vendedor, llega entre 50 y 100 metros.

Por último a comentar sobre el DSTIKE, hay que decir que crea una red de forma automática, esta red se llama <b>pwned</b>, la contraseña por defecto es <b>deauther</b>, aunque se puede cambiar. Al conectarse desde otro dispositovo, si se accede al navegador y se busca google.es, debería redirigir de forma automática a la web del reloj, en esta web, se puede gestionar todo lo comentado anteriormente des del otro terminal. Adicionalmente, únicamente desde la web, se puede cambiar la MAC (cosa que no se puede desde el reloj), recomiendo cambiar la MAC como buena práctica, (con el comando <b>mcchanger -l</b> hay una larga lista de MAC's, recomiendo utilizar una de ese listado).
