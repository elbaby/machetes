# Instalación de PowerDNS Authoritative Server

https://doc.powerdns.com/authoritative/installation.html

https://doc.powerdns.com/authoritative/guides/basic-database.html


La versión de PowerDNS Authoritative Server en los repos de Debian es medio
vieja (en 2021-08, con la versión 4.5.0 en la calle, los repos de Buster iban
por la 4.1.6), con lo cual vamos a instalarla desde los
[repositorios de PowerDNS](https://repo.powerdns.com/).

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
deb [arch=amd64 signed-by=/usr/share/keyrings/powerdns-archive-keyring.gpg] http://repo.powerdns.com/${NOMBRESO} ${VERSIONSO}-auth-${VERSIONSERVER} main
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

# Instalo pdns-server y utilitarios basicos para el DNS
sudo apt install pdns-server dnsutils
```

# Configuraciones generales

https://doc.powerdns.com/authoritative/settings.html

En `/etc/powerdns/pdns.conf` está el archivo de configuración general (en
`/etc/powerdns/pdns.d/` se pueden agregar configuraciones puntuales en forma
modular).

Algunos _defaults_ que puede ser interesante cambiar:

* [**`default-soa-content`**](https://doc.powerdns.com/authoritative/settings.html#default-soa-content)
es el valor que tendrá el registro SOA cuando se crea una zona. 
Conviene configurar aquí el nombre del servidor primario público que usarán las
nuevas zonas y la dirección de correo de contacto. Por ejemplo:
```
default-soa-content=ns1.example.com hostmaster.example.com 0 10800 3600 604800 3600
```

* [**`edns-cookie-secret`**](https://doc.powerdns.com/authoritative/settings.html#edns-cookie-secret)
**debe** ser una cadena de _exactamente_ 32 caracteres hexadecimales. Cuando
está configurada, se utiliza para responder con [_Cookies_ 
EDNS](https://www.rfc-editor.org/rfc/rfc9018.html) a consultas que tiene la
opción Cookie EDNS0.

* [**`server-id`**](https://doc.powerdns.com/authoritative/settings.html#server-id)
es el string que devolverá si se consulta por la opción [NSID (_Name Server 
Identifier_)](https://www.rfc-editor.org/rfc/rfc5001.html) de EDNS (por ejemplo, 
con la opción `+nsid` de `dig`). Por default, el servidor contesta el `hostname`, 
lo que podría exponer nombres privados. Opcionalmente, se puede configurar como
`disabled` y el servidor no contestará por la opción NSID.

* [**`version-string`**](https://doc.powerdns.com/authoritative/settings.html#version-string)
es el string que contesta cuando se consulta la versión del servidor a través de
DNS (`dig chaos txt version.bind @server`). Las opciones son:
  * `full` (_default_) muestra el nombre y la versión de PowerDNS
  * `powerdns` sólo muestra el nombre (Served by PowerDNS - 
https://www.powerdns.com)
  * `anonymous` devuelve un error `ServFail` sin ninguna información
  * alternativamente, se puede poner un string cualquiera que será devuelto
ante la consulta

* [**`xfr-cycle-interval`**](https://doc.powerdns.com/authoritative/settings.html#xfr-cycle-interval)
en el primario, es el intervalo (en segundos) que se deja pasar para verificar
si el serial del SOA se incrementó y hay que enviar `NOTIFY`s a los secundarios.
En los secundarios es el tiempo en segundos para chequear actualizaciones a las
zonas. 
El _default_ es 60 (1 minuto). En servidores con mucha carga y/o muchas zonas, 
puede ser eficiente alargar este período. En servidores tranquilos, si se 
quiere que los secundarios actualicen más rápido, se puede acortar a 10
segundos, por ejemplo.

# Instalación y configuración de backends

PowerDNS tiene varios [backends
distintos](https://doc.powerdns.com/authoritative/backends/index.html) (algunos
para usos muy específicos, otros de uso general).

Por default, sólo está habilitado el [backend para archivos de zona tipo 
BIND](https://doc.powerdns.com/authoritative/backends/bind.html).

Hay distintos backends que utilizan bases de datos SQL. El [backend genérico 
SQL](https://doc.powerdns.com/authoritative/backends/generic-sql.html)
describe la funcionalidad común a todos ellos.

* [Instrucciones para instalar y configurar](pdns_server-be-postgresql.md) el 
[backend genérico 
PostgreSQL](https://doc.powerdns.com/authoritative/backends/generic-postgresql.html)
* [Instrucciones para instalar y configurar](pdns_server-be-sqlite3.md) el 
[backend genérico SQLite 
3](https://doc.powerdns.com/authoritative/backends/generic-sqlite3.html)

# Pruebas básicas

Para probar que el servicio está levantado y funcionando hacemos una consulta 
cualquiera:
```
dig www.example.com a @127.0.0.1
```

El resultado debería ser algo parecido a:
```
; <<>> DiG 9.11.5-P4-5.1+deb10u5-Debian <<>> www.example.com a @127.0.0.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: REFUSED, id: 58643
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;www.example.com.		IN	A

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Fri Oct 15 21:11:10 -03 2021
;; MSG SIZE  rcvd: 44

```
Razonablemente, el estado es **`REFUSED`** ya que el servidor no tiene
configurado el dominio example.com (ni ningún otro), pero sí respondió 
(demostrando que está levantado).

**Nota**: Para hacer pruebas de DNS siempre es recomendable usar 
**[dig](https://manpages.debian.org/bullseye/bind9-dnsutils/dig.1.en.html)** 
(o [drill](https://manpages.debian.org/bullseye/ldnsutils/drill.1.en.html)). 
Nunca conviene usar 
[host](https://manpages.debian.org/bullseye/bind9-host/host.1.en.html) o 
[nslookup](https://manpages.debian.org/bullseye/bind9-dnsutils/nslookup.1.en.html).

Para hacer un par de pruebas usamos el comando 
`[pdnsutil](https://doc.powerdns.com/authoritative/manpages/pdnsutil.1.html#zone-manipulation-commands)`
que nos permite manipular zonas, registros y claves (entre otras cosas).

Creamos una zona example.com con un registro NS:
```
$ sudo --user=pdns pdnsutil create-zone example.com a.example.com
Creating empty zone 'example.com'
Also adding one NS record
```
Agrego un registro MX:
```
$ sudo --user=pdns pdnsutil add-record example.com '' MX '10 correo.example.com'
New rrset:
example.com. 3600 IN MX 10 correo.example.com
```
Y un registro A:
```
$ sudo --user=pdns pdnsutil add-record example.com. www A 11.22.33.44
New rrset:
www.example.com. 3600 IN A 11.22.33.44
```

Ahora podemos hacer la consulta (_query_) por el registro A www.example.com
```
dig www.example.com a @127.0.0.1
```
y nos debería contestar con los datos:
```
; <<>> DiG 9.11.5-P4-5.1+deb10u5-Debian <<>> www.example.com a @127.0.0.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 14205
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;www.example.com.		IN	A

;; ANSWER SECTION:
www.example.com.	3600	IN	A	11.22.33.44

;; Query time: 13 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Fri Oct 29 12:01:19 -03 2021
;; MSG SIZE  rcvd: 60
```

Listamos todas las zonas configuradas:
```
$ sudo --user=pdns pdnsutil list-all-zones
```

Finalmente, borramos la zona que creamos para probar:
```
$ sudo --user=pdns pdnsutil delete-zone example.com
```

El otro comando que se utiliza para controlar y consultar el servidor es
`pdns_control`. Por ejemplo, para ver el tiempo que hace que el servidor está
levantado:
```
$ sudo --user=pdns pdns_control uptime
```

# _Logging_

En los Debian nuevos, pdns utiliza las facilidades de _logging_ de `systemd`,
llamada 
[`journald`](https://www.server-world.info/en/note?os=Debian_11&p=journald).

Los logs se manejan en un formato binario y se consultan con la herramienta
[**`journalctl`**](https://manpages.debian.org/systemd/journalctl.1.en.html).

Sin ningún parámetro, `journalctl` muestra todo el log del sistema dentro de
un _pager_.

En general conviene filtrar para ver los mensajes que uno quiere.

Para ver los mensajes de `pdns` se puede filtrar por el nombre de la `unit`:
```
sudo journalctl --unit pdns.service
sudo journalctl -u pdns.service
```
o filtrar por el identificador de syslog:
```
sudo journalctl --identifier pdns_server
sudo journalctl -t pdns_server
```
La ventaja de filtrar por `unit` es que se muestran también mensajes producidos
por `systemd` respecto de esa `unit` (además de los que produce la `unit` en 
sí).

También podemos buscar un _pattern_ en particular con la opción `--grep`. Si
el _pattern_ es todo en minúsculas, la búsqueda es _case insensitive_. Si se
quiere hacer una búsqueda _case sensitive_ con un _pattern_ que sólo tiene
minúsculas hay que agregar la opción `--case-sensitive=y`.
```
sudo journalctl --unit pdns.service --grep fail
sudo journalctl --unit pdns.service -g fail
```

Se puede acotar el rango de tiempo de la búsqueda usando las opciones `--since`
y `--until`:
```
sudo journalctl --unit pdns.service --since "2022-01-11 00:00:00" --until "2022-01-13 23:59:59"
sudo journalctl --unit pdns.service -S "2022-01-11 00:00:00" -U "2022-01-13 23:59:59"
```

Para "seguir" un log a medida que avanza (como si se siguiera un log de texto
utilizando el comando `tail -f`) se puede utilizar la opción `--follow`:
```
sudo journalctl --unit pdns.service --follow
sudo journalctl --unit pdns.service -f
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
