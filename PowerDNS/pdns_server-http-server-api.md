# Servidor web http para monitoreo y API REST

https://doc.powerdns.com/authoritative/http-api/index.html

PowerDNS tiene un servidor web que expone [información de funcionamiento del
servidor autoritativo](https://doc.powerdns.com/authoritative/performance.html).

Esta información está en un formato para "consumo humano" (en la raíz **`/`**
del servidor web) y también en un formato apto para procesamiento (en particular
siguiendo el [formato de exposición de
Prometheus](https://github.com/prometheus/docs/blob/main/content/docs/instrumenting/exposition_formats.md))
en el path **`/metrics`** del servidor web.

Adicionalmente, se puede habilitar una [API 
REST](https://doc.powerdns.com/authoritative/http-api/index.html#endpoints-and-objects-in-the-api)
que se expone en **`/api/v1`** y permite consultar los objetos y configurarlos 
en el servidor autoritativo.

## (In)seguridad

El servidor web de monitoreo sólo permite su protección a través de una clave
(única) y la API sólo permite su protección con una API-key (también única).

Por default, el servidor escucha en la interfaz local (en el puerto 8081) y 
sólo permite conexiones locales.

Dado que el servidor web tampoco soporta conexiones seguras https, si se desea
permitir acceso remoto, en vez de cambiar la interfaz y permitir conexiones
remotas, instalar localmente un proxy reverso (como [Caddy](/Caddy)) y 
configurar allí conexiones más seguras.

Ponemos las configuraciones para el servidor web en un archivo de configuración
modular en **`/etc/powerdns/pdns.d/http-server-api.conf`**:                                
## Instalación de caddy server

Seguir las instrucciones de instlación de [Caddy](/Caddy)


## Configuración de servidor web para monitoreo y API

https://doc.powerdns.com/authoritative/http-api/index.html

Configuramos los siguientes parámetros en un archivo de configuración
**`/etc/powerdns/pdns.d/http-server-api.conf`**:
* [webserver](https://doc.powerdns.com/authoritative/settings.html#webserver)
* [api](https://doc.powerdns.com/authoritative/settings.html#setting-api)
* [webserver-password](https://doc.powerdns.com/authoritative/settings.html#webserver-password)
* [api-key](https://doc.powerdns.com/authoritative/settings.html#setting-api-key)
* [webserver-address](https://doc.powerdns.com/authoritative/settings.html#webserver-address)
* [webserver-port](https://doc.powerdns.com/authoritative/settings.html#webserver-port)
* [webserver-allow-from](https://doc.powerdns.com/authoritative/settings.html#webserver-allow-from)

```
sudo tee /etc/powerdns/pdns.d/http-server-api.conf <<EOF
webserver=yes
api=yes
api-key=UnaClaveMuyMuySecretaParaLaAPI
webserver-password=OtraClaveMuyMuySecretaParaElWebServer
webserver-address=127.0.53.80
webserver-port=5380
webserver-allow-from=127.0.0.0/8,::1
EOF
```

Ajustamos los permisos del archivo de configuración:

```
sudo chmod 0640 /etc/powerdns/pdns.d/http-server-api.conf
sudo chgrp pdns /etc/powerdns/pdns.d/http-server-api.conf
```

**IMPORTANTE**: ¡Cambiar los valores de las claves en `api-key` y 
`webserver-password` en el archivo `/etc/powerdns/pdns.d/http-server-api.conf`!

Y después reiniciamos el servidor powerdns:
```
sudo systemctl restart pdns.service
```

## Configuración de caddy

Agregar a la configuración por defecto de Caddy (que simplemente muestra una
página estática en `/usr/share/caddy` si se llega por dirección IP o por nombre
desconocido) lo siguiente


```
###################################################
# PowerDNS Authoritative Server webserver and API #
###################################################
# Acá abajo debe ir el nombre del server:
pdns-server.example.com
{
	# Si el nombre de dominio de arriba NO ES público (es decir, si no está
	# en el DNS global y resuelve a la IP del servidor en el que estamos
	# instalando todo), hay que DESCOMENTAR la línea siguiente para que
	# use una CA interna para firmar los certificados.
	# Obviamente, a menos que se agregue el certificado de esa CA en los
	# navegadores donde se usará, recibirá un aviso de página insegura.
	#tls internal

	# Enviar requests al webserver publicado por PowerDNS
	reverse_proxy * 127.0.53.80:5380
}
```
Activar la nueva configuración del caddy:
```
sudo systemctl reload caddy.service
```

Si el nombre DNS del servidor es público, la primera vez que alguien se conecte
a él, el caddy solicitará certificados en letsencrypt.org y los instalará. Y
las conexiones http (sin encriptar) al puerto 80, serán redirigidas 
automáticamente a https en el puerto 443 utilizando estos certificados.

Si se utiliza `tls_internal` (porque el nombre no está en el DNS público), luego
de la primera vez que uno se conecte al servidor, se generará automáticamente
una autoridad de certificación (CA) raíz y una intermedia locales.

El certificado de la CA raíz queda en 
`/var/lib/caddy/.local/share/caddy/pki/authorities/local/root.crt` y se puede
usar para instalar en los navegadores para que no dé avisos de error de 
certificados.

Es **muy** importante que el resto de los archivos bajo 
`/var/lib/caddy/.local/share/caddy/pki/` no puedan ser vistos por nadie más
(en particular, los que terminan en `.key`).

Ahora al entrar en un navegador a https://pdns-server.example.com (o como se
llame el servidor) pedirá el login a nivel http (se pone cualquier nombre de
usuario y la clave configurada en **`webserver-password`**.

El _endpoint_ de la API será https://pdns-server.example.com/api/v1 con el
nombre correcto del servidor (hay que configurar la **`api-key`** que se 
guardó en la configuración del servidor)


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
