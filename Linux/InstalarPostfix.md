# Instalar y configurar un servidor Postfix en Ubuntu


### Bibliografía

* [Postfix Documentation](https://www.postfix.org/documentation.html)
* [Postfix Basic
Configuration](https://www.postfix.org/BASIC_CONFIGURATION_README.html)
* [Postfix Standard Configuration
Examples](https://www.postfix.org/STANDARD_CONFIGURATION_README.html)
* [Postfix Architecture Overview](https://www.postfix.org/OVERVIEW.html)
* [Postfix Address Classes](https://www.postfix.org/ADDRESS_CLASS_README.html)
* [Postfix Address
Rewriting](https://www.postfix.org/ADDRESS_REWRITING_README.html)
* [Postfix Virtual Domain Hosting
Howto](https://www.postfix.org/VIRTUAL_README.html)
* [Postfix Lookup Table Overview](https://www.postfix.org/DATABASE_README.html)
* [Postfix SASL Howto](https://www.postfix.org/SASL_README.html)
* [Postfix TLS Support](https://www.postfix.org/TLS_README.html)
* [Postfix SMTP relay and access
control](https://www.postfix.org/SMTPD_ACCESS_README.html)
* [Postfix Bottleneck Analysis](https://www.postfix.org/QSHAPE_README.html)
  * [Postfix queue
directories](https://www.postfix.org/QSHAPE_README.html#queues)
* [Postfix Performance Tuning](https://www.postfix.org/TUNING_README.html)
* [Postfix Debugging Howto](https://www.postfix.org/DEBUG_README.html)

## Instalar el paquete Postfix de los repositorios estándar

2023-07-17: Esto está probado con Ubuntu 22.04

```
sudo apt-get install postfix postfix-pcre
```
Respuestas y configuraciones al instalador:

* General mail configuration type: **`Internet Site`**
* System mail name: el nombre de dominio que usarán los mensajes que salen del
servidor (si los mails son usuario@example.com, entonces acá va
**`example.com`** (y _no_ `correo.example.com`)


## Configuraciones básicas

Editar `/etc/postfix/main.cf`

En la variable [`myhostname`](https://www.postfix.org/postconf.5.html#myhostname)
poner el nombre con el que se quiere que se identifique el server al atender una
conexión (por default es el hostname del equipo, pero podríamos querer que sea
otro nombre).

En la variable
[`mydestination`](https://www.postfix.org/postconf.5.html#mydestination) agregar
otros dominios para los que se quiera aceptar mail.

Otra variante es [utilizar un archivo con expresiones regulares compatibles con
perl (pcre) con una línea por cada expresión regular que acepte
dominios](https://serverfault.com/questions/133190/host-wildcard-subdomains-using-postfix).

Por ejemplo, en `/etc/main.cf` poner:
```
mydestination = pcre:/etc/postfix/mydestination.pcre
```
y en `/etc/postfix/mydestination.pcre` poner:
```
/^myhost\.example\.com$/    ACCEPT
/^correo\.example\.com$/    ACCEPT
/^localhost\.example\.com$/ ACCEPT
/^localhost$/               ACCEPT
/^example\.com$/            ACCEPT
/^.*\.example\.net$/        ACCEPT
/^example\.net$/            ACCEPT
```
En lugar de "ACCEPT" puede ir cualquier cosa, ya que lo único que se verifica
es que la clave exista (que la dirección matchee la expresión regular).

Esto acepta mails para direcciones:
* `<localpart>@myhost.example.com`
* `<localpart>@correo.example.com`
* `<localpart>@localhost.example.com`
* `<localpart>@localhost`
* `<localpart>@example.com`
* `<localpart>@<cualquiercosa>.example.net`
* `<localpart>@example.net`

### Aliases

Postfix utiliza la tabla en
[`/etc/aliases`](https://www.postfix.org/aliases.5.html) para [reescribir las
direcciones](https://www.postfix.org/ADDRESS_REWRITING_README.html) destino
de _únicamente_ los mensajes que se envían utilizando el servicio de despacho
[`local`](https://www.postfix.org/local.8.html).

Esto está configurado en la variable
[`alias_maps`](https://www.postfix.org/postconf.5.html#alias_maps) del archivo
de configuración.

Las búsquedas para la reescritura de direcciones son recursivas.

El formato de la base de datos de alias local está definido en
[`aliases(5)`](https://www.postfix.org/aliases.5.html)

La tabla de aliases que efectivamente se utiliza es una tabla hasheada en
`/etc/aliases.db`. Para generarla, hay que utilizar el comando
[`newaliases`](https://www.postfix.org/newaliases.1.html):
```
sudo newaliases
```

### Virtual aliases

Para [reescribir las
direcciones](https://www.postfix.org/ADDRESS_REWRITING_README.html) de cualquier
tipo (tanto locales como remotas o virtuales) hay que crear una [tabla de alias
virtuales](https://www.postfix.org/virtual.5.html).

Esa tabla hay que agregarla en la configuración en la variable
[`virtual_alias_maps`](https://www.postfix.org/postconf.5.html#virtual_alias_maps)
del archivo de configuración.

Las búsquedas para la reescritura de direcciones son recursivas.

El formato de la tabla de alias virtual está definido en
[`virtual(5)`](https://www.postfix.org/virtual.5.html)

La tabla de aliases virtuales que efectivamente se utiliza es una tabla hasheada
con el mismo nombre que el archivo de texto y extensión `.db`. Para generarla,
hay que utilizar el comando [`postmap`](https://www.postfix.org/postmap.1.html).

Si la tabla está en `/etc/postfix/virtual_aliases` el comando será:
```
sudo postmap /etc/postfix/virtual_aliases
```

## Usar TLS con certificados letsencrypt

### Directorio para certificados

Armamos en `/etc/postfix` un directorio para poner las claves y certificados
TLS:
```
sudo mkdir --mode=0700 --verbose --parents /etc/postfix/certs
```

### Instalar `certbot`

Seguir las instrucciones para [instalar
certbot](LetsencryptCertbot.md#instalación-de-certbot-en-debian-o-ubuntu) en la
[página de certbot de esta wiki](LetsencryptCertbot.md).


### Obtener certificados

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

Cada uno de estos tiene la clave privada
del server, seguida por el certificado para el server, seguida por la cadena de
certificados hasta la raíz (esto es, la concatenación de `privkey.pem` y
`fullchain.pem` que es el formato que prefieren las versiones nuevas de Postfix
(desde Postfix 3.4 en adelante).

Finalmente, se hace un `reload` del servicio `postfix` para que lea los nuevos
certificados.

Estos certificados [se renuevan
automáticamente](LetsencryptCertbot.md#renovar-certificados) (y esto incluye la
corrida del `deploy-hook`).

## Configurar los certificados para _recibir_ mail con `smtpd`

Ver: https://www.postfix.org/TLS_README.html#server_tls

Editar el archivo de configuración **`/etc/postfix/main.cf`**.

* _Borrar_ (o comentar) si existen las líneas con las variables
[`smtpd_tls_key_file`](https://www.postfix.org/postconf.5.html#smtpd_tls_key_file) y
[`smtpd_tls_cert_file`](https://www.postfix.org/postconf.5.html#smtpd_tls_cert_file)
(normalmente en Debian y Ubuntu apuntan a una clave privada 
`/etc/ssl/private/ssl-cert-snakeoil.key` y un certificado autofirmado 
`/etc/ssl/certs/ssl-cert-snakeoil.pem` respectivamente).
* _Agregar_ la variable
[`smtpd_tls_chain_files`](https://www.postfix.org/postconf.5.html#smtpd_tls_chain_files)
que acepta un archivo o una lista de archivos (separados por comas o espacio en
blanco) consistentes cada uno en una clave privada seguida del certificado del
sitio y la cadena de verificación.
* _Agregar_ (o verificar que está configurada) la variable
[`smtpd_tls_security_level`](https://www.postfix.org/postconf.5.html#smtpd_tls_security_level)
con el valor **`may`** para usar TLS oportunístico (**no** usar el valor
`encrypt` ya que no se aceptarían conexiones de clientes que no utilizan TLS
y la Internet está _llena_ de servidores que no lo utilizan)
* Configurar una [cache de sesiones TLS para el
servidor](https://www.postfix.org/TLS_README.html#server_tls_cache) usando la
variable
[`smtpd_tls_session_cache_database`](https://www.postfix.org/postconf.5.html#smtpd_tls_session_cache_database)
* Configurar explícitamente la variable
[`smtpd_tls_loglevel`](https://www.postfix.org/postconf.5.html#smtpd_tls_loglevel)
con el [nivel de _logging_ deseado para
TLS](https://www.postfix.org/TLS_README.html#server_logging). Este valor debería
estar entre `0` y `2`. `3` únicamente si hay errores de negociación TLS,
mientras se lo revisa. El nivel `4` no debería usarse nunca.
```
# smtpd server TLS parameters
smtpd_tls_chain_files=/etc/postfix/certs/CERT_<nombre-del-server>_ECDSA.pem
                      /etc/postfix/certs/CERT_<nombre-del-server>_RSA.pem
smtpd_tls_security_level=may
smtpd_tls_session_cache_database=btree:${data_directory}/smtpd_scache
smtpd_tls_loglevel=1
```

Recargar el servicio `postfix` para tomar la nueva configuración:
```
systemctl reload postfix
```

### Recibir conexiones TLS en el port SMTPS

Si, además de aceptar conexiones en el port SMTP 25 (que pueden utilizar
STARTTLS para iniciar el _handshake_ TLS) se desean aceptar conexiones TLS en
el port SMTPS 465 (que realizan el _handshake_ antes del HELO/EHLO), hay que
habilitar el servicio `smtps` en el archivo `/etc/postfix/master.cf`.

En general esa línea ya está en el archivo y simplemente hay que quitarle el `#`
del principio de la línea
```
# ==========================================================================
# service type  private unpriv  chroot  wakeup  maxproc command + args
#               (yes)   (yes)   (no)    (never) (100)
# ==========================================================================
smtps     inet  n       -       y       -       -       smtpd
```

Recargar el servicio `postfix` para tomar la nueva configuración:
```
systemctl reload postfix
```

## Configurar los certificados para _enviar_ mail con `smtp`

Ver: https://www.postfix.org/TLS_README.html#client_tls

Editar el archivo de configuración **`/etc/postfix/main.cf`**.

* [_No conviene_](https://www.postfix.org/TLS_README.html#client_cert_key)
configurar una clave privada y certificado del lado del cliente, por lo que hay
que asegurarse de que la variable
[`smtp_tls_chain_files`](https://www.postfix.org/postconf.5.html#smtp_tls_chain_files)
esté vacía
* Configurar la variable
[`smtp_tls_CApath`](https://www.postfix.org/postconf.5.html#smtp_tls_CApath) con
el path al directorio donde están los certificados de autoridades de
certificación (CA) aceptadas
* _Agregar_ (o verificar que está configurada) la variable
[`smtp_tls_security_level`](https://www.postfix.org/postconf.5.html#smtp_tls_security_level)
con el valor **`may`** para usar TLS oportunístico (**no** usar el valor
`encrypt` ya que no se aceptarían conexiones a servidores que no utilizan TLS
y la Internet está _llena_ de servidores que no lo utilizan)
* Configurar una [cache de sesiones TLS para el
cliente](https://www.postfix.org/TLS_README.html#client_tls_cache) usando la
variable
[`smtp_tls_session_cache_database`](https://www.postfix.org/postconf.5.html#smtp_tls_session_cache_database)
* Configurar la [reutilización de las conexiones TLS del
cliente](https://www.postfix.org/TLS_README.html#client_tls_reuse) usando la
variable
[`smtp_tls_connection_reuse`](https://www.postfix.org/postconf.5.html#smtp_tls_connection_reuse)
* Configurar explícitamente la variable
[`smtp_tls_loglevel`](https://www.postfix.org/postconf.5.html#smtp_tls_loglevel)
con el [nivel de _logging_ deseado para
TLS](https://www.postfix.org/TLS_README.html#client_logging). Este valor debería
estar entre `0` y `2`. `3` únicamente si hay errores de negociación TLS,
mientras se lo revisa. El nivel `4` no debería usarse nunca.
```
# smtp/lmtp client TLS parameters
smtpd_tls_chain_files=
smtp_tls_CApath=/etc/ssl/certs
smtp_tls_security_level=may
smtp_tls_session_cache_database=btree:${data_directory}/smtp_scache
smtp_tls_connection_reuse=yes
smtp_tls_loglevel=1
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
