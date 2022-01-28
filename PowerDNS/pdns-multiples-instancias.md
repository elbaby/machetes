# Configuración de múltiples intancias de PowerDNS Autoritativo en el mismo server

https://doc.powerdns.com/authoritative/guides/virtual-instances.html

# Nombres de instancias
Las instancias conviene nombrarlas con un string que no contenga guíones.
Por ejemplo: PRINCIPAL, PRIVADO, EXTERNO, INTERNO, CLIENTES, etc.

# Convertir la instalación _default_ en una instancia.
Una vez [instalado y configurado](pdns-instalacion.md) el servidor, convertimos
la configuración única en una instancia.

Primero detenemos el servicio actual:
```
sudo systemctl stop pdns.service
```
Ahora hay **renombrar** el archivo `/etc/powerdns/pdns.conf` para incluir el
nombre de la instancia. En base al nombre de este archivo es que los scripts
de control de `systemd`.

Además, si bien no es obligatorio, podemos crear un directorio `pdns.d` para
cada instancia que nos permita mantener cada configuración modularizada en
pequeños archivos con configuraciones específicas.

Supongamos que a la configuración actual (única) la queremos llamar
**`PRINCIPAL`** entonces:

```
# renombrar el archivo de configuración principal (único paso obligatorio)
sudo mv /etc/powerdns/pdns.conf /etc/powerdns/pdns-PRINCIPAL.conf

# renombrar el directorio de configuraciones modulares
sudo mv /etc/powerdns/pdns.d /etc/powerdns/pdns-PRINCIPAL.d

# cambiar el nombre del directorio de configuraciones modulares en el
# archivo de configuración
sudo sed --in-place=.borrar \
  --expression='s#/etc/powerdns/pdns.d#/etc/powerdns/pdns-PRINCIPAL.d#' \
  /etc/powerdns/pdns-PRINCIPAL.conf

# Si todo salió bien, se puede borrar el backup del archivo de configuración
sudo rm /etc/powerdns/pdns-PRINCIPAL.conf
```

## Controlar la instancia con systemd

Usando `systemctl` ahora tenemos que especificar el nombre de la instancia
cuando arrancamos, detenemos o consultamos el estado del servicio:
```
sudo systemctl start pdns@PRINCIPAL.service
sudo systemctl status pdns@PRINCIPAL.service
sudo systemctl stop pdns@PRINCIPAL.service
```

## Configurar systemd para que la instancia arranque automáticamente

El _target file_ que hay que usar en `systemd` es 
`/lib/systemd/system/pdns@.service` (en lugar de 
`/lib/systemd/system/pdns.service` que es el que se usa en la configuración
_default_).

Para levantar automáticamente la instancia **`PRINCIPAL`** cuando arranca
el _target_  `multi-user` hay que poner un link simbólico al _target file_
correspondiente con el nombre de la instancia **`PRINCIPAL`** en
`/etc/systemd/system/multi-user.target.wants`:
```
sudo ln -s /lib/systemd/system/pdns@.service /etc/systemd/system/multi-user.target.wants/pdns@PRINCIPAL.service
```

También hay que borrar el link al _target file_ de la configuración _default_
para que no intente arrancar (de todos modos va a fallar porque no existe el
archivo `/etc/powerdns/pdns.conf`):
```
sudo rm -fv /etc/systemd/system/multi-user.target.wants/pdns.service
```
## Línea de comandos de `pdnsutil` y `pdns_control`

En los comandos `pdnsutil` `pdns_control` hay que agregar la opción 
`--config-name` para explicitar cuál es la instancia que se debe utilizar y la
opción `--socket-dir` para indicarle el directorio donde está el _socket_ de
control:
```
$ sudo --user=pdns pdnsutil --config-name=PRINCIPAL list-all-zones
$ sudo --user=pdns pdns_control --config-name=PRINCIPAL --socket-dir=/var/run/pdns-PRINCIPAL uptime
```

# Configurar una segunda instancia

Cuando se configuran múltiples instancias hay que planificar antes para que
las instancias atiendan en direcciones y/o puertos diferentes.

Por default, el servidor escucha en todas las direcciones IP del equipo:
```
local-address=0.0.0.0, ::
local-port=53
```

## Usar diferentes direcciones IP
En un equipo con múltiples direcciones IP, se puede configurar que cada
instancia utilice una dirección IP distinta:
```
# pdns-PRINCIPAL
local-address=10.10.10.10 f2da:de4e:e5e:1010::
local-port=53
```
```
# pdns-PRIVADO
local-address=10.10.50.50 f2da:de4e:e5e:5050::
local-port=53
```

Si se configura el servidor web o la API REST también hay que asegurarse de que
también atiendan en direcciones IP diferentes.

