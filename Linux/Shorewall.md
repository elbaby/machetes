# Shoreline Firewall (Shorewall)

## Bibliografía

* [Introducción a _Shoreline Firewall_
(_Shorewall_)](https://shorewall.org/Introduction.html)
* [Guías rápidas (_HOWTOs_) de
_Shorewall_](https://shorewall.org/shorewall_quickstart_guide.htm)
* [Firewall para un linux _standalone_](https://shorewall.org/standalone.htm)
(el caso nuestro en RIU)
* [Operación de _Shorewall_ y _Shorewall
Lite_](https://shorewall.org/starting_and_stopping_shorewall.htm)
* [Consejos para los archivos de
configuración](https://shorewall.org/configuration_file_basics.htm)

## Instalación

* [Instalación y actualización de
_Shorewall_](https://shorewall.org/Install.htm)

Se puede [configurar APT para que use 
_testing_](https://shorewall.org/Install.htm#Debian) que suele tener la última
versión disponible de Shorewall (en 2023-07, la 5.2.8), pero la versión incluída 
en buster y bullseye (5.2.3) es razonable.
 
La instalación es simplemente:
```
sudo apt-get udpate
sudo apt-get install shorewall shorewall6
```
Con esto instalamos el firewall para IPv4 (shorewall) y para IPv6 (shorewall6).

En `/usr/share/doc/shorewall/README.Debian.gz` y 
`/usr/share/doc/shorewall6/README.Debian` quedan las notas específicas para
las instalaciones en Debian.

En Debian y Ubuntu, en principio, los servicios están desactivados, con lo cual,
por sólo instalarlos, no se levanta el firewall y no corremos riesgo -todavía-
de quedar fuera.

## Configuración

Las configuraciones de ejemplo para un firewall de una sola interfaz (es decir
donde sólo estamos protegiendo el mismo equipo) quedan en el directorio
`/usr/share/doc/shorewall/examples/one-interface/` para IPv4 y en el directorio
`/usr/share/doc/shorewall6/examples/one-interface/` para IPv6.

Como habilitamos _Shorewall_ para IPv4 e IPv6, vamos a tener que configurar
ambos en forma análoga al mismo tiempo en `/etc/shorewall` y `/etc/shorewall6`
respectivamente.

### Archivos de configuración vacíos y anotados

La instalación del paquete en Debian y Ubuntu deja en `/etc/shorewall` (y 
`/etc/shorewall6`) solamente el archivo `shorewall(6).conf` con una
configuración básica (muy parecida a la que trae el archivo en
`/usr/share/doc/shorewall(6)/examples/one-interface/`).

Vamos a copiar otros archivos que necesitamos para modificar desde
`/usr/share/doc/shorewall(6)/examples/one-interface/` y además dejaremos links
simbólicos a las versiones "anotadas" de esos archivos, que vienen con
explicaciones de las opciones que hay en los mismos.

```
for IPversion in 4 6 ; do
  # La versión IPv4 no usa el número 4 en los nombres de archivo
  if [ ${IPversion} -eq 4 ] ; then IPversion="" ; fi

  # symlink a shorewall(6).conf anotado
  sudo ln -sv /usr/share/doc/shorewall${IPversion}/examples/one-interface/shorewall${IPversion}.conf.annotated.gz \
      /etc/shorewall${IPversion}/

  for ConfigFile in params interfaces zones policy rules stoppedrules; do
    EXT=".gz"
    EXPATH="examples/one-interface"
    # el archivo params no viene comprimido
    if [ ${ConfigFile} = "params" ] ; then EXT="" ; fi
    # el archivo stoppedrules no está en los ejemplos de una interfaz
    if [ ${ConfigFile} = "stoppedrules" ] ; then EXPATH="examples/two-interfaces" ; fi

    # copiar archivo de configuración vacío
    sudo cp -v /usr/share/doc/shorewall${IPversion}/${EXPATH}/${ConfigFile} \
        /etc/shorewall${IPversion}/
    # symlink a archivo de configuración anotado
    sudo ln -sv /usr/share/doc/shorewall${IPversion}/${EXPATH}/${ConfigFile}.annotated${EXT} \
        /etc/shorewall${IPversion}/

  done
done
```

### Archivo de configuración general (`shorewall.conf`/`shorewall6.conf`)

El archivo
[`shorewall.conf`](https://shorewall.org/manpages/shorewall.conf.html) permite
configurar opciones de funcionamiento y configuración de _Shorewall_. Para IPv6,
el archivo (que está en `/etc/shorewall6`) se llama `shorewall6.conf`. En
principio, el que se instala con el paquete tiene valores razonables.

**_Asegurarse_** de que esté la siguiente línea configurada (en general está):
```
ADMINISABSENTMINDED=Yes
```
Esto hace que cuando se detiene el firewall no se cierren las conexiones que ya
están (por ejemplo el acceso remoto a través del cual se detuvo el firewall).

Algunos cambios que hay que hacer:
```
LOGFILE=systemd
```
Esto _no_ define dónde loguea shorewall, si no dónde busca el log del sistema
para los comandos `shorewall show log`, `shorewall hits`, etc. 
Los debian nuevos utilizan el logging de systemd.

En el caso de que haya contenedores [Docker](https://www.docker.com/) en el
equipo es _imprescindible_ poner `Yes` en la opción
[`DOCKER`](https://shorewall.org/Docker.html) (ya que Docker también utiliza
`iptables` y, si no está esta opción, Shorewall borra todas las reglas que puso
Docker):
```
DOCKER=Yes
```
(esta opción se usa únicamente en IPv4, en `/etc/shorewall/shorewall.conf`).

### Parámetros (`params`)

En el archivo[`params`](https://shorewall.org/manpages/shorewall-params.html) se
configuran variables que se pueden utilizar en otros archivos de configuración.

* `/etc/shorewall/params`

### Interfaces (`interfaces`)

En el archivo 
[`interfaces`](https://shorewall.org/manpages/shorewall-interfaces.html) se
definen las interfaces de red.

Es importante conocer el **nombre** de la interfaz de red (es fácil de ver
usando el comando `ip a s`). En general en Debian 9 o más nuevos es algo tipo
`enoX`, `ensX`, `enpXsY` o similar.

En caso de que el equipo tenga un IP fija y no use (ni brinde) el servicio
DHCP, no hay que poner la opción `dhcp` que viene en los archivos de ejemplo.

Si el equipo usa Docker, hay que identificar la interfaz del bridge default de
Docker (usualmente `docker0`).

Suponiendo que la interfaz de red se llame **`ens18`** y el bridge de docker se
llame **`docker0`** los archivos de interfaces quedarían así:

* `/etc/shorewall/interfaces`:
```
# For information about entries in this file, type "man shorewall-interfaces"
###############################################################################
?FORMAT 2
###############################################################################
#ZONE   INTERFACE   OPTIONS
net     NET_IF      tcpflags,logmartians,nosmurfs,sourceroute=0,physical=ens18
dock    docker0     bridge #Allow ICC (bridge implies routeback=1)
```

* `/etc/shorewall6/interfaces`:
```
# For information about entries in this file, type "man shorewall6-interfaces"
###############################################################################
?FORMAT 2
###############################################################################
#ZONE   INTERFACE   OPTIONS
net     NET_IF      tcpflags,physical=ens18
```

### Zonas (`zones`)

En el archivo [`zones`](https://shorewall.org/manpages/shorewall-zones.html) que
define las zonas, vamos a definir la zona `fw` para el firewall en sí mismo
(obligatoria) y una zona `net` para la (única) red externa, de tipo `ipv4` o
`ipv6` (según cuál de las dos instancias estamos configurando).

* `/etc/shorewall/zones`:
```
# For information about entries in this file, type "man shorewall-zones"
###############################################################################
#ZONE   TYPE    OPTIONS         IN          OUT
#                               OPTIONS     OPTIONS
fw      firewall
net     ipv4
dock    ipv4
```

* `/etc/shorewall6/zones`:
```
# For information about entries in this file, type "man shorewall6-zones"
###############################################################################
#ZONE   TYPE    OPTIONS         IN          OUT
#                               OPTIONS     OPTIONS
fw      firewall
net     ipv6
```

### Política (`policy`)

En el archivo [`policy`](https://shorewall.org/manpages/shorewall-policy.html)
se define la _política_ de qué se debe hacer con las conexiones en general que
van de una zona a otra (el _default_).

Normalmente, la política consiste en descartar lo que viene desde la red
externa hacia el firewall. Aceptar lo que va del firewall a la red externa y
rechazar todo lo demás (esta regla es obligatoria y _debe_ ir última).

* `/etc/shorewall/policy`:
```
# For information about entries in this file, type "man shorewall-policy"
###############################################################################
#SOURCE DEST        POLICY      LOGLEVEL    RATE    CONNLIMIT
$FW     net         ACCEPT
net     all         DROP        $LOG_LEVEL
dock    $FW         REJECT      $LOG_LEVEL
dock    all         ACCEPT
# The FOLLOWING POLICY MUST BE LAST
all     all         REJECT      $LOG_LEVEL
```

* `/etc/shorewall6/policy`:
```
# For information about entries in this file, type "man shorewall6-policy"
###############################################################################
#SOURCE DEST        POLICY      LOGLEVEL    RATE    CONNLIMIT
$FW     net         ACCEPT
net     all         DROP        $LOG_LEVEL
# The FOLLOWING POLICY MUST BE LAST
all     all         REJECT      $LOG_LEVEL
```

### Reglas (`rules`)

En el archivo [`rules`](https://shorewall.org/manpages/shorewall-rules.html) se
definen las reglas específicas que normalmente funcionan como **excepciones** a
la _política_.

Aquí vamos a configurar, sobre el archivo de reglas que trae la configuración
de ejemplo, permiso para conectividad **web** (**http** y **https**) y **ssh**
desde cualquier equipo en internet hacia el firewall.

* `/etc/shorewall/rules`:
```
# For information on entries in this file, type "man shorewall-rules"
##################################################################################################################################################################
#ACTION         SOURCE      DEST        PROTO   DEST    SOURCE      ORIGINAL    RATE        USER/   MARK    CONNLIMIT   TIME        HEADERS     SWITCH      HELPER
#                                               PORT    PORT(S)     DEST        LIMIT       GROUP
?SECTION ALL
?SECTION ESTABLISHED
?SECTION RELATED
?SECTION INVALID
?SECTION UNTRACKED
?SECTION NEW

# Drop packets in the INVALID state
Invalid(DROP)   net         $FW         tcp

# Drop Ping from the "bad" net zone.. and prevent your log from being flooded..
Ping(DROP)      net         $FW

# Permit all ICMP traffic FROM the firewall TO the net zone
ACCEPT          $FW         net         icmp

# Aceptar conexiones web (http/https)
Web(ACCEPT)     net         $FW

# Aceptar conexiones ssh
SSH(ACCEPT)     net         $FW
```

* `/etc/shorewall6/rules`:
```
# For information on entries in this file, type "man shorewall6-rules"
##################################################################################################################################################################
#ACTION         SOURCE      DEST        PROTO   DEST    SOURCE      ORIGINAL    RATE        USER/   MARK    CONNLIMIT   TIME        HEADERS     SWITCH      HELPER
#                                               PORT    PORT(S)     DEST        LIMIT       GROUP
?SECTION ALL
?SECTION ESTABLISHED
?SECTION RELATED
?SECTION INVALID
?SECTION UNTRACKED
?SECTION NEW

# Drop packets in the INVALID state
Invalid(DROP)   net         $FW         tcp

# Drop Ping from the "bad" net zone.. and prevent your log from being flooded..
Ping(DROP)      net         $FW

# Permit all ICMP traffic FROM the firewall TO the net zone
ACCEPT          $FW         net         ipv6-icmp

# Aceptar conexiones web (http/https)
Web(ACCEPT)     net         $FW

# Aceptar conexiones ssh
SSH(ACCEPT)     net         $FW
```

### Reglas para cuando se _detiene_ el firewall (`stoppedrules`)

En este archivo hay que configurar las reglas mínimas que permiten utilizar el
equipo. En principio, la opción `ADMINISABSENTMINDED` del `shorewall.conf` se
ocupa de mantener las conexiones existentes y de permitir conexiones _desde_ el
firewall hacia cualquier lado.

Como este equipo sólo se puede gestionar remotamente vía ssh, tenemos que 
agregar una regla que permita este acceso aún cuando el firewall está detenido:

* `/etc/shorewall/stoppedrules`:
```
# For information about entries in this file, type "man shorewall-stoppedrules"
###############################################################################
#ACTION     SOURCE          DEST        PROTO   DPORT   SPORT
# Aceptar conexiones ssh desde cualquier IP
ACCEPT      NET_IF          $FW         tcp     22
```

* `/etc/shorewall6/stoppedrules`:
```
# For information about entries in this file, type "man shorewall6-stoppedrules"
###############################################################################
#ACTION     SOURCE          DEST        PROTO   DPORT   SPORT
# Aceptar conexiones ssh desde cualquier IP
ACCEPT      NET_IF          $FW         tcp     22
```

## Servicios systemd

**ATENCIÓN**: _Shorewall_ **no** es un servicio que arranca y para (como un
servidor web o ssh). _Shorewall_ solamente compila un conjunto de reglas y
ejecuta comandos ´iptables´ para lograr la configuración deseada del firewall
interno del kernel de linux ([netfilter](https://www.netfilter.org/)).

### Corregir los archivos del servicio systemd

El servicio configurado en Debian hace que si se "detiene" el servicio (`sudo 
systemctl start shorewall`), en vez de utilizar el comando `shorewall stop` (que
deja el sistema protegido) utiliza el comando `shorewall clear` (que borra todas
las reglas y deja el sistema abierto).

Además, al menos en Ubuntu 22.04 y Debian 11, el archivo de configuración del
servicio tiene configurado `StandardOutput=syslog` lo que ya está deprecado (hay
que usar `journal` para loguear a systemd).

Para corregir esto (y que la corrección sobreviva a actualizaciones de
_Shorewall_), ejecutar el comando:
```
sudo systemctl edit shorewall.service
```
Esto va a abrir un editor que mustra un comentario arriba indicando que se va a
editar el archivo `/etc/systemd/system/shorewall.service.d/override.conf` y
debajo el contenido del archivo de servicio de systemd
`/lib/systemd/system/shorewall.service`.

En el medio, hay que agregar lo siguiente:
```
[Service]
# reset StandardOutput
StandardOutput=
# set StandardOutput to "journal" instead of "syslog"
StandardOutput=journal
# reset ExecStop
ExecStop=
# set ExecStop to "stop" instead of "clear"
ExecStop=/sbin/shorewall $OPTIONS stop
```

Análogamente, ejecutar el comando:
```
sudo systemctl edit shorewall6.service
```

y agregar lo siguiente:
```
[Service]
# reset StandardOutput
StandardOutput=
# set StandardOutput to "journal" instead of "syslog"
StandardOutput=journal
# reset ExecStop
ExecStop=
# set ExecStop to "stop" instead of "clear"
ExecStop=/sbin/shorewall -6 $OPTIONS stop
```

Para activar los cambios en systemd ejecutar:
```
sudo systemctl daemon-reload
```

### Habilitar el servicio

En Ubuntu y Debian los servicios está deshabilitado luego de la instalación.

Para habilitarlos (es decir, para que los servicios arranquen al bootear), 
ejecutar los siguientes comandos:
```
sudo systemctl enable shorewall shorewall6
```

Para iniciarlo manualmente:
```
sudo systemctl start shorewall shorewall6
```

## Soporte en fail2ban

Si el equipo tiene configurado **fail2ban** lo más probable es que las `action`s
(`actionban` y `actionunban`) estén utilizando `iptables` _standalone_.

Como fail2ban tiene soporte para Shorewall, simplemente hay que adaptarlo para
que las use.

La configuración mínima es agregar un archivo de configuración 
`/etc/fail2ban/jail.local` (si no existe), con el siguiente contenido:

```
[DEFAULT]
banaction = shorewall
banaction_allports = shorewall
```

Si el archivo ya existe, buscar la sección `[DEFAULT]` y agregar las dos últimas
líneas.

La acción de _banning_ configurada por defecto en 
`/etc/fail2ban/action.d/shorewall.conf` es **`reject`**. Preferimos utilizar
**`drop`**. Para cambiar esto, agregamos un archivo
`/etc/fail2ban/action.d/shorewall.local` con el siguiente contenido:
```
[Init]

# Option:  blocktype
# Note:    This is what the action does with rules.
#          See man page of shorewall for options that include drop, logdrop, reject, or logreject
# Values:  STRING
blocktype = drop
```

Para que se tomen todos los cambios, ejecutar:
```
sudo systemctl restart fail2ban
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
