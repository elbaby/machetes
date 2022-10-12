# Instalación de Caddy server

https://caddyserver.com/

https://caddyserver.com/docs/install#debian-ubuntu-raspbian

Vamos a instalar desde los [repositorios de Caddy](https://caddyserver.com/).

Obtener la clave pública con la que están firmados los paquetes y repositorios
y ponerla en **`/usr/share/keyrings/caddy-stable-archive-keyring.gpg`**.
Agregar el repositorio para la versión `stable` en un archivo 
**`/etc/apt/sources.list.d/caddy-stable.list`** 
La versión para `debian` sirve tanto para
Debian como para Ubuntu y Raspbian:

```
export NOMBRESO=debian
export VERSIONSERVER=stable
# Obtener la clave pública con la que están firmados los paquetes y repositorios
curl "https://dl.cloudsmith.io/public/caddy/${VERSIONSERVER}/gpg.key" | sudo gpg --dearmor -o /usr/share/keyrings/caddy-${VERSIONSERVER}-archive-keyring.gpg
# Bajar la configuración del repositorio (ya está configurado para buscar la firma en /usr/share/keyrings)
sudo curl --output /etc/apt/sources.list.d/caddy-${VERSIONSERVER}.list "https://dl.cloudsmith.io/public/caddy/${VERSIONSERVER}/${NOMBRESO}.deb.txt" 
```

Ejecutar los siguientes comandos para actualizar los datos de los repositorios
e instalar el servidor caddy

```
# Actualizo base de datos de paquetes
sudo apt update

# Instalo Caddy
sudo apt install caddy

# Copiar el sitio estático que sirve por default en /var/www/caddy
sudo mkdir -pv /var/www
sudo cp -rpv /usr/share/caddy /var/www
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