## Usar diferentes ports

La otra variante es que las instancias utilicen ports diferentes (para que
funcione esto, hay que configurar luego un resolver que consulte en el port
no estándar):
```
# pdns-PRINCIPAL
local-address=0.0.0.0 ::
local-port=53
```
```
# pdns-PRIVADO
local-address=0.0.0.0 ::
local-port=5300
```

Si se configura el servidor web o la API REST también hay que asegurarse de que
utilicen ports diferentes.

### Notificaciones y transferencias en puertos no estándar

Si se configuran primarios y secundarios atendiendo en puertos distintos del 53
hay que asegurarse de enviar las notificaciones a estos puertos y NO al puerto
53 y que las transferencias se soliciten a estos puertos.

Suponiendo que el primario utiliza el port 5300 y el secundario el 5353, 
entonces el primario debe enviar las notificaciones al puerto 5353 del 
secundario y este debe solicitar las transferencias al puerto 5300 del primario.

#### Configuraciones en el servidor primario

Suponiendo que la instancia que utiliza puertos no estándar se llama PRIVADO,
el primario utiliza el port 5300 y el secundario el 5353, en el archivo de
configuración `/etc/powerdns/pdns-PRIVADO.conf` se puede poner:
```
also-notify=10.20.30.40:5353 10.10.90.90:5353
```
de este modo, siempre notificará a los secundarios en esas direcciones IP y
esos puertos.

Si se quieren configurar direcciones IP y/o puertos diferentes según cada
dominio en particular, entonces se puede dejar la configuración `also-notify`
en blanco y configurar los metadatos de cada dominio, por ejemplo:
```
sudo -u pdns pdnsutil --config-name PRIVADO set-meta example.com ALSO-NOTIFY 10.20.30.40:5353
sudo -u pdns pdnsutil --config-name PRIVADO set-meta example.org ALSO-NOTIFY 10.10.90.90:5353
```

El problema adicional es que las zonas normalmente tienen registros de tipo
**`NS`** y estos registros sólo llevan nombres de dominio y nunca un puerto.

Normalmente, pdns **notificará** a las direcciones IP que pueda resolver de
los nombres que encuentre en los registros de tipo `NS`. 

Si se quiere evitar esto, en el archivo de configuración
`/etc/powerdns/pdns-PRIVADO.conf` hay que poner:
```
only-notify=
```
Si la configuración `only-notify` contiene una lista vacía, no notificará a
ningún servidor **_excepto_** los que estén en la configuración `also-notify`
o en el metadato `ALSO-NOTIFY` del dominio (pdns _siempre_ notifica a los
servidores en `also-notify` y `ALSO-NOTIFY`).

#### Configuraciones en el servidor secundario

En el secundario, cuando se crea la zona, hay que indicarle el primario
indicando la dirección **y** el número de puerto:
```
sudo -u pdns pdnsutil --config-name PRIVADO create-secondary-zone example.com 10.10.10.50:5300
```
Si la zona ya está creada y no se indicó el número de puerto al hacerlo (o si 
se quiere configurar un primario distinto):
```
sudo -u pdns pdnsutil --config-name PRIVADO change-secondary-zone-primary example.com 10.10.10.50:5300
```

## Arranque de las instancias en `systemd`

También hay que configurar para que _cada_ instancia arranque automáticamente
en `systemd`.

# Ver los logs de las instancias con `journalctl`
Las opciones de [_logging_](pdns-instalacion.md#logging) ahora hay que
ajustarlas a las instancias. El nombre de la `unit` ahora es 
`pdns@INSTANCIA.service` (o simplemente `pdns@INSTANCIA`).
El identificador es `pdns_server-INSTANCIA`, con lo cual, para ver sólo los
mensajes de una instancia de pdns se puede usar:
```
sudo journalctl --unit pdns@PRINCIPAL.service
sudo journalctl -u pdns@PRINCIPAL.service
```
o filtrar por el identificador de syslog:
```
sudo journalctl --identifier pdns_server-PRINCIPAL
sudo journalctl -t pdns_server-PRINCIPAL
```
Una ventaja adicional que tiene la opción `--unit` por sobre `--identifier` es
que la primera soporta un _pattern_  además de un string fijo, con lo cual
podemos ver **todas** las instancias de pdns:
```
sudo journalctl --unit 'pdns@*.service'
sudo journalctl -u 'pdns@*.service'
```
o más simplemente:
```
sudo journalctl --unit 'pdns*'
sudo journalctl -u 'pdns*'
```
Es **_importante_** utilizar comillas simples (**`'`**) al escribir el 
_pattern_ para evitar que el shell lo interprete (si, por ejemplo, hay un 
arhivo cuyo nombre comience con '`pdns`' en el directorio actual).

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
