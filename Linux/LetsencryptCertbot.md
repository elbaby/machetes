# Certificados Let's Encrypt con `certbot` de la EFF

Referencias:
* https://letsencrypt.org/
  * https://letsencrypt.org/getting-started/
* https://certbot.eff.org/
  * https://certbot.eff.org/instructions
  * https://certbot.eff.org/docs
    * https://certbot.eff.org/docs/using.html#certbot-command-line-options
* https://snapcraft.io/docs/installing-snapd
  * https://snapcraft.io/docs/installing-snap-on-debian

# Instalación de `certbot` en Debian o Ubuntu

## DESINSTALAR versiones viejas

Si el equipo ya estaba usando certificados de Let's Encrypt, es posible que
tenga una versión vieja de `certbot`. O bien un paquete `.deb` instalado con
APT o el más antiguo script `certbot-auto` que se instalaba manualmente.

### El script `certbot-auto`

Si está instalado el viejo script `certbot-auto`, hay que buscarlo y borrarlo.

El script en sí es posible que esté en `/usr/local/bin` o `/usr/local/sbin`:
```
sudo rm -fv /usr/local/*bin/certbot-auto
```

La entrada del `cron` podría estar en `/etc/crontab` o en `/etc/cron.d/*`:
```
sudo sed -i '/certbot-auto/d' /etc/crontab /etc/cron.d/*
```

Todo el paquete en sí posiblemente esté en `/opt/eff.org`:
```
sudo rm -rf /opt/eff.org
```

### El paquete .deb `certbot` instalado con APT

```
sudo apt-get purge certbot
```

## Prerrequisitos

En Ubuntu, `snapd` suele estar instalado, en Debian, en general, no.

### Instalar `snapd` y el snap `core`

```
sudo apt-get update
sudo apt-get install snapd
sudo snap install core
sudo snap refresh core
```

## Instalar el snap de certbot

El snap de certbot hay que instalarlo en modo `classic` para que tenga acceso a
archivos del sistema (el modo `strict` confina todo el snap y no permite que
lea o modifique archivos del sistema).

Después, linkeamos el comando en `/usr/bin` para poder invocarlo fácilmente
sin especificar el path.

```
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin
```

# Uso de `certbot`

