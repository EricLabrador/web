---
layout: single
title: wp_userScan
excerpt: "En este artículo, explico la utilidad de la herramienta wp_userScan, escrita en Bash."
date: 2021-08-31
classes: wide
header:
  teaser: /assets/images/wpsUsersScan/logo.png
  teaser_home_page: true
categories:
  - script
tags:
  - wordpress
  - shell
  - enumeration

---
## ¿Qué es wps_usersScan?


**wps_usersScan** es un pequeño script en bash ideal para realizar la enumeración de usuarios en cualquier web que utilice Wordpress como CMS.

![](/assets/images/wpsUsersScan/wpsusers.png)

Para utilizar la herramienta, es tan fácil como ejecutarla:

```bash
./wps_usersScan.sh
```

Seguidamente, saldrá un pequeño input en el cual se tiene que introducir una dirección URL. El programa concaternará de forma automática la dirección proporcionada por el usuario con el lugar donde se almacenan los usuarios de forma pública. Hay un bucle para que se ejecute del 1 a 1000, en caso que se quiera seguir enumerando, se puede alterar la cifra.

<h3>Código de la herramienta</h3>

```bash
#!/bin/bash

greenColour="\e[0;32m\033[1m"
endColour="\033[0m\e[0m"
redColour="\e[0;31m\033[1m"
blueColour="\e[0;34m\033[1m"
yellowColour="\e[0;33m\033[1m"
purpleColour="\e[0;35m\033[1m"
turquoiseColour="\e[0;36m\033[1m"
grayColour="\e[0;37m\033[1m"

function banner(){
	echo -e "${grayColour}"
	echo -e "__      __ _ __   ___      _   _  ___   ___  _ __  ___ / _\  ___  __ _  _ __  " &&  sleep 0.25
	echo -e "\ \ /\ / /| '_ \ / __|    | | | |/ __| / _ \| '__|/ __|\ \  / __|/ \` ||\'_ \ "
	echo -e " \ V  V / | |_) |\__ \    | |_| |\__ \|  __/${blueColour}| |   \__ \_\ \| (__| (_| || | | | ${endColour}- Created by Eric Labrador ${blueColour}" && sleep 0.25
	echo -e "  \_/\_/  | .__/ |___/_____\__,_||___/ \___||_|   |___/\__/ \___|\__,_||_| |_|" && sleep 0.25
	echo -e "          |_|        |_____|                                                  "
	echo -e
	echo -e "${endColour}"
}


banner
echo -ne "\n\t${turquoiseColour}[*]${endColour} ${yellowColour}Url:${endColour} " && read url
for i in $(seq 1 1000); do
	users=$(curl -s "$url/wp-json/wp/v2/users/?per_page=$i&page=$i" | jq | grep "name" | tr -d ' ' | tr -d ',' | tr -d '"' | sed 's/name://g')
	for user in $users; do
		echo -e "\n\t\t${redColour}[*] - ${endColour} ${yellowColour}$user${endColour}";
	done;
done;

```
