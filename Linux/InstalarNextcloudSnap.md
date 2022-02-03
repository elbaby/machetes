# Instalar nextcloud con el paquete snap

* https://snapcraft.io/nextcloud
* https://github.com/nextcloud-snap/nextcloud-snap
* https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-nextcloud-on-ubuntu-20-04
* https://www.frankindev.com/2019/12/05/setting-up-snap-nextcloud-on-ubuntu/

Instalar primero [Caddy](../Caddy) para atender las conexiones web desde fuera
del server y gestionar los certificados TLS.

Vamos a instalar el [snap de Nextlcoud Hub desde
snapcraft](https://snapcraft.io/nextcloud). Los fuentes están en github en
[nextcloud-snap/nextcloud-snap](https://github.com/nextcloud-snap/nextcloud-snap).

Si no está instalado `snapd` hay que instalarlo:
```
sudo apt install snapd
```

Luego, usamos snap para instalar nextcloud (las dependencias que hagan falta se
instalarán automáticamente):
```
sudo snap install nextcloud --channel=latest/stable
```

El snap incluye un servidor web que, por default, escucha en el port 80 y en el
443 (que están siendo utilizado por Caddy). Por _default_ https está 
desactivado.

Para cambiar los ports que usa:
```
sudo snap set nextcloud ports.http=4001 ports.https=4002
```

Para crear el usuario administrador por línea de comandos (en lugar de hacerlo
desde un navegador):
```
sudo nextcloud.manual-install admin <PONER ACA UNA BUENA CLAVE>
```

Por default, el único nombre de dominio que acepta nextcloud es **localhost**.
Se puede ver con:
```
sudo nextcloud.occ config:system:get trusted_domains
```

Hay que agregar los nombres que se van a utilizar, del siguiente modo:
```
sudo nextcloud.occ config:system:set trusted_domains 1 --value=nextcloud.example.net
sudo nextcloud.occ config:system:set trusted_domains 2 --value=nc.example.net
```

Podemos ver todos los nombres configurados con:
```
sudo nextcloud.occ config:system:get trusted_domains
```

Ahora, en [Caddy](../Caddy) agregamos lo siguiente en el archivo de
configuración `/etc/caddy/Caddyfile`:
```
nextcloud.example.net,
nc.example.net
{
	rewrite	/.well-known/cardav /remote.php/dav
	rewrite	/.well-known/caldav /remote.php/dav
	reverse_proxy localhost:4001
}
```

Y cargamos la nueva configuración del caddy:
```
sudo systemctl reload caddy.service
```

Y ahora [configuramos el proxy en 
nextcloud](https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/reverse_proxy_configuration.html) 
para que maneje los encabezados recibidos del cliente y escriba los URL 
correctamente para que se vean desde "afuera" del proxy:
```
sudo nextcloud.occ config:system:set trusted_proxies 0 --value='localhost'
sudo nextcloud.occ config:system:set overwritehost --value='nc.example.net'
sudo nextcloud.occ config:system:set overwriteprotocol --value='https'
```

A partir de ahora, se puede ingresar en un navegador usando el URL 
http://nextcloud.example.net (que será automáticamente redireccionado por Caddy
a https://nextcloud.example.net y luego enviado vía reverse proxy a
localhost:4001 donde está funcionando el nextcloud)

El usuario **admin** que creamos más arriba se puede utilizar para administrar
la plataforma vía web.

También se pueden utilizar el comando `nextcloud.occ` en el servidor. Para ver
una lista de los comandos disponibles:
```
sudo nextcloud.occ list
```

Para crear un usuario y luego agregarlo al grupo admin:
```
# user:add va a pedir la clave para ponerle al usuario
sudo nextcloud.occ user:add carlitos
sudo nextcloud.occ group:adduser admin carlitos
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
