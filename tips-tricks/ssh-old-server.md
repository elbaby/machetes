# SSH - Diversos problemas al conectarse a equipos viejos

## Pide password al conectarse aun cuando el equipo tiene la clave pública del
cliente

Los nuevos clientes SSH piden password al conectarse a un server viejo aun
cuando el servidor tiene la clave pública del cliente.

El error que se ve al usar la opción `-v` de `ssh` es:
```
debug1: send_pubkey_test: no mutual signature algorithm
```

La solución la encontré en el [AskUbuntu de 
StackExchange](https://askubuntu.com/questions/1404049/ssh-without-password-does-not-work-after-upgrading-from-18-04-to-22-04):

Para un uso muy eventual, agregar la opción `-o PubkeyAcceptedKeyTypes=+ssh-rsa`
en la invocación del cliente:
```
ssh -o PubkeyAcceptedKeyTypes=+ssh-rsa user@old-server
```

La otra opción es agregarla en el archivo de configuración personal del cliente
`~/.ssh/config`:
```
PubkeyAcceptedKeyTypes +ssh-rsa
```

Para más seguridad en general, esto conviene configurarlo _únicamente_ para los
hosts específicos que lo requieran. Lamentablemente, si al host uno se conecta
a veces por dirección IP, a veces por nombre corto y a veces por FQDN, es
necesario poner cada una de estas variantes en la línea `Host` del
`~/.ssh/config`:
```
Host 10.10.20.20 old-server old-server.example.net
	PubkeyAcceptedKeyTypes +ssh-rsa
```

## No se conecta y da un error respecto del tipo de clave de host

El error que se ve es similar a este:
```
$ ssh user@old-server.example.com

Unable to negotiate with old-server.example.com port 22: no matching host key type found. Their offer: ssh-dss,ssh-rsa
```

La solución también la encontré en el [AskUbuntu de 
StackExchange](https://askubuntu.com/questions/836048/ssh-returns-no-matching-host-key-type-found-their-offer-ssh-dss):

```
ssh -o HostKeyAlgorithms=+ssh-rsa,ssh-dss user@old-server.example.com
```

O agregando en el `~/.ssh/config`:
```
Host 10.10.20.20 old-server old-server.example.net
	HostKeyAlgorithms +ssh-rsa,ssh-dss
```

## No se conecta y da un error respecto del método de intercambio de claves

El error que se ve es similar a este:
```
$ ssh user@old-server.example.com

Unable to negotiate with old-server.example.com port 22: no matching key exchange method found. Their offer: diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1
```

La solución a esto la saqué por analogía con los casos anteriores:

```
ssh -o KexAlgorithms=+diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1 user@old-server.example.com
```

O agregando en el `~/.ssh/config`:
```
Host 10.10.20.20 old-server old-server.example.net
	KexAlgorithms +diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1
```

## No se conecta y da un error respecto del cifrado

El error que se ve es similar a este:
```
$ ssh user@old-server.example.com

Unable to negotiate with old-server.example.com port 22: no matching cipher found. Their offer: aes128-cbc,3des-cbc,des-cbc
```

La solución a esto la saqué por analogía con los casos anteriores:

```
ssh -o Ciphers=+aes256-cbc,aes128-cbc user@old-server.example.com
```

O agregando en el `~/.ssh/config`:
```
Host 10.10.20.20 old-server old-server.example.net
	Ciphers +aes256-cbc,aes128-cbc
```

## No se conecta y da un error en `libcrypto`

El error que se ve es similar a este:
```
$ ssh user@old-server.example.com

ssh_dispatch_run_fatal: Connection to UNKNOWN port 65535: error in libcrypto
```
(esto me ocurrió conectándome desde un RedHat 9 a un RedHat6)

La solución la encontré en el [Serverfault de 
StackExchange](https://serverfault.com/questions/1125843/error-in-libcrypto-connecting-rhel-9-server-to-centos-6-via-sftp-ssh)
combinando las dos primeras respuestas:

Por un lado, en el `~/.ssh/config` hay que agregar lo siguiente:
```
Host 10.10.20.20 old-server old-server.example.net
	KexAlgorithms=+diffie-hellman-group14-sha1
	MACs=+hmac-sha1
	HostKeyAlgorithms=+ssh-rsa
	PubkeyAcceptedKeyTypes=+ssh-rsa
	PubkeyAcceptedAlgorithms=+ssh-rsa
```

_Además_ hay que armar una configuración de OpenSSL que permita parámetros
inseguros _sin cambiar_ la configuración general de OpenSSL.

Una forma simple de hacerlo es armar un archivo `~/.ssh/opensslinsecure.cnf`
que incluya la configuración estándar más los parámetros inseguros:
```
.include /etc/ssl/openssl.cnf
[openssl_init]
alg_section = evp_properties
[evp_properties]
rh-allow-sha1-signatures = yes
```

Ahora, para invocar ssh utilizando la configuración insegura hay que usar:
```
OPENSSL_CONF=${HOME}/.ssh/opensslinsecure.cnf ssh user@old-server.example.com
```

## Configuración completa

Es común que varios de estos problemas aparezcan en los mismos hosts. Lo más
conveniente es tener una o más secciones que combinen las opciones requeridas
por cada conjunto de equipos en el `~/.ssh/config`:

```
Host 10.11.111.33 mikrotik-router mikrotik-router.example.net
	HostKeyAlgorithms=+ssh-rsa,ssh-dss
	PubkeyAcceptedKeyTypes=+ssh-rsa

Host 10.11.111.77 old-hp-switch-01 old-hp-switch-01.example.net 10.11.111.78 old-hp-switch-02 old-hp-switch-02.example.net
	HostKeyAlgorithms=+ssh-rsa,ssh-dss
	PubkeyAcceptedKeyTypes=+ssh-rsa
	Ciphers=+aes256-cbc,aes128-cbc
	KexAlgorithms=+diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1

Host 10.11.111.6 oldhat6 oldhat6.example.net
	KexAlgorithms=+diffie-hellman-group14-sha1
	MACs=+hmac-sha1
	HostKeyAlgorithms=+ssh-rsa
	PubkeyAcceptedKeyTypes=+ssh-rsa
	PubkeyAcceptedAlgorithms=+ssh-rsa
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