`certbot` se ocupa de solicitar, desplegar y renovar certificados X.509
utilizando el [protocolo **ACME** (Automatic Certificate Management
Environment)](https://datatracker.ietf.org/doc/rfc8555/) (_nada que ver con [el
Coyote y el Correcaminos](https://en.wikipedia.org/wiki/Acme_Corporation)_).

El protocolo ACME utiliza una "prueba de posesión" (_challenge_) de los nombres
de dominio ya sea utilizando el protocolo HTTP o el protocolo DNS.

Por ahora nos limitamos a utilizar el protocolo HTTP en el mismo servidor donde
se utilizará el certificado.

## Antes de empezar

### Configurar el firewall

Es necesario que el port 80 (default del servicio HTTP) esté abierto en el
servidor.

Si al servidor se llega a través de un NAT o similar, es importante que el port
80 de la dirección IP pública que corresponde al nombre solicitado se conecte al
servidor donde se va a ejecutar el `certbot` (en caso contrario habrá que
utilizar un _challenge_ vía DNS).

Si el servidor es el que se va a utilizar para brindar el servicio HTTPS,
también hay que abrir el port 443.

### Configurar el nombre de dominio en el DNS público

Todos los nombres de dominio para los que se van a solicitar el certificado
deben estar configurados en el DNS.

Asegurarse de que los nombres estén correctos en _todos_ los servidores
autoritativos para los nombres.

## Solicitar y desplegar un certificado nuevo

### Uso simple (plugins para servidores)

El uso más simple de `certbot` es con el comando `run` (default) que solicita y
despliega los certificados y configura automáticamente el servidor web para que
los utilice.

Se debe especificar la opción para el plugin correspondiente al servidor web
utilizado (por ahora, 2023-07, sólo soporta Apache Web Server y nginx).

La invocación para apache es:
```
certbot --apache
# es lo mismo que certbot run --apache
```
y para nginx:
```
certbot --nginx
# es lo mismo que certbot run --nginx
```

Esto va a pedir interactivamente la aceptación de los términos de servicio
(ToS) y una dirección de mail a donde la autoridad de certificación
(letsencrypt u otra) que emite los certificados puede enviar notificaciones
importantes. Esta dirección debería ser real. Si se utiliza una dirección que
se utiliza para otras cosas, podría ser conveniente poder diferenciar a qué
servidor se refiere. Es común que a la parte local de una dirección se le pueda
agregar un '+' y luego cualquier texto sin que cambie (por ejemplo:
micorreo+certbot_servidorweb.example.com@gmail.com va a ser enviado a la
dirección micorreo@gmail.com).

`certbot` va a leer la configuración del servidor y va a solicitar un
certificado para cada servidor virtual que tenga definido uno o más nombres de
dominio con esos nombres de dominio.

Utilizará el mismo servidor web para validar el _challenge_ HTTP que le presenta
el servidor ACME para obtener los certificados.

Una vez obtenidos los certificados, los despliega y modifica los servidores
virtuales para que redireccione cada uno a un nuevo servidor virtual que atienda
en el port 443 (https), tenga una configuración semejante al servidor virtual
original pero utilizando el certificado obtenido.

#### Ejemplo de interacción

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): hostmaster+certbot_servidorweb.example.com@example.net
```
* Hay que aceptar los términos del servicio del servidor ACME:
```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.3-September-21-2022.pdf. You must
agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: y
```
* Finalmente, lo más importante, es seleccionar el nombre o nombres para los
cuales se solicita el certificado (esto va a leer los `ServerName` y
`ServerAlias` de las configuraciones de apache para elegir los nombres):
```
Which names would you like to activate HTTPS for?
We recommend selecting either all domains, or all domains in a VirtualHost/server block.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: servidorweb.example.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 1
```

Si todo funciona bien, se instala y despliega el certificado
```
Requesting a certificate for servidorweb.example.com

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/servidorweb.example.com/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/servidorweb.example.com/privkey.pem
This certificate expires on 2023-08-27.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Deploying certificate
Successfully deployed certificate for servidorweb.example.com to /etc/apache2/sites-available/servidorweb.example.com-le-ssl.conf
Congratulations! You have successfully enabled HTTPS on https://servidorweb.example.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

#### Timer para renovar los certificados

La instalación de los certificados debe haber configurado un timer en `systemd`
para renovar periódicamente los certificados en
`/etc/systemd/system/snap.certbot.renew.timer`.

Para verificar el funcionamiento de la renovación (sin renovar los certificados)
se puede correr el siguiente comando:
```
sudo certbot renew --dry-run
```

### Configurar los servidores manualmente

Si se prefiere modificar manualmente los archivos de configuración de los
servidores (digamos que el plugin no es especialmente prolijo al armar las
configuraciones), en lugar de `certbot run` hay que usar `certbot certonly`:

La invocación para apache es:
```
sudo certbot certonly --apache
```
y para nginx:
```
sudo certbot certonly --nginx
```

### Validar con servidor web interno de `certbot`

`certbot` puede utilizar un servidor web interno (_standalone_) para validar el
_challenge_ HTTP que le presenta el servidor ACME para obtener los certificados.

Esto sirve para generar certificados en servidores que no son web (por ejemplo
servidores de mail) o en servidores web que no son apache o nginx.

Para utilizarlo, hay que asegurarse de que no esté corriendo otro servidor en
el port 80 al momento de pedir o renovar los certificados.

La invocación es:
```
sudo certbot certonly --standalone --domain servidorweb.example.com
```

### Crear certificados en forma no-interactiva

