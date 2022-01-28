# Configuración de logging con syslog

**ESTO NO SIRVE**: Terminan peleándose el `journald` con el `rsyslog` y no
consigo hacer lo que quiero. Habrá que aprender a usar `journalctl` para ver
los logs de `journald`.

https://doc.powerdns.com/authoritative/running.html#logging-to-syslog

Normalmente, pdns envía logging a 
[`syslog`](https://devconnected.com/syslog-the-complete-system-administrator-guide/)
usando la
[_facility_](https://devconnected.com/linux-logging-complete-guide/#Syslog_Facilities_Explained) **`daemon`**.

Para evitar que los logs de pdns se mezclen con los demás servicios que corren
en el servidor, se puede configurar para que use alguna de las 8 _facilities_
locales (`local0` a `local7`).

Elegimos (arbitrariamente) `local3` (hay que confirmar que no haya otro
servicio utilizando esa _facility_.

Primero, configuramos el `rsyslog` (el daemon de syslog que tienen los linux
actuales).

Como pdns corre con usuario `pdns` creamos un directorio para los logs donde
ese usuario tenga permiso de escritura:

```
sudo mkdir /var/log/pdns
sudo chown pdns.pdns /var/log/pdns
sudo chmod 0755 /var/log/pdns
```

Ponemos la configuración de `rsyslog` en un archivo separado
`/etc/rsyslog.d/pdns.conf`:
```
# Configuración de rsyslog para pdns
# Usamos facility local3
# en /etc/powerdns/pdns.conf configurar
#   logging-facility=3
# (tener en cuenta que sólo se pone el número 3
# NO el nombre local3)

# Cada uno de los siguientes archivos tendrá
# los mensajes de ese nivel y todos los inferiores
# Sin embargo, para habilitarlo hay que configurar
# el loglevel en /etc/powerdns/pdns.conf
# El default es 4 (warn) y no debe configurarse
# menos que 3 (ya que pdns no produce mensajes
# de mayor criticidad que err).

# debug: loglevel=7
local3.debug    -/var/log/pdns/pdns.debug
# info: loglevel=6
local3.info     -/var/log/pdns/pdns.info
# notice: loglevel=5
local3.notice   -/var/log/pdns/pdns.notice
# warn: loglevel=4
local3.warn     -/var/log/pdns/pdns.warn
# err: loglevel=3
local3.err      /var/log/pdns/pdns.err
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
