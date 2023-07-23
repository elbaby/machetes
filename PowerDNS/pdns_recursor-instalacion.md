# Instalación de PowerDNS Recursor

https://doc.powerdns.com/recursor/getting-started.html#installation

Vamos a instalar la versión 4.6.x de los [repositorios de 
PowerDNS](https://repo.powerdns.com/).

Obtener la clave pública con la que están firmados los paquetes y repositorios
y ponerla en **`/usr/share/keyrings/powerdns-archive-keyring.gpg`**:
```
curl https://repo.powerdns.com/FD380FBB-pub.asc | gpg --dearmor | sudo tee /usr/share/keyrings/powerdns-archive-keyring.gpg >/dev/null
```

Agregar el repositorio (para la versión 4.6.X del server en Debian version 11
bullsey) en un archivo **`/etc/apt/sources.list.d/pdns.list`**:
```
export NOMBRESO=debian
export VERSIONSO=bullseye
export VERSIONSERVER=46
sudo tee --append /etc/apt/sources.list.d/pdns.list <<EOF
deb [arch=amd64 signed-by=/usr/share/keyrings/powerdns-archive-keyring.gpg] https://repo.powerdns.com/${NOMBRESO} ${VERSIONSO}-rec-${VERSIONSERVER} main
EOF

```

_Pinear_ el repositorio agregando un archivo **`/etc/apt/preferences.d/pdns`**:
```
sudo tee /etc/apt/preferences.d/pdns <<EOF
Package: pdns-*
Pin: origin repo.powerdns.com
Pin-Priority: 600
EOF

```

Ejecutar los siguientes comandos para actualizar los datos de los repositorios
e instalar el servidor y utilitarios básicos de DNS:

```
# Actualizo base de datos de paquetes
sudo apt update

# Instalo pdns-recursor y utilitarios basicos para el DNS
sudo apt install pdns-recursor dnsutils
```

# Configuraciones generales

https://doc.powerdns.com/recursor/settings.html

En `/etc/powerdns/recursor.conf` está el archivo de configuración general (en
`/etc/powerdns/recursor.d/` se pueden agregar configuraciones puntuales en forma
modular).

## `local-address`

Recién instalado, el recursor sólo escucha en la interfaz local (127.0.0.1).
Normalmente, si se quiere brindar servicio a otros equipos, hay que cambiar la
configuración 
[**`local-address`**](https://doc.powerdns.com/recursor/settings.html#setting-local-address)
para que escuche en una interfaz externa. Para aceptar conexiones en todas las
interfaces, se puede configurar así:
```
local-address=0.0.0.0, ::
```

## `allow-from`

Dada la naturaleza agresiva de la internet actual, no es recomendable tener un
servidor DNS iterativo abierto a cualquier público. 

La configuración
[**`allow-from`**](https://doc.powerdns.com/recursor/settings.html#setting-allow-from)
es la que permite controlar el origen de las consultas DNS que aceptará el
recursor.

La configuración _default_ del recursor sólo permite aceptar consultas DNS desde 
direcciones "locales" a la organización que lo opera: 
* las direcciones [_loopback_](https://en.wikipedia.org/wiki/Localhost)
* las direcciones [de enlace local 
(_link local_)](https://es.wikipedia.org/wiki/Direcci%C3%B3n_de_Enlace-Local)
* las redes privadas definidas en el [RFC 
1918](https://datatracker.ietf.org/doc/html/rfc1918.html)
* las direcciones de tipo [dirección local única _(Unique Local Address - 
ULA)_](https://es.wikipedia.org/wiki/Unique_Local_Address)

Esto es, por _default_:
```
allow-from=127.0.0.0/8, 10.0.0.0/8, 100.64.0.0/10, 169.254.0.0/16, 192.168.0.0/16, 172.16.0.0/12, ::1/128, fc00::/7, fe80::/10
```

Es posible, de ser necesario, aceptar conexiones de algunas direcciones IP
públicas. Conviene hacer esto para direcciones sobre las que se tiene algún
control. Por ejemplo si, además de las direcciones ya configuradas por 
_default_, se quieren aceptar consultas de la red 192.0.2.0/24 y de la red 
2001::1:/64, se debe configurar así:
```
allow-from+=192.0.2.0/24, 2001:DB8::1:/64
```

## Reenvío (_forwarding_) de zonas

* [**`forward-zones`**](https://doc.powerdns.com/recursor/settings.html#forward-zones)
permite reenviar consultas para dominios específicos a servidores 
**autoritativos** para esos dominios (las consultas reenviadas tienen el bit 
'`recursion desired`' en `0`).  Se deben especificar las direcciones IP de los
servidores (y se puede agregar el puerto al que se consultará, utilizando 
puertos no estándar) Los nombres de dominio se separan por comas. Se pueden 
especificar múltiples servidores para cada dominio separándolos por punto y 
coma.

* [**`forward-zones-recurse`**](https://doc.powerdns.com/recursor/settings.html#forward-zones-recurse)
funciona en forma similar a `forward-zones`, pero reenvía las consultas a otros 
servidores **iterativos** (las consultas reenviadas tienen el bit '`recursion
desired`' en `1`).
```
forward-zones=example:org=203.0.113.210:5300;10.20.30.40, example.net=::1;[2001:DB8::1:3]:5300
forward-zones-recurse=google.com=8.8.8.8;8.8.4.4, debian.org=9.9.9.9
```

* [**`forward-zones-file`**](https://doc.powerdns.com/recursor/settings.html#forward-zones-file)
permite configurar el nombre de un archivo donde se especifican los reenvíos 
(_forwarding_) de zonas de a una por línea.  Si el nombre de la zona está 
prefijado con un '`+`', la misma será tratada como con `forward-zones-recurse`, 
en caso contrario, será tratada como `forward-zones`.
```
forward-zones-file=/etc/dns/forward.zones
```
El contenido del archivo `/etc/dns/forward.zones`:
```
example:org=203.0.113.210:5300;10.20.30.40
example.net=::1;[2001:DB8::1:3]:5300
+google.com=8.8.8.8;8.8.4.4
+debian.org=9.9.9.9
```


## Otras configuraciones

* [**`auth-zones`**](https://doc.powerdns.com/recursor/settings.html#auth-zones)
permite cargar zonas desde archivos en formato BIND y responder por estas zonas
como si fuese un servidor autoritativo (útil para contestar por zonas locales
como ser `.test`, `.invalid`, `.local`, etc):
```
auth-zones=empresa.local=/var/dns/zones/empresa.local, example.org=/var/dns/zones/example.org
```

* [**`export-etc-hosts`**](https://doc.powerdns.com/recursor/settings.html#export-etc-hosts)
permite exportar (y responder consultas por) los nombres de hosts y direcciones
IP que están en el archivo `/etc/hosts`.
* [**`export-etc-hosts-search-suffix`**](https://doc.powerdns.com/recursor/settings.html#export-etc-hosts-search-suffix)
permite agregar un sufijo (nombre de dominio) a los nombres que están en el
archivo `/etc/hosts` exportado con la opción `export-etc-hosts` que no tengan un
punto (**`.`**) como parte del nombre.
* [**`etc-hosts-file`**](https://doc.powerdns.com/recursor/settings.html#etc-hosts-file)
permite exportar (con `export-etc-hosts` un archivo del tipo `/etc/hosts` en 
otra ubicación).
```
export-etc-hosts=yes
export-etc-hosts-search-suffix=example.org
etc-hosts-file=/var/dns/hosts.txt
```

* [**`dnssec`**](https://doc.powerdns.com/recursor/settings.html#dnssec)
especifica el [modo en que el recursor utilizará dnssec para hacer consultas y
funcionará según las opciones existentes en las consultas recibidas de los
clientes](https://doc.powerdns.com/recursor/dnssec.html). El modo por _default_
(desde la versión 4.5.0 del recursor) es 
[`process`](https://doc.powerdns.com/recursor/dnssec.html#process).
```
dnssec=log-fail
```


# _Logging_

En los Debian nuevos, pdns utiliza las facilidades de _logging_ de `systemd`,)
llamada 
[`journald`](https://www.server-world.info/en/note?os=Debian_11&p=journald).

Los logs se manejan en un formato binario y se consultan con la herramienta
[**`journalctl`**](https://manpages.debian.org/systemd/journalctl.1.en.html).

Sin ningún parámetro, `journalctl` muestra todo el log del sistema dentro de
un _pager_.

En general conviene filtrar para ver los mensajes que uno quiere.

Para ver los mensajes de `pdns` se puede filtrar por el nombre de la `unit`:
```
sudo journalctl --unit pdns-recursor.service
sudo journalctl -u pdns-recursor.service
```
o filtrar por el identificador de syslog:
```
sudo journalctl --identifier pdns-recursor
sudo journalctl -t pdns-recursor
```
La ventaja de filtrar por `unit` es que se muestran también mensajes producidos
por `systemd` respecto de esa `unit` (además de los que produce la `unit` en 
sí).

También podemos buscar un _pattern_ en particular con la opción `--grep`. Si
el _pattern_ es todo en minúsculas, la búsqueda es _case insensitive_. Si se
quiere hacer una búsqueda _case sensitive_ con un _pattern_ que sólo tiene
minúsculas hay que agregar la opción `--case-sensitive=y`.
```
sudo journalctl --unit pdns-recursor.service --grep fail
sudo journalctl --unit pdns-recursor.service -g fail
```

Se puede acotar el rango de tiempo de la búsqueda usando las opciones `--since`
y `--until`:
```
sudo journalctl --unit pdns-recursor.service --since "2022-01-11 00:00:00" --until "2022-01-13 23:59:59"
sudo journalctl --unit pdns-recursor.service -S "2022-01-11 00:00:00" -U "2022-01-13 23:59:59"
```

Para "seguir" un log a medida que avanza (como si se siguiera un log de texto
utilizando el comando `tail -f`) se puede utilizar la opción `--follow`:
```
sudo journalctl --unit pdns-recursor.service --follow
sudo journalctl --unit pdns-recursor.service -f
```
alternativamente, estando dentro del _pager_, apretar la tecla `F` mayúscula y
el _pager_ irá hasta el final del log y comenzará a "seguirlo".

___
<!-- LICENSE -->
___
<a rel="licencia" href="https://creativecommons.org/licenses/by-sa/4.0/deed.es">
<img alt="Creative Commons License" style="border-width:0"
src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a>
<br /><br />
Este documento está licenciado en los términos de una <a rel="licencia"
href="https://creativecommons.org/licenses/by-sa/4.0/deed.es">
Licencia Atribución-CompartirIgual 4.0 Internacional de Creative Commons</a>.
<br /><br />
This document is licensed under a <a rel="license" 
href="https://creativecommons.org/licenses/by-sa/4.0/deed.en">
Creative Commons Attribution-ShareAlike 4.0 International License</a>.
<!-- END --> 
