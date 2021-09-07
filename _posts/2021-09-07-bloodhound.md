---
layout: single
title: BloodHound
excerpt: "En este artículo, explico que es BloodHound y como utilizarlo en entornos de Active Directory."
date: 2021-09-07
classes: wide
header:
  teaser: /assets/images/bloodhound/logo.png
  teaser_home_page: true
categories:
  - Windows
  - Active_Directory
---
## ¿Qué es BloodHound?

<p align="center">
  <img width="960" height="700" src="/assets/images/bloodhound/bh.png">
</p>

BloodHound es una herramienta enfocada en la enumeración de Active Directory. BloodHound ayuda a los atacantes a identificar rutas de ataque altamente complejas que de otra forma sería mas complicado visualizar. Esta herramienta es muy útil, ya que representa de forma gráfica los resultados, se utiliza tanto para "red team" como para "blue team", ya que a fin de cuentas, sirve para enumerar vectores de ataque/vulnerabilidades en un Controlador de Dominio.

<h2>Instalación de la base de datos</h2>

Para el uso de BloodHound, es necesario disponer de la base de datos neo4j. Para una instalación rápida:

```bash
apt update && apt install neo4j
``` 

Cuando esté instalada, se tiene que iniciar:

```bash
neo4j start
```

Con esto, nos abrirá un servidor http que será accesible des del navegador web. Tendremos que loggearnos y crear un nuevo usuario/contraseña.

<h2>Instalación de BloodHound</h2>

Para utilizar BloodHound, recomiendo utilizarlo descargando el binario, pero se puede instalar también:

```bash
apt update && apt install bloodhound
```
La forma de utilizarlo sin necesidad de instalarlo es descargarlo: 

* [https://github.com/BloodHoundAD/BloodHound/releases/download/4.0.3/BloodHound-linux-x64.zip](https://github.com/BloodHoundAD/BloodHound/releases/download/4.0.3/BloodHound-linux-x64.zip)

Con el siguiente comando, debería funcionar sin problemas:

```bash
./bloodhound --no-sandbox
```

Ahora que ya tenemos preparada la base de datos, tenemos que utilizar el recurso llamado SharpHound.exe, este script, en el sistema, realizará la enumeración de forma automática:

* [https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors](https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors)

```powershell
C:\Windows\Temp\test>.\SharpHound.exe -c all

----------------------------------------------

Initializing SharpHound at 14:06 on 07/09/2021

----------------------------------------------

Resolved Collection Methods: Group, Sessions, LoggedOn, Trusts, ACL, ObjectProps, LocalGroups, SPNTargets, Container

[+] Creating Schema map for domain SI4CORP.LOCAL using path CN=Schema,CN=Configuration,DC=si4corp,DC=local

[+] Cache File not Found: 0 Objects in cache

[+] Pre-populating Domain Controller SIDS

Status: 0 objects finished (+0) -- Using 18 MB RAM

Status: 62 objects finished (+62 8)/s -- Using 26 MB RAM

Enumeration finished in 00:00:00.3215783

Compressing data to .\20210907140603_BloodHound.zip

You can upload this file directly to the UI


SharpHound Enumeration Completed at 14:06 on 07/09/2021! Happy Graphing!
```

Como vemos, se ha generado el archivo "20210907140603_BloodHound.zip", este archivo lo tendremos que transferir a nuestra máquina atacante y arrastrar el archivo en bloodhound:

![](/assets/images/bloodhound/blood.png)

<h2>Como utilizar BloodHound?</h2>

Cuando haya cargado, tendremos varias opciones de listar el sistema, para este ejemplo, he puesto la que más contenido salía (shortest path to high value targets).

![](/assets/images/bloodhound/grafico.png)

En BloodHound, tendremos la posibilidad de indicarle al programa con que usuarios tenemos acceso al sistema y, a través de dicho usuario, como poder escalar privilegios, por ejemplo, si estamos en el caso que tenemos acceso a un usuario (ej: e1abrador) y queremos ver si estamos en algún grupo el cual sea vulnerable a la escalada de privilegios, lo podemos hacer de la siguiente manera:

* El primer paso sería marcar el usuario e1abrador como <b>owned</b> (click derecho en el usuario y "Mark user as Owned").

* El siguiente paso sería buscar el grupo que queremos utilizar para realizar la escalada de privilegios, en este caso, "Account Operators", y marcarlo "Mark user as Owned"

* Finalmente, buscamos el grupo objetivo, que normalmente es "Domain Admins", le daremos a click derecho y "Shortest path to here from owned"

Hecho esto, se nos abrirá un gráfico y nos indicará como explotar cada grupo para ir elevando privilegios.

Lo que comentaba en el paso anterior, BloodHound tiene un panel el cual incorpora como abusar de ese privilegio para ganar acceso como Administrador.

<p align="center">
  <img width="760" height="500" src="/assets/images/bloodhound/generic_all.png">
</p>

Salen también las referiencias para explotar esa vulnerabilidad y las referencias por si se necesita mas información sobre dicha vulnerabilidad.

<h2>Python3 Bloodhound</h2>

En caso de tener credenciales válidas a nivel de sistema pero no tener acceso directo (por ejemplo con winrm), podremos utilizar la utilidad escrita en python3 de bloodhound. Con esta utilidad, no hace falta tener acceso al sistema, simplemente, con tener credenciales válidas, se podrá realizar la enumeración.

```bash
python3 bloodhound.py -u "username" -p "password" -d e1abrador.local -c all
```

En este caso, se generarán los archivos sin estar comprimidos. Los archivos generados deberían llamarse:

* 123456_domains.json
* 123456_computers.json
* 123456_users.json
* 123456_groups.json

En este punto el proceso es igual que con el archivo .zip comentado anteriormente, la idea sería arrastrar los archivos a bloodhound y funcionaría de la misma manera.

<h2>Como limpiar la base de datos</h2>

Esta pregunta la he visto mucho por las comunidades y los posts online, es bastante sencillo, si vamos a realizar una nueva enumeración, para dejar todo limpio de datos, lo podemos hacer de la siguiente manera:

<p align="center">
  <img width="660" height="500" src="/assets/images/bloodhound/db.png">
</p>
