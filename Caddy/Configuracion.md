# Configuraciones generales

* https://caddyserver.com/docs/getting-started
* https://caddyserver.com/docs/caddyfile
* https://caddyserver.com/docs/quick-starts/caddyfile
* https://caddyserver.com/docs/caddyfile/concepts


La configuración más simple se hace usando el archivo 
[**`/etc/caddy/Caddyfile`**](https://caddyserver.com/docs/caddyfile).

La instalación por defecto viene con un ejemplo que contesta las conexiones al 
port 80 usando la dirección IP o un nombre que no esté configurada en Caddy con
un servidor de archivos estático en `/usr/share/caddy`.

Un default más razonable para poner en un servidor que se llame 
caddy.example.net podría ser así:

```
# Global Options
{
	# El mail que mandamos al server ACME para registrar
	# los certificados 
	email   tls-ssl-master@example.net

	# El "hostname" que tomaremos por default si el cliente
	# no manda un SNI
	default_sni     caddy.example.net
	# Descomentar la línea de abajo para loguear
	#debug
}

# Usar para las conexiones por IP o con nombre desconocido
caddy.example.net,
:80 
{
	root * /var/www/caddy
	# Enable the static file server.
	file_server * {
		index	index.html
	}

}
```

## Proxy a un port local

* https://caddyserver.com/docs/quick-starts/reverse-proxy
* https://caddyserver.com/docs/quick-starts/https
* https://caddyserver.com/docs/caddyfile/directives/reverse_proxy

Si tenemos un servicio web que funciona en el mismo equipo en el port 12345 
podemos hacer un proxy reverso hacia el mismo con un nombre público (que debe 
estar registrado en el DNS público para que Caddy gestione un certificado TLS 
gratuito en letsencrypt.org usando el protocolo ACME).

Caddy también arma una redirección automática http -> https

```
servicio.example.net,
www.servicio.example.net
{
	reverse_proxy localhost:12345
}
```

## Proxy a otro equipo

Del mismo modo, se puede realizar un proxy reverso hacia otro equipo (por
ejemplo un equipo en la red interna que no tiene una dirección IP pública).
Se puede utilizar tanto un la dirección IP del equipo como un nombre que
el equipo donde está el caddy reconozca (ya sea por un DNS interno o en el
`/etc/hosts`).

Si no se especifica el port, se utiliza el port 80 (http).

```
servicio.example.net,
www.servicio.example.net
{
	reverse_proxy 10.20.30.40:8000
}

otroservicio.example.net,
www.otroservicio.example.net
{
	reverse_proxy server-interno.local
}
```

## Redirección

* https://caddyserver.com/docs/caddyfile/directives/redir

Alternativamente, se puede hacer que caddy realice una redirección en http (es
decir, informarle al cliente que debe cargar otro URL.

El URL _debe_ ser público, ya que lo tiene que resolver el cliente (por otra
parte, no tiene por qué ser en un servidor propio):

```
nuestro-guguel.example.net
{
	redir https://www.google.com/
}
```

## Matcheo de requests

* https://caddyserver.com/docs/caddyfile/matchers

Por default, se matchean todos los requests (`*`), pero se pueden especificar
distintos _matchers_.

### Matcheo _wildcard_ (`*`)

Matchea _todos_ los requests:
```
servicio.example.net
{
	reverse_proxy * localhost:8000
}
```
En general es optativo. Lo siguiente es equivalente:
```
servicio.example.net
{
	reverse_proxy localhost:8000
}
```

### Matcheo de ruta (path) (`/path`)

Los matcheos de path _deben_ comenzar con una barra (`/`).  Por ejemplo, esto va 
a hacer un proxy reverso de https://servicio.example.net/sitionuevo a 
localhost:8000/index.php
```
servicio.example.net
{
	reverse_proxy /sitionuevo localhost:8000/index.php
}
```

Los matcheos de path son exactos. Normalmente, lo que uno quiere hacer es un
matcheo por prefijo, para eso hay que agregar un (`*`):
```
servicio.example.net
{
	reverse_proxy /sitionuevo/* localhost:8000
}
```

### Matcheo con nombre (`@nombre`)

* https://caddyserver.com/docs/caddyfile/matchers#standard-matchers

Para hacer otros matcheos, o  más complejos, con múltiples condiciones, se 
utilizan matcheos con nombre (_named matchers_).

El nombre hay que definirlo en el mismo bloque que se utilizará:
```
servicio.example.net
{
	@post {
		method POST
	}
	reverse_proxy @post localhost:8000
}
```

Por ejemplo, se pueden hacer diferentes acciones según el origen de las
conexiones:
```
servicio.example.net
{
	@redes_internas {
		remote_ip 172.16.0.0/12 192.168.0.0/16 10.0.0.0/8
	}
	@redes_externas {
		not remote_ip 172.16.0.0/12 192.168.0.0/16 10.0.0.0/8
	}

	reverse_proxy @redes_internas localhost:8000
	redir @redes_externas https://www.google.com
}
```


# Aplicar cambios

Para activar cambios en la configuración usar:
```
sudo systemctl reload caddy.service
```

Se recomienda _fuertemente_ no usar `restart` ya que desde que se apaga hasta
que se reenciende no funciona nada.

En cambio, utilizando `reload` se inicia un servidor nuevo, se mantienen dos
procesos en paralelo hasta que termina de configurarse el nuevo y recién cuando
está todo listo se mata el servidor viejo y el nuevo toma los ports.

# Ubicación de los certificados

Los certificados que Caddy genera localmente u obtiene a través del protocolo 
ACME, se encuentran dentro del `${HOME}` del mismo Caddy (usualmente en
`/var/lib/caddy` bajo el path `local/share/caddy/certificates`).

Dentro de ese directorio hay un directorio `local` para los certificados que se
generan dentro de la CA local (no válidos públicamente) y uno para cada
proveedor de certificados a través del protocolo ACME (como letsencrypt.org o
zerssl.com).

Dentro de cada uno de estos directorios hay un subdirectorio por cada nombre de
dominio/certificado donde está el certificado en formato PEM (ascii) y la clave
privada también en formato PEM, además de un archivito JSON con información del
emisor del certificado.

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
