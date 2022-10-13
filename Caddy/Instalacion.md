# Instalación de Caddy server

https://caddyserver.com/

https://caddyserver.com/docs/install#debian-ubuntu-raspbian

Vamos a instalar desde los [repositorios de Caddy en 
cloudsmith](https://cloudsmith.io/~caddy/repos/).

Obtener la clave pública con la que están firmados los paquetes y repositorios
y ponerla en **`/usr/share/keyrings/caddy-stable-archive-keyring.gpg`**.
Agregar el repositorio para la versión `stable` en un archivo 
**`/etc/apt/sources.list.d/caddy-stable.list`** 
La versión para `debian` sirve tanto para
Debian como para Ubuntu y Raspbian:

```
export NOMBRESO=debian
export VERSIONSO=bullseye
export VERSIONSERVER=stable
# Obtener la clave pública con la que están firmados los paquetes y repositorios
curl "https://dl.cloudsmith.io/public/caddy/${VERSIONSERVER}/gpg.key" | sudo gpg --dearmor -o /usr/share/keyrings/caddy-${VERSIONSERVER}-archive-keyring.gpg
# Bajar la configuración del repositorio (ya está configurado para buscar la firma en /usr/share/keyrings)
sudo curl --output /etc/apt/sources.list.d/caddy-${VERSIONSERVER}.list "https://dl.cloudsmith.io/public/caddy/${VERSIONSERVER}/config.deb.txt?distro=${NOMBRESO}&codename=${VERSIONSO}"
```

Ejecutar los siguientes comandos para actualizar los datos de los repositorios
e instalar el servidor caddy

```
# Actualizar base de datos de paquetes
sudo apt update

# Instalar Caddy
sudo apt install caddy

# Copiar el sitio estático que sirve por default en /var/www/caddy
sudo mkdir -pv /var/www
sudo cp -rpv /usr/share/caddy /var/www
```

# Instalación desde fuentes para poder incorporar módulos

https://caddyserver.com/docs/build

## Instalar Go

Según la página hace falta Go 1.17 o más nuevo. Pero Debian 11 tiene Go 1.15.

Para poder instalar una versión más nueva (sin compilar) hay que usar 
`bullseye-backports`:

```
sudo apt install -t bullseye-backports golang
```
## Instalar xcaddy

Vamos a instalar desde los [repositorios de xcaddy en 
cloudsmith](https://cloudsmith.io/~caddy/repos/).

Obtener la clave pública con la que están firmados los paquetes y repositorios
y ponerla en **`/usr/share/keyrings/caddy-xcaddy-archive-keyring.gpg`**.
Agregar el repositorio en un archivo 
**`/etc/apt/sources.list.d/caddy-xcaddy.list`** 

```
export NOMBRESO=debian
export VERSIONSO=bullseye
export VERSIONSERVER=xcaddy
# Obtener la clave pública con la que están firmados los paquetes y repositorios
curl "https://dl.cloudsmith.io/public/caddy/${VERSIONSERVER}/gpg.key" | sudo gpg --dearmor -o /usr/share/keyrings/caddy-${VERSIONSERVER}-archive-keyring.gpg
# Bajar la configuración del repositorio (ya está configurado para buscar la firma en /usr/share/keyrings)
sudo curl --output /etc/apt/sources.list.d/caddy-${VERSIONSERVER}.list "https://dl.cloudsmith.io/public/caddy/${VERSIONSERVER}/config.deb.txt?distro=${NOMBRESO}&codename=${VERSIONSO}"
```

Ejecutar los siguientes comandos para actualizar los datos de los repositorios
e instalar el xcaddy

```
# Actualizar base de datos de paquetes
sudo apt update

# Instalar xcaddy
sudo apt install xcaddy
```

## Compilar caddy

Compilación simple:
```
xcaddy build
```

Compilación agregando módulos:
```
xcaddy build --with github.com/caddyserver/transform-encoder \
	--with github.com/caddyserver/nginx-adapter \
	--with github.com/caddyserver/ntlm-transport@v0.1.1
```

Esto dejó el ejecutable **`caddy`** en el directorio donde se hizo la
compilación.

Se lo puede poner en cualquier lugar del path.

## Usar la versión compilada junto con el paquete `caddy` de Debian

https://caddyserver.com/docs/build#package-support-files-for-custom-builds-for-debianubunturaspbian

Para aprovechar todo el entorno y archivos de soporte (e.g. scripts de systemd,
etc), se puede activar el uso del ejecutable compilado dentro del entorno del
paquete.

Suponiendo que el ejecutable compilado `caddy` está en el directorio actual:

```
# Mover el binario original a /usr/bin/caddy.default
sudo dpkg-divert --divert /usr/bin/caddy.default --rename /usr/bin/caddy
# Mover el binario compilado a /usr/bin/caddy.custom
sudo mv ./caddy /usr/bin/caddy.custom
# Configurar el binario original como alternativa con baja prioridad (10)
sudo update-alternatives --install /usr/bin/caddy caddy /usr/bin/caddy.default 10
# Configurar el binario compilado como alternativa con alta prioridad (50)
sudo update-alternatives --install /usr/bin/caddy caddy /usr/bin/caddy.custom 50
```

Si más adelante se desea volver al original se puede reconfigurar usando

```
sudo update-alternatives --config caddy
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
