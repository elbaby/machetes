# Configuración del backend genérico SQLite 3 en PowerDNS autoritativo

https://doc.powerdns.com/authoritative/guides/basic-database.html

https://doc.powerdns.com/authoritative/backends/generic-sqlite3.html

Ejecutar el siguiente comando para instalar el SQLite 3 y el backend
correspondiente para pdns:
```
sudo apt install sqlite3 pdns-backend-sqlite3
```

## Esquema sqlite3
Ahora necesitamos el
[esquema](https://doc.powerdns.com/authoritative/backends/generic-sqlite3.html#setting-up-the-database)
para el sqlite3. El paquete para Debian deja este esquema en
`/usr/share/pdns-backend-sqlite3/schema/schema.sqlite3.sql`

Creamos la base de datos con el esquema en `/var/lib/powerdns/pdns.sqlite3`

```
sudo --user=pdns sqlite3 /var/lib/powerdns/pdns.sqlite3 < /usr/share/pdns-backend-sqlite3/schema/schema.sqlite3.sql
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
sqlite3 y le cambiamos los permisos
```
sudo cp /usr/share/doc/pdns-backend-sqlite3/examples/gsqlite3.conf /etc/powerdns/pdns.d/
sudo chmod 0640 /etc/powerdns/pdns.d/gsqlite3.conf
sudo chgrp pdns /etc/powerdns/pdns.d/gsqlite3.conf
```

Ahora hay que editar el archivo de configuración del backend de sqlite3 
(`/etc/powerdns/pdns.d/gsqlite3.conf`) con los [parámetros 
deseados](https://doc.powerdns.com/authoritative/backends/generic-sqlite3.html#configuration-parameters).

El único parámetro imprescindible es:

* **`gsqlite3-database`** el path a la base de datos SQLite3 (ya viene
apuntando a `/var/lib/powerdns/pdns.sqlite3` que es donde la pusimos).

Una vez configurados los datos, reiniciar el servicio:
```
sudo systemctl restart pdns.service
```

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