`certbot` tiene una opción `--non-interactive` (o `-n`). Para que esta opción
funcione, hay que utilizar también la opción `--agree-tos` para aceptar los
términos del servicio, y la opción `--email` (o `-m`) para especificar la
dirección de mail para pasarle al servidor para recibir notificaciones de
eventuales problemas.

También hay que especificar el nombre (o nombres) de dominio para el que se
solicita el certificado, con la opción `--domain` (o `--domains` o `-d`).

Para especificar múltiples nombres de dominio se puede utilizar la opción varias
veces, cada vez con un nombre, o una vez, con una lista de nombres de dominios
separadas por comas.

Una invocación posible sería:
```
sudo certbot certonly --apache --non-interactive --agree-tos \
    --email hostmaster+tls-certbot_${HOSTNAME}@example.net \
    --domain servidorweb.example.com
```

## Listar los certificados

El comando `sudo certbot certificates` lista todos los certificados en el
sistema, con los nombres de dominio, tipo de clave, fecha de expiración y paths.

## Renovar certificados

`certbot` en general se configura para [renovar
automáticamente](https://certbot.eff.org/docs/using.html#automated-renewals).
En particular, en los Debian y Ubuntu modernos lo hace con un _timer_ de
`systemd`.

Para ver los _timers_ activos en `systemd` se puede usar el comando:
```
sudo systemctl list-timers
```

Allí debería aparecer una unidad `snap.certbot.renew.timer` o similar listando
el servicio que activa, cuándo fue la última vez que lo activo y cuándo la
próxima que lo activará.

Para [renovar](https://certbot.eff.org/docs/using.html#renewing-certificates)
certificados se utiliza el comando **`certbot renew`**.

Si se lo invoca sin más opciones, va a revisar todos los certificados e
intentará renovar todos aquellos a los que le falte **menos de 30 días** para
la fecha de expiración (los certificados Let's Encrypt se emiten por 90 días).

También se puede forzar la renovación aunque falten más de 30 días para la fecha
de expiración utilizando la opció `--force-renewal`.

Si se desea renovar un certificado en particular, hay que usar la opción
`--cert-name` con el [nombre del certificado](#nombre-del-certificado) que se
quiere renovar.

## Revocar un certificado

Para [revocar un
certificado](https://certbot.eff.org/docs/using.html#revoking-certificates) se
utiliza el comando
```
sudo certbot revoke --cert-name <nombre-del-certificado>
```

Alternativamente, en lugar de la opción `--cert-name` se puede utilizar la
opción `--cert-path` y pasar el path completo del archivo `cert.pem`.

Opcionalmente, se puede pasar la opción `--reason` que acepta los valores
`unspecified` (el default), `keycompromise`, `affiliationchanged`, `superseded`
y `cessationofoperation`.

Luego de revocar el certificado, `certbot` preguntará si se quiere borrar el
certificado (y todas las versiones anteriores) o dejarlo. Para evitar esta
interacción, se puede usar la opción `--delete-after-revoke` para borrarlo
automáticamente después de revocarlo o `--no-delete-after-revoke` para
mantenerlo.

Si se revoca un certificado pero _no_ se lo borra, el mismo será renovado la
próxima vez que se ejecute un `certbot renew` (esto normalmente se ejecuta una
o dos veces al día automáticamente).

Si se revocó un certificado porque se sospecha que se pudo haber filtrado la
clave privada, pero se lo desea seguir usando, lo mejor es no borrar el
certificado y luego renovarlo inmediatamente (lo que generará una nueva clave
privada y un nuevo certificado):
```
NOMBRE=<nombre-del-certificado>
sudo certbot revoke --cert-name ${NOMBRE} --reason keycompromise \
    --no-delete-after-revoke
sudo certbot renew --cert-name ${NOMBRE}
```

## Borrar un certificado

**OJO**: Si se borra un certificado, esta acción es _irreversible_ (en el
sentido que la clave privada y el certificado borrado no se pueden volver a
conseguir). Sí es posible emitir un _nuevo_ certificado con los mismos
parámetros.

También, **_antes_** de borrar un certificado, hay que revisar bien que no haya
ninguna referencia a él en algún servicio que esté ejecutándose (como apache,
nginx, postfix u otro).

Antes de borrar un certificado en particular, se pueden ejecutar estos comandos
para ver si hay alguna referencia en algún archivo de configuración bajo `/etc`
(ojo que también _podría_ haber alguna configuración en _otro_ lado que lo
utilice):
```
NOMBRE=<nombre-del-certificado>
sudo grep --dereference-recursive /etc/letsencrypt/live/${NOMBRE} /etc | \
  grep --invert-match ^/etc/letsencrypt
```
Antes de borrar el certificado, modificar los archivos de configuración de los
servicios que lo referencian para que no lo utilicen y reiniciar esos servicios.

Un certificado se puede borrar sin revocar (suponiendo que la clave no haya sido
compromentida). Sin embargo, como la CA no se entera de este borrado, cuando el
certificado (borrado) esté por expirar, la CA enviará mensajes a la dirección de
mail utilizada cuando el mismo se creó avisando esto. Por eso conviene [revocar
el certificado](#revocar-un-certificado) antes de borrarlo.

Para [borrar un
certificado](https://certbot.eff.org/docs/using.html#deleting-certificates) se
utiliza el comando
```
sudo certbot delete
```

Esto muestra la lista de certificados y permite elegir qué certificado borrar.

Alternativamente, se puede especificar el certificado en la línea de comandos:
```
sudo certbot delete --cert-name <nombre-del-certificado>
```

## _Hooks_

Los comandos `run`, `certonly` y `renew` de `certbot` soportan las opciones
`--pre-hook`, `--post-hook` y `--deploy-hook`.

* `--pre-hook` recibe como argumento un comando que ejecutará antes de un
intento por obtener o renovar un certificado (por ejemplo, para detener un
servicio antes de realizar la solicitud).

* `--post-hook` recibe como argumento un comando que ejecutará después de un
intento por obtener o renovar un certificado (por ejemplo, para reiniciar el
servicio después de realizar la solicitud).

* `--deploy-hook` recibe como argumento un comando que ejecutará después de
_obtener_ un certificado (por ejemplo para recargar la configuración de un
servicio que usa el certificado).

Si un `certbot renew` se aplica a múltiples certificados y todos tienen el mismo
`--pre-hook`, el _hook_ se ejecutará una sola vez antes de intentar renovar el
primer certificado.

Si un `certbot renew` se aplica a múltiples certificados y todos tienen el mismo
`--post-hook`, el _hook_ se ejecutará una sola vez después de intentar renovar
el último certificado.

Si un `certbot renew` se aplica a múltiples certificados, el `--deploy-hook` se
ejecutará _cada_ vez que se obtenga un certificado. En este caso, en la variable
de entorno `${RENEWED_LINEAGE}` estará el path completo al subdirectorio donde
están los archivos correspondientes al certificado obtenido (por ejemplo
`/etc/letsencrypt/live/servidorweb.example.com`) y la variable de entorno
`${RENEWED_DOMAINS}` tendra el o los nombres de dominio contenidos en el
certificado (si es más de uno, los nombres están separados por un espacio en
blanco).

## "_Nombre_" del certificado

`certbot` utiliza internamente un "_nombre_" para cada certificado que solicita.
Este nombre se usa para identificar al certificado localmente y es el nombre que
se utiliza como nombre para los directorios que contienen certificados y claves
bajo `/etc/letsencrypt/live` y bajo `/etc/letsencrypt/live`.

Por default, `certbot` utiliza el primer nombre de dominio del certificado como
_nombre_ de ese certificado (e.g: dominio.example.com). Si ya hay en el sistema
un certificado con ese nombre, les agrega un número detrás (e.g:
dominio.example.com-001).

El _nombre_ se asigna al crear el certificado (no cambia con las renovaciones).

Se puede usar la opción **`--cert-name`** con `certbot run` o `certbot certonly`
para poner un nombre arbitrario al certificado que se solicita.

Para ver el _nombre_ de todos los certificados en el sistema, se pueden [listar
los certificados](#listar-los-certificados) con el comando `sudo certbot
certificates` y allí se listarán todos los certificados, cada uno con su nombre.

## Otras opciones

Todas las opciones de línea de comandos del `certbot` están en
https://certbot.eff.org/docs/using.html#certbot-command-line-options

### Ejecuciones de prueba

Por un lado, la opción **`--dry-run`** simula el comando (ya sea de creación o
renovación del certificado), pero no modifica nada.

Por otro lado, la opción `--test-cert` (o `--staging`) utiliza una autoridad de
certificación (CA) alternativa que emite certificados que **no son válidos** (ya
que la CA no tiene una cadena de certificación hacia una CA raíz aceptada).
Los certificados obtenidos con esta opción son correctos (en cuanto al formato
X.509), pero no serán aceptados por los clientes automáticamente (el navegador
dará un error de certificado inválido).

La ventaja de estos certificados es que el _rate limiting_ de la CA es mucho
menor y se pueden solicitar y renovar múltiples veces, con o sin errores, antes
de que la CA empiece generar un error automático por exceso de uso.

### Tipo de clave

Las versiones actuales (2023-07) de `certbot` generan por default un par de
claves utilizando el Algoritmo de Firma Digital de Curva Elíptica (ECDSA).
Alternativamente, puede generar pares de claves utilizando el algoritmo RSA.
En algunos casos en que se esperen clientes (o en el caso de un servidor de
mail) puede ser conveniente utilizar un certificado RSA en lugar del default
ECDSA.

La opción 
[**`--key-type`**](https://certbot.eff.org/docs/using.html#rsa-and-ecdsa-keys)
que acepta las opciones `rsa` o `ecdsa` (default) selecciona el tipo de clave.

Cuando se especifica una clave **RSA** se puede especificar el tamaño de la
clave en bits con la opción **`--rsa-key-size`**. El default es `2048` (y hoy en
día los navegadores podrían rechazar sitios con certificados cuya clave RSA es
más corta que eso).

Cuando se especifica una clave **ECDSA** se puede especificar la curva elíptica
específica y sus parámetros con la opción **`--elliptic-curve`**. El default es
`secp256r1`. La otra opción soportada es `secp384r1`.

# Algunos ejemplos

## Generación de certificados para Postfix

Armamos en `/etc/postfix` un directorio para poner las claves y certificados
TLS:
```
sudo mkdir --mode=0700 --verbose --parents /etc/postfix/certs
```

Vamos a intentar obtener un certificado válido para el nombre definido como
`myhostname` en la configuración de postfix y también para el hostname del
equipo (que podría ser el mismo o no).

Primero hacemos una corrida "en vacío" para verificar que funciona crear un
certificado:
```
sudo certbot certonly --non-interactive --agree-tos \
    --email hostmaster+tls-certbot-${HOSTNAME}@`sudo postconf -h mydomain` \
    --test-cert --dry-run --standalone \
    --domains `sudo postconf -h myhostname`,${HOSTNAME}
```

Si la corrida funcionó bien, se puede emitir un certificado con clave RSA  con
los siguientes comandos:
```
CERTNAME="CERT_`sudo postconf -h myhostname`_RSA"
KEYOPTIONS="--key-type rsa --rsa-key-size 4096"

DOMAINS="`sudo postconf -h myhostname`,${HOSTNAME}"
EMAIL="hostmaster+tls-certbot-${HOSTNAME}@`sudo postconf -h mydomain`"
POSTFIXCERTDIR="/etc/postfix/certs"

sudo certbot certonly --non-interactive --agree-tos --email ${EMAIL} \
    --standalone --cert-name ${CERTNAME} --domains ${DOMAINS} ${KEYOPTIONS} \
    --deploy-hook "cat \${RENEWED_LINEAGE}/privkey.pem \${RENEWED_LINEAGE}/fullchain.pem \
      > ${POSTFIXCERTDIR}/${CERTNAME}.pem && chmod 0600 ${POSTFIXCERTDIR}/${CERTNAME}.pem \
      && systemctl reload postfix"
```

y un certificado con clave ECDSA con los siguientes comandos:
```
CERTNAME="CERT_`sudo postconf -h myhostname`_ECDSA"
KEYOPTIONS="--key-type ecdsa --elliptic-curve secp384r1"

DOMAINS="`sudo postconf -h myhostname`,${HOSTNAME}"
EMAIL="hostmaster+tls-certbot-${HOSTNAME}@`sudo postconf -h mydomain`"
POSTFIXCERTDIR="/etc/postfix/certs"

sudo certbot certonly --non-interactive --agree-tos --email ${EMAIL} \
    --standalone --cert-name ${CERTNAME} --domains ${DOMAINS} ${KEYOPTIONS} \
    --deploy-hook "cat \${RENEWED_LINEAGE}/privkey.pem \${RENEWED_LINEAGE}/fullchain.pem \
      > ${POSTFIXCERTDIR}/${CERTNAME}.pem && chmod 0600 ${POSTFIXCERTDIR}/${CERTNAME}.pem \
      && systemctl reload postfix"
```

Esto dejó los siguientes archivos del certificado RSA en la carpeta
`/etc/letsencrypt/live/CERT_<nombre-del-server>_RSA/` y los del certificado
ECDSA en la carpeta `/etc/letsencrypt/live/CERT_<nombre-del-server>_ECDSA/`.
En cada carpeta están los siguientes archivos:
* `cert.pem`: el certificado para el server
* `chain.pem`: la cadena de certificados desde el emisor hasta la raíz
* `fullchain.pem`: el certificado para el server junto con la cadena de
certificados hasta la raíz
* `privkey.pem`: la clave privada del server
Estos son links simbólicos a las últimas versiones de estos archivos que se
mantienen en `/etc/letsencrypt/archive/CERT_<nombre-del-server>_ECDSA/`.

Adicionalmente, la opción `--deploy-hook` armó (en la carpeta
`/etc/postfix/certs`) los archivos:
* **`CERT_<nombre-del-server>_RSA.pem`**
* **`CERT_<nombre-del-server>_ECDSA.pem`**

Cada uno de estos tiene la clave privada del server, seguida por el certificado
para el server, seguida por la cadena de certificados hasta la raíz (esto es, la
concatenación de `privkey.pem` y `fullchain.pem` que es el formato que prefieren
las versiones nuevas de Postfix (desde Postfix 3.4 en adelante).

Finalmente, se hace un `reload` del servicio `postfix` para que lea los nuevos
certificados.

Estos certificados [se renuevan
automáticamente](LetsencryptCertbot.md#renovar-certificados) (y esto incluye la
corrida del `deploy-hook`).

## Revocar y borrar todos los certificados de un servidor que se migró

Asegurarse de que ningún servicio (apache, nginx, postfix, etc) esté usando
un certificado de estos:
```
sudo grep --dereference-recursive /etc/letsencrypt/live/ /etc | \
  grep --invert-match ^/etc/letsencrypt
```
Si acá no apareció nada, es una buena señal. Si apareció algo, revisar los
archivos que aparecen.

**CUIDADO**: Esto va a revocar y borrar todos los certificados que haya en el
sistema **sin preguntar nada**:
```
for NOMBRE in `sudo ls /etc/letsencrypt/live/` ; do
  [ ${NOMBRE} == "README" ] && continue
  sudo certbot revoke --cert-name ${NOMBRE} --reason superseded \
    --delete-after-revoke --non-interactive
done
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
