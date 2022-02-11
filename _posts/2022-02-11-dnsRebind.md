---
layout: single
title: DNS Rebind
excerpt: "En este artículo, explico que es el DNS Rebind y como explotarlo."
date: 2022-02-11
classes: wide
header:
  teaser: /assets/images/dnsrebind/dns.jpg
  teaser_home_page: true
categories:
  - dns
tags:
  - DNS
---

Este ataque va indirectamente vinculado al SSRF, ya que aunque son ataques completamente distintos, concatenándolo con el SSRF se puede ganar una mayor criticidad, en este ataque, se busca el fallo en el DNS para que, (explicado a alto nivel) después de X número de requests que se envíen desde X número de IPs (1 o varias), el servidor DNS nos responda con una IP distinta a la que debería responder.

Si hago esto en un SSRF:

````console
example.com/test.php?url=http://127.0.0.1:80
```` 

Aunque el puerto 80 esté abierto internamente y aunque la IP sea correcta, lo mas probable es que el WAF bloquee la request ya que "127.0.0.1" estará en la blacklist (se puede intentar bypassear de diferentes formas, pero ahora hablaremos del DNS Rebinding).
La idea de esto, es que ejecutará la request al dominio example.com desde nuestra IP, por lo que no tendremos acceso. Pues bien, con el DNS Rebinding se puede probar si el servidor es vulnerable, voy a poner un ejemplo práctico (únicamente testeando el dominio):

Si hacemos una request al dominio <b>make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms</b>:

````console
> host make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms                                   
make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms has address 169.254.169.254
Host make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms not found: 3(NXDOMAIN)
````
Sale que la IP es la 169..., esto es correcto, pero, si se hacen mas requests seguidas a este dominio, se puede intentar que salga una IP distinta:

````console
> host make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms                                   
make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms has address 169.254.169.254
Host make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms not found: 3(NXDOMAIN)
> host make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms                                   
make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms has address 169.254.169.254
Host make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms not found: 3(NXDOMAIN)
> host make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms                                   
make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms has address 169.254.169.254
Host make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms not found: 3(NXDOMAIN)
> host make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms                                   
make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms has address 169.254.169.254
Host make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms not found: 3(NXDOMAIN)
> host make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms                                   
make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms has address 1.2.3.4
Host make-1.2.3.4-rebind-169.254.169.254-rr.1u.ms not found: 3(NXDOMAIN)
````

En la última petición DNS ha cambiado la IP, pues bien, si se hace el SSRF justo cuando la IP cambia a 1.2.3.4 (o a la que corresponda), la request ya no se interpretará desde la IP del atacante sino de la del propio servidor remoto, en este caso, si tendríamos acceso a el contenido desde el navegador que estemos buscando a través del SSRF (archivos, enumeración de puertos internos/IPs internas, etc).

Sabiendo esto en caso del SSRF, la mejor opción es enviarlo al intruder para ver si, por ejemplo, después de 100 requests, el servidor responde con una longitud/código de estado distinto.

Para probar el servidor sin necesidad de utilizar el comando host X veces manualmente o probarlo a través de un SSRF, he creado un script en bash muy simple que ayuda a automatizar el testeo del DNS:

Ejecución del script:


````console
chmod +x testDnsRebind.sh
./testDnsRebind.sh google.com 10
````

````bash
#!/bin/bash

domain=$1
tests=$2

for i in $(seq 1 $tests); do
	host_ip=$(host $domain | grep address | awk '{print $NF}')
	echo -e "$host_ip";
done | sort -u; wait
```` 

Para limpiar el output, el script mostrará únicamente la IP con la que responde el servidor y 1 sola vez todas las IPs que se obtengan en el output.
