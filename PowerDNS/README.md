# PowerDNS

[PowerDNS](https://www.powerdns.com/) es uno de los servidores DNS que andan
bien y son open source (GPLv2).

Es usual la confusión respecto de qué hace (y especialmente cómo se configura)
un "_servidor DNS_" ya que hay **dos cosas distintas** a las que llamamos
"_servidor DNS_".
Esto está acentuado porque algunos productos (notoriamente [ISC BIND](
https://www.isc.org/bind/) y [Microsoft DNS](
https://docs.microsoft.com/en-us/windows-server/networking/dns/dns-top)) son
servidores monolíticos que brindan ambos servicios.

Los dos servicios son:

* **Servidor de nombres autoritativo de zona** que se utiliza para publicar
información acerca de zonas por parte del titular de dichas zonas. Esta
publicación es, en principio, para que la consuma toda la internet.

* **Servidor iterativo** (mal llamado **_recursivo_**) **de resolución de
nombres** (o simplemente **_resolver_**) que se utiliza para obtener la
información de _cualquier_ zona en internet. Esta publicación es, en principio,
para que la consuman los clientes de ese servidor (normalmente, dentro de la
misma organización o de clientes de un proveedor). Existen también _resolvers_
públicos (como el
[8.8.8.8 de Google](https://developers.google.com/speed/public-dns), el
[9.9.9.9 de Quad9](https://www.quad9.net/) o el
[1.1.1.1 de Cloudflare](https://www.quad9.net/)) que dan servicio de resolución
para cualquiera que lo desee utilizar.

En [NIC Argentina](https://nic.ar) hay una [explicación de cómo funcionan estos
servicios](https://nic.ar/es/novedades/noticias/como-funciona-el-dns).

La mayoría de los servidores modernos (al menos los open source que conozco),
brindan ambos servicios por separado (a través de distintos servidores).

En el caso de [PowerDNS](https://www.powerdns.com/software.html), el [PowerDNS
Authoritative Server](https://www.powerdns.com/auth.html) es un **servidor de
nombres autoritativo** y el
[PowerDNS Recursor](https://www.powerdns.com/recursor.html) es un **servidor
iterativo de resolución** (**_resolver_**). PowerDNS tiene otro producto llamado
[PowerDNS dnsdist](https://www.powerdns.com/dnsdist.html) que es un _load
balancer_ para DNS.

## Servidores autoritativos con _PowerDNS Authoritative Server_ (pdns)

**[pdns](https://doc.powerdns.com/authoritative/)** soporta múltiples back-ends.
Es decir, la información de las zonas que publica puede estar en bases de datos
relacionales, en archivos de zona tipo BIND o inclusive ser accedidos a traves
de un _pipe_ desde otro proceso o inclusive a través de una API desde otro tipo
de servidor.

Nosotros vamos a instalarlo con el backend en un servidor
[PostgreSQL](https://www.postgresql.org/).

Si bien pdns puede actuar como un servidor primario o secundario y transferir
zonas a través del mismo protocolo DNS usando NOTIFY, AXFR e IXFR, se recomienda
realizar la sincronización entre servidores autoritativos _fuera de banda_. Esto
es usualmente simple utilizando los mecanismos de replicación de las bases de
datos (especialmente si todos los servidores son administrados por la misma
organización). Esto es lo que la documentación de PowerDNS llama [replicación
nativa](https://doc.powerdns.com/authoritative/modes-of-operation.html#native-replication).


### Instalación

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

#### Esquema postgres
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

#### Configuración básica del backend
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

**Nota**: Para hacer pruebas de DNS siempre es recomendable usar **`dig`** (o
`drill`). Nunca conviene usar `host` o `nslookup`.


<!--
### Modo de operación
PowerDNS Authoritative tiene varios
[modos de operación](https://doc.powerdns.com/authoritative/modes-of-operation.html).
Nosotros vamos a utilizar
[replicación nativa](https://doc.powerdns.com/authoritative/modes-of-operation.html#native-replication)
con lo que tendremos que ocuparnos de replicar la base de datos postgres entre
el primario y todos los secundarios.
-->

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