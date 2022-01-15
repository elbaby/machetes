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

Agregar el repositorio en un archivo **`/etc/apt/sources.list.d/pdns.list`** con
el siguiente contenido (para la versión 4.5.x del servidor autoritativo):
```
deb [arch=amd64 signed-by=/usr/share/keyrings/powerdns-archive-keyring.gpg] http://repo.powerdns.com/debian buster-auth-45 main
```

_Pinear_ el repositorio agregando un archivo **`/etc/apt/preferences.d/pdns`**
con el siguiente contenido:
```
Package: pdns-*
Pin: origin repo.powerdns.com
Pin-Priority: 600
```

Ejecutar los siguientes comandos:

```
# Actualizo base de datos de paquetes
sudo apt update

# Instalo pdns y utilitarios basicos para el DNS
sudo apt install pdns-server dnsutils
```

# Instalación y configuración de backends

PowerDNS tiene varios [backends
distintos](https://doc.powerdns.com/authoritative/backends/index.html) (algunos
para usos muy específicos, otros de uso general).

Por default, sólo está habilitado el [backend para archivos de zona tipo 
BIND](https://doc.powerdns.com/authoritative/backends/bind.html).

Hay distintos backends que utilizan bases de datos SQL. El [backend genérico 
SQL](https://doc.powerdns.com/authoritative/backends/generic-sql.html)
describe la funcionalidad común a todos ellos.

* [Instrucciones para instalar y configurar](pdns-be-postgresql.md) el 
[backend genérico 
PostgreSQL](https://doc.powerdns.com/authoritative/backends/generic-postgresql.html)
* [Instrucciones para instalar y configurar](pdns-be-sqlite3.md) el 
[backend genérico SQLite 
3](https://doc.powerdns.com/authoritative/backends/generic-sqlite3.html)

# Pruebas básicas

Para probar que el servicio está levantado le hacemos una consulta cualquiera:
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
$ sudo -u pdns pdnsutil create-zone example.com a.example.com
Creating empty zone 'example.com'
Also adding one NS record
```
Agrego un registro MX:
```
$ sudo -u pdns pdnsutil add-record example.com '' MX '10 correo.example.com'
New rrset:
example.com. 3600 IN MX 10 correo.example.com
```
Y un registro A:
```
$ sudo -u pdns pdnsutil add-record example.com. www A 11.22.33.44
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
$ sudo -u pdns pdnsutil list-all-zones
```

Finalmente, borramos la zona que creamos para probar:
```
$ sudo -u pdns pdnsutil delete-zone example.com
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
