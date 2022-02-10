# Configuración del backend genérico PostgreSQL en PowerDNS autoritativo

https://doc.powerdns.com/authoritative/guides/basic-database.html

https://doc.powerdns.com/authoritative/backends/generic-postgresql.html

Ejecutar el siguiente comando para instalar el cliente PostgreSQL y el backend
correspondiente para pdns:
```
sudo apt install postgresql-client pdns-backend-pgsql
```

Si se quiere usar un servidor PostgreSQL local, hay que instalar también dicho
servidor:
```
sudo apt install postgresql pdns-backend-pgsql
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

sudo --user=postgres psql -c "CREATE DATABASE $DB_NOMBRE"
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

Ahora hay que editar el archivo de configuración del backend de postgres 
(`/etc/powerdns/pdns.d/gpgsql.conf`) con los [parámetros 
deseados](https://doc.powerdns.com/authoritative/backends/generic-postgresql.html#settings).

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
