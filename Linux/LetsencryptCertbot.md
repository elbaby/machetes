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

## Configurar el nombre de dominio en el DNS público

Todos los nombres de dominio para los que se van a solicitar el certificado
deben estar configurados en el DNS.

Asegurarse de que los nombres estén correctos en _todos_ los servidores
autoritativos para los nombres.

## Configurar el firewall

Es necesario que el port 80 (default del servicio HTTP) esté abierto en el
servidor.

Si al servidor se llega a través de un NAT o similar, es importante que el port
80 de la dirección IP pública que corresponde al nombre solicitado se conecte al
servidor donde se va a ejecutar el `certbot` (en caso contrario habrá que
utilizar un _challenge_ vía DNS).

Si el servidor es el que se va a utilizar para brindar el servicio HTTPS,
también hay que abrir el port 443.

## Uso simple (plugins para servidores)

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

### Ejemplo de interacción

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

### Timer para renovar los certificados

La instalación de los certificados debe haber configurado un timer en `systemd`
para renovar periódicamente los certificados en
`/etc/systemd/system/snap.certbot.renew.timer`.

Para verificar el funcionamiento de la renovación (sin renovar los certificados)
se puede correr el siguiente comando:
```
sudo certbot renew --dry-run
```

## Configurar los servidores manualmente

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

## Validar con servidor web interno de `certbot`

`certbot` puede utilizar un servidor web interno (_standalone_) para validar el
_challenge_ HTTP que le presenta el servidor ACME para obtener los certificados.

Esto sirve para generar certificados en servidores que no son web (por ejemplo
servidores de mail) o en servidores web que no son apache o nginx.

Para utilizarlo, hay que asegurarse de que no esté corriendo otro servidor en
el port 80 al momento de pedir o renovar los certificados.

La invocación es:
```
sudo certbot certonly --standalone
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
