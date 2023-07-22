# Instalar y configurar un servidor Postfix en Ubuntu

2023-07-17: Esto está probado con Ubuntu 22.04

## Instalar el paquete Postfix de los repositorios estándar

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

En la variable [`myhostname`](http://www.postfix.org/postconf.5.html#myhostname)
poner el nombre con el que se quiere que se identifique el server al atender una
conexión (por default es el hostname del equipo, pero podríamos querer que sea
otro nombre).

En la variable 
[`mydestination`](http://www.postfix.org/postconf.5.html#mydestination) agregar
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
/^myhost\.example\.com$/	ACCEPT
/^correo\.example\.com$/	ACCEPT
/^localhost\.example\.com$/	ACCEPT
/^localhost$/			ACCEPT
/^example\.com$/		ACCEPT
/^.*\.example\.net$/		ACCEPT
/^example\.net$/		ACCEPT
```
Entre la expresión regular y el `ACCEPT` _debe_ ir _exactamente **UN (1)**_ `<TAB>`.
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
