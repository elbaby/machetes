# Instalación de PowerDNS WebUI

En https://github.com/PowerDNS/pdns/wiki/WebFrontends hay varios frontends web
para administrar un servidor autoritativo PowerDNS.

El más completo parece ser [PowerDNS
Admin](https://github.com/ngoduykhanh/PowerDNS-Admin), escrito en Python y JS y
se comunica con el server a través de la API de pdns (no escribe directamente en
la base de datos), pero, a mediados de 2021, el creador ha dejado de 
mantenerlo activamente y está buscando quién lo mantenga.
Eso, sumado a que no hay instrucciones claras para instalarlo con PostgreSQL
hace que por ahora dejemos las [instrucciones](pdns-admin-instalacion.md) a
medio camino.


Otra UI interesante es [PowerDNS 
WebUI](https://github.com/james-stevens/powerdns-webui).

Esto es una **única página** escrita en HTML/CSS/JavaScript que permite ver y
modificar los datos en la base de PowerDNS utilizando _exclusivamente_ la [API
REST](https://doc.powerdns.com/authoritative/http-api/index.html).

Esta es una herramienta **_exclusivamente_** para _sysadmins_ y no tiene
mayores controles de seguridad. Para autenticarte usás directamente la API-KEY
que definís en el servidor PowerDNS, con lo cual no hay control de usuarios
ni se pueden delegar algunos dominios a un usuario y otros a otro. Ver la
sección sobre 
**[seguridad](https://github.com/james-stevens/powerdns-webui#security)** al 
respecto.

Según la documentación, está pensado para un servidor DNS Master, pero debería
funcionar con zona nativas también.

Hay 
[ejemplos](https://github.com/james-stevens/powerdns-webui/tree/master/example) 
de configuración usando **apache** y **ngnix**,pero vamos a intentar usar
**[caddy](https://caddyserver.com/)** que es un solo ejecutable en go y es
especialmente simple para configurar proxies reversos con https automático 
utilizando certificados de **letsencrypt**.

## Instalación de caddy server

Configuramos el repositorio oficial de
[caddy](https://caddyserver.com/docs/install#debian-ubuntu-raspbian), las
claves públicas de firmado de los paquetes y lo instalmos:
```
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo tee /etc/apt/trusted.gpg.d/caddy-stable.asc
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

## Configuración de la API REST de PowerDNS

https://doc.powerdns.com/authoritative/http-api/index.html

Configuramos los siguientes parámetros en un archivo de configuración
**`/etc/powerdns/pdns.d/http-server-api.conf`**:
* [api](https://doc.powerdns.com/authoritative/settings.html#setting-api)
* [api-key](https://doc.powerdns.com/authoritative/settings.html#setting-api-key)
* [webserver](https://doc.powerdns.com/authoritative/settings.html#webserver)
* [webserver-address](https://doc.powerdns.com/authoritative/settings.html#webserver-address)
* [webserver-port](https://doc.powerdns.com/authoritative/settings.html#webserver-port)
* [webserver-allow-from](https://doc.powerdns.com/authoritative/settings.html#webserver-allow-from)
* [webserver-password](https://doc.powerdns.com/authoritative/settings.html#webserver-password)

```
sudo tee /etc/powerdns/pdns.d/http-server-api.conf <<EOF
api=yes
api-key=UnaClaveMuyMuySecretaParaLaAPI
webserver=yes
webserver-address=127.0.53.80
webserver-port=5380
webserver-allow-from=127.0.0.0/8,::1/128
webserver-password=OtraClaveMuyMuySecretaParaElWebServer
EOF
```

Ajustamos los permisos del archivo de configuración:

```
sudo chmod 0640 /etc/powerdns/pdns.d/http-server-api.conf
sudo chgrp pdns /etc/powerdns/pdns.d/http-server-api.conf
```

**IMPORTANTE**: ¡Cambiar los valores de las claves en `api-key` y 
`webserver-password` en el archivo `/etc/powerdns/pdns.d/http-server-api.conf`!

Y después reiniciamos el servidor powerdns:
```
sudo systemctl restart pdns.service
```

## Instalación de pdns-webui

La webui consiste en realidad en un solo archivo html. De todos modos, clonamos 
el repositorio y lo copiamos desde allí a /opt/powerdns-webui/htdocs

```
git clone https://github.com/james-stevens/powerdns-webui.git ${HOME}/powerdns-webui
sudo mkdir -pv /opt/powerdns-webui/htdocs
sudo cp ${HOME}/powerdns-webui/htdocs/index.html /opt/powerdns-webui/htdocs

```

___
<!-- LICENSE -->
___
<a rel="licencia" href="http://creativecommons.org/licenses/by-sa/4.0/deed.es">
<img alt="Creative Commons License" style="border-width:0"
src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a>
<br /><br />
Este documento está licenciado en los términos de una <a rel="licencia"
href="http://creativecommons.org/licenses/by-sa/4.0/deed.es">
Licencia Atribución-CompartirIgual 4.0 Internacional de Creative Commons</a>.
<br /><br />
This document is licensed under a <a rel="license" 
href="http://creativecommons.org/licenses/by-sa/4.0/deed.en">
Creative Commons Attribution-ShareAlike 4.0 International License</a>.
<!-- END --> 
