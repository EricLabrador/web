---
layout: single
title: subdomainDump
excerpt: "En este artículo, explico la utilidad de la herramienta subdomainDump.sh, escrita en Bash."
date: 2021-10-17
classes: wide
header:
  teaser: /assets/images/subdomain/logo.png
  teaser_home_page: true
categories:
  - script
tags:
  - API KEY
  - web
  - Bug Bounty
---
<p align="center">
  <img width="960" height="700" src="/assets/images/subdomain/securitytrails.png">
</p>

## ¿Qué es subdomainDump?

**subdomainDump** es una herramienta escrita en bash. Este script automatiza la obtención de subdominios a través de la web [https://securitytrails.com/](https://securitytrails.com/). En lugar de arrastrar las cookies de sesión, dado que es mas sencillo con la API, simplemente utilizaremos dicha API para gestionar todas las requests. Es una web muy útil en Bug Bounty, sobretodo en la parte de Reconocimiento en los dominios "scope" que correspondan.

Para utilizar la herramienta, el primer paso es añadir el API KEY de la cuenta del usuario. Debería quedar de la siguiente forma:

````console
API_KEY="'apikey:rWxW89Sbee...'"
````

La API KEY se puede obtener a través del siguiente enlace:

* [https://securitytrails.com/app/account/credentials](https://securitytrails.com/app/account/credentials)

A continuación, ya se puede ejecutar el script:

````bash
bash subdomainDump.sh
````

El script solicitará un dominio, para realizar una prueba se puede utilizar <b>google.es</b>.

Código del script:

````bash
#!/bin/bash

API_KEY="'apikey:<api key here>'"

echo -ne "\t[*] Domain: " && read domain
#Example: google.es


first_url="Y3VybCAtcyAtLXJlcXVlc3QgR0VUIC0tdXJsIGh0dHBzOi8vYXBpLnNlY3VyaXR5dHJhaWxzLmNvbS92MS9kb21haW4vCg=="
base64_domain=$(echo -e "$domain" | base64)
subdomains=$(echo -e "/subdomains --header " | base64)
api_key_base64=$(echo $API_KEY | base64)
third_url="fCB0ciAnXG4nICcgJyB8IHRyIC1kICcgJyB8IGdyZXAgLW9QICJcWy4qP1xdIiB8IHRyIC1kICdbXScgfHRyICcsJyAnXG4nIHwgdHIgLWQgJyIn"
command=$(echo -e "$first_url$base64_domain$subdomains$api_key_base64$third_url")


#execute command + output
echo -e "$command" | base64 -d > /tmp/sec.txt
all_domains=$(cat /tmp/sec.txt | tr -d '\n' | bash)
for line in $(echo -e $all_domains); do
	echo -e "$line.$domain ";
done

rm -r /tmp/sec.txt

````
