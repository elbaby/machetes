# Instalación de Caddy server

https://caddyserver.com/

https://caddyserver.com/docs/install#debian-ubuntu-raspbian

Vamos a instalar desde los [repositorios de Caddy](https://caddyserver.com/).

Obtener la clave pública con la que están firmados los paquetes y repositorios
y ponerla en **`/usr/share/keyrings/caddyserver-archive-keyring.gpg`**:
```
curl https://dl.cloudsmith.io/public/caddy/stable/gpg.key | gpg --dearmor | sudo tee /usr/share/keyrings/caddyserver-archive-keyring.gpg >/dev/null
```

Agregar el repositorio para la versión `stable` en un archivo 
**`/etc/apt/sources.list.d/caddy-stable.list`** (hay que agregarle la
información de dónde está la clave con la que se firma el repositorio, ya que
no quedó en `/etc/apt/strusted.gpg.d` para evitar que esa clave se pueda usar
para firmar _otros_ repositorios). La versión para `debian` sirve tanto para
Debian como para Ubuntu y Raspbian:

```
export NOMBRESO=debian
export VERSIONSERVER=stable
curl "https://dl.cloudsmith.io/public/caddy/${VERSIONSERVER}/${NOMBRESO}.deb.txt" \
| sed -e 's#\(deb\(-src\)\?\) #\1 [arch=amd64 signed-by=/usr/share/keyrings/caddyserver-archive-keyring.gpg] #' \
| sudo tee /etc/apt/sources.list.d/caddy-${VERSIONSERVER}.list
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

# Configuraciones generales

* https://caddyserver.com/docs/getting-started
* https://caddyserver.com/docs/caddyfile
* https://caddyserver.com/docs/quick-starts/caddyfile
* https://caddyserver.com/docs/caddyfile/concepts


La configuración más simple se hace usando el archivo 
[**`/etc/caddy/Caddyfile`**](https://caddyserver.com/docs/caddyfile).

La instalación por defecto viene con un ejemplo que contesta las conexiones al 
port 80 usando la dirección IP o un nombre que no esté configurada en Caddy con
un servidor de archivos estático en `/usr/share/caddy`.

Un default más razonable para poner en un servidor que se llame 
caddy.example.net podría ser así:

```
# Global Options
{
	# El mail que mandamos al server ACME para registrar
	# los certificados 
	email   tls-ssl-master@example.net

	# El "hostname" que tomaremos por default si el cliente
	# no manda un SNI
	default_sni     caddy.example.net
	# Descomentar la línea de abajo para loguear
	#debug
}

# Usar para las conexiones por IP o con nombre desconocido
caddy.example.net,
:80 
{
	root * /var/www/caddy
	# Enable the static file server.
	file_server * {
		index	index.html
	}

}

# Si tenemos un servicio web que funciona en el mismo equipo
# en el port 12345 podemos hacer un proxy reverso hacia el mismo
# con un nombre público (que debe estar registrado en el DNS público
# para que Caddy gestione un certificado TLS gratuito en letsencrypt.org
# usando el protocolo ACME). 
# Caddy también arma una redirección automática http -> https
servicio.example.net,
www.servicio.example.net
{
	reverse_proxy localhost:12345
}
```

Para activar cambios en la configuración usar:
```
sudo systemctl reload caddy.service
```

**[TBC]**

# _Logging_

En los Ubuntu y Debian nuevos, Caddy utiliza las facilidades de _logging_ de 
`systemd`, llamada 
[`journald`](https://www.server-world.info/en/note?os=Debian_11&p=journald).

Los logs se manejan en un formato binario y se consultan con la herramienta
[**`journalctl`**](https://manpages.debian.org/systemd/journalctl.1.en.html).

Sin ningún parámetro, `journalctl` muestra todo el log del sistema dentro de
un _pager_.

En general conviene filtrar para ver los mensajes que uno quiere.

Para ver los mensajes de `caddy` se puede filtrar por el nombre de la `unit`:
```
sudo journalctl --unit caddy.service
sudo journalctl -u caddy.service
```
o filtrar por el identificador de syslog:
```
sudo journalctl --identifier caddy
sudo journalctl -t caddy
```
La ventaja de filtrar por `unit` es que se muestran también mensajes producidos
por `systemd` respecto de esa `unit` (además de los que produce la `unit` en 
sí).

También podemos buscar un _pattern_ en particular con la opción `--grep`. Si
el _pattern_ es todo en minúsculas, la búsqueda es _case insensitive_. Si se
quiere hacer una búsqueda _case sensitive_ con un _pattern_ que sólo tiene
minúsculas hay que agregar la opción `--case-sensitive=y`.
```
sudo journalctl --unit caddy.service --grep fail
sudo journalctl --unit caddy.service -g fail
```

Se puede acotar el rango de tiempo de la búsqueda usando las opciones `--since`
y `--until`:
```
sudo journalctl --unit caddy.service --since "2022-01-11 00:00:00" --until "2022-01-13 23:59:59"
sudo journalctl --unit caddy.service -S "2022-01-11 00:00:00" -U "2022-01-13 23:59:59"
```

Para "seguir" un log a medida que avanza (como si se siguiera un log de texto
utilizando el comando `tail -f`) se puede utilizar la opción `--follow`:
```
sudo journalctl --unit caddy.service --follow
sudo journalctl --unit caddy.service -f
```
alternativamente, estando dentro del _pager_, apretar la tecla `F` mayúscula y
el _pager_ irá hasta el final del log y comenzará a "seguirlo".

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
