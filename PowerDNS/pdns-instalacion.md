# Instalación de PowerDNS Authoritative Server

https://doc.powerdns.com/authoritative/installation.html

https://doc.powerdns.com/authoritative/guides/basic-database.html

https://doc.powerdns.com/authoritative/backends/generic-postgresql.html

La versión de PowerDNS Authoritative Server en los repos de Debian es medio
vieja (en 2021-08, con la versión 4.5.0 en la calle, los repos de Buster van por
la 4.1.6), con lo cual vamos a instalarla desde los
[repositorios de PowerDNS](https://repo.powerdns.com/).

Agregar el repositorio en un archivo **`/etc/apt/sources.list.d/pdns.list`** con
el siguiente contenido (para la versión 4.5.x del servidor autoritativo):
```
deb [arch=amd64] http://repo.powerdns.com/debian buster-auth-45 main
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
# Obtengo la clave pública con la que están firmados los paquetes y el repositorio
curl https://repo.powerdns.com/FD380FBB-pub.asc | sudo apt-key add -

# Actualizo base de datos de paquetes
sudo apt update

# Instalo pdns con el backend para postgresql, el postgresql y utilitarios basicos para el DNS
sudo apt install pdns-server pdns-backend-pgsql postgresql dnsutils
```

## Esquema postgres
Ahora necesitamos el
[esquema](https://doc.powerdns.com/authoritative/backends/generic-postgresql.html#default-schema)
para el postgres. El paquete para Debian deja este esquema en
`/usr/share/pdns-backend-pgsql/schema/schema.pgsql.sql`

Creamos una base de datos, un usuario con clave y cargamos el esquema:

```
# Hay que tener valores cargados en las siguientes variables de entorno
# $DB_NOMBRE (nombre para la base de datos)
# $DB_USUARIO (nombre de usuario para esa base de datos) ¡NO USAR "postgres"!
# $DB_USU_CLAVE (clave para ese usuario)

sudo --user=postgres psql -c "CREATE USER $DB_USUARIO WITH PASSWORD '$DB_USU_CLAVE'"
sudo --user=postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE $DB_NOMBRE TO $DB_USUARIO"

# seteo la variable de ambiente $PGPASSWORD con la clave para que no me la pida
export PGPASSWORD=$DB_USU_CLAVE
psql --host="localhost" --username="$DB_USUARIO" $DB_NOMBRE < /usr/share/pdns-backend-pgsql/schema/schema.pgsql.sql
```

## Configuración básica del backend
Al instalar `pdns-server` se instaló también el backend que utiliza
configuración al estilo BIND (`pdns-backend-bind`). Para desactivar la
configuración por defecto (pero guardarla por si acaso) hacemos lo siguiente:
```
# directorio para backup de configuraciones originales
sudo mkdir /etc/powerdns/conf.ORI
# backup de la configuración básica
sudo cp -p /etc/powerdns/pdns.conf /etc/powerdns/conf.ORI
# desactivamos y backupeamos la configuración del backend BIND
sudo mv /etc/powerdns/pdns.d/bind.conf /etc/powerdns/conf.ORI
sudo mv /etc/powerdns/named.conf /etc/powerdns/conf.ORI
```

Ahora copiamos la configuración ejemplo que viene en el paquete del backend de
postgres y le cambiamos los permisos ya que ahí vamos a tener que poner los
datos de acceso a la base de datos (incluyendo usuario y clave del postgres).
```
sudo cp /usr/share/doc/pdns-backend-pgsql/examples/gpgsql.conf /etc/powerdns/pdns.d/
sudo chmod 0640 /etc/powerdns/pdns.d/gpgsql.conf
sudo chgrp pdns /etc/powerdns/pdns.d/gpgsql.conf
```

Ahora hay que editar el archivo de conifguración del backend de postgres 
(`/etc/powerdns/pdns.d/gpgsql.conf`) con los [parametros 
deseados](https://doc.powerdns.com/authoritative/backends/generic-postgresql.html).

Los parámetros imprescindibles son:

* **`gpgsql-host`** el host donde está la base postgres. Se recomienda 
**_fuertemente_** poner la **dirección IP** (y no el _hostname_) ya que como el
servicio que se está brindando es DNS, si no hay servicio DNS podría no 
resolverse el nombre y el servicio no va a arrancar hasta que no haya 
servicio de DNS (el problema del huevo y la gallina)

* **`gpgsql-port`** el port donde atiende el postgres (usualmente `5432`)

* **`gpgsql-dbname`** el nombre de la base de datos

* **`gpgsql-user`** el nombre del usuario con acceso a la base de datos
(**no usar `postgres`**)

* **`gpgsql-password`** la password del usuario con acceso a la base de datos

Una vez configurados estos datos, reiniciar el servicio:
```
sudo systemctl restart pdns.service
```

## Pruebas básicas

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
