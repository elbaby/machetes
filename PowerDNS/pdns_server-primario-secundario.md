# Configuración de servidores primarios y secundarios con PowerDNS

https://doc.powerdns.com/authoritative/modes-of-operation.html#secondary-operation

Desde la versión 4.5.0 del servidor PowerDNS Autoritativo se sigue la 
terminología recomendada en el [RFC 8499/BCP 219: DNS 
Terminology](https://www.rfc-editor.org/rfc/rfc8499.html) que a su vez sigue
recomendaciones del [Internet Draft `draft-knodel-terminology-08`: Terminology,
Power, and Exclusionary Language in Internet-Drafts and 
RFCs](https://tools.ietf.org/id/draft-knodel-terminology-08.html#name-master-slave-2)
reemplazando la nomenclatura ~~_master_~~ y ~~_slave_~~ por **_primary_** y
**_secondary_** (de todos modos la vieja nomenclatura sigue funcionando por
ahora).

## Modos de operación _nativo_, primario y secundario

PowerDNS Autoritativo soporta varios [modos de
operación](https://doc.powerdns.com/authoritative/modes-of-operation.html):
_nativo_, primario y secundario.

En el [modo de operación
nativo](https://doc.powerdns.com/authoritative/modes-of-operation.html#native-replication)
la replicación se realiza fuera de banda. Es decir, hay que asegurar que el
_back end_ se sincronice (por ejemplo, utilizando replicación a nivel de base
de datos).

En el modo de operación
[primario](https://doc.powerdns.com/authoritative/modes-of-operation.html#primary-operation)
y [secundario](https://doc.powerdns.com/authoritative/modes-of-operation.html#secondary-operation)
los servidores que actúan como primarios deben enviar una 
[notificación](https://doc.powerdns.com/authoritative/manpages/pdns_notify.1.html)
a los secundarios para que estos revisen si tienen la zona actualizada y, en
caso contrario, soliciten una [transferencia de zona
(AXFR)](https://doc.powerdns.com/authoritative/manpages/saxfr.1.html) desde
algún primario.

Hay que tener en cuenta que el funcionamiento de un servidor como **primario**
o **secundario** es algo que se configura **_para cada zona_**. Es decir, un
servidor puede ser primario para unas zonas y secundario para otras (y, de 
hecho, puede ser tanto primario como secundario para la _misma_ zona).

La misma zona puede tener múltiples servidores primarios y múltiples servidores
secundarios.

## Notificaciones y transferencias de zona autenticadas con TSIG

https://doc.powerdns.com/authoritative/tsig.html

Si bien se pueden configurar en los primarios un conjunto de direcciones IP
desde las cuales aceptar solicitudes de transferencia, lo recomendable hace
ya varios años es utilizar notificaciones y solicitudes de transferencias de
zona autenticadas. El método que se utiliza, especificado en el [RFC 8945
Secret Key Transaction Authentication for DNS 
(TSIG)](https://www.rfc-editor.org/rfc/rfc8945.html) es el de [compartir una
clave secreta entre el servidor primario y secundario
(clave TSIG)](https://doc.powerdns.com/authoritative/tsig.html).

La **clave TSIG** lleva un nombre y debe ser compartida para una zona 
determinada entre un servidor primario y uno secundario. Si bien se puede
utilizar la misma clave TSIG para cualquier zona entre cualquier par de 
servidores, se recomienda utilizar claves TSIG _distintas_ entre servidores
distintos, de modo tal que si se compromete un servidor sólo es necesario
cambiar las claves que involucran a _ese_ servidor.

Lo que de ninguna manera se debe hacer es compartir claves TSIG con 
servidores administrados por distintas entidades.

### Manejo de claves TSIG

Las **claves TSIG** se manejan con varios [comandos para gestión de claves TSIG
de la herramienta
`dnsutil`](https://doc.powerdns.com/authoritative/manpages/pdnsutil.1.html#tsig-related-commands).

Las claves tienen un algoritmo entre los siguientes:

* hmac-md5
* hmac-sha1
* hmac-sha224
* hmac-sha256
* hmac-sha384
* hmac-sha512

Si bien el más utilizado es hmac-md5, los hmac-shaXXX son más seguros.

Además, a las claves hay que asignarle un **nombre** arbitrario (compuesto como 
si fuese una etiqueta DNS: letras, números, guíones). Este nombre es local al
servidor (es decir, la misma clave podría tener distintos nombres en dos
servidores distintos).

#### Crear una nueva clave

Para generar una clave TSIG nueva se utiliza el comando `pdnsutil 
generate-tsig-key` poniendo a continuación el nombre que se le quiere dar y el
algoritmo de la clave:
```
sudo --user=pdns pdnsutil generate-tsig-key clave-de-prueba-0 hmac-sha256
```

Esto creará una nueva clave con algoritmo **hmac-sha256** le pondrá como nombre
local **clave-de-prueba-0** y la almacenará en el _backend_.

#### Importar una clave existente

Para cargar una clave generada externamente (por ejemplo, una que se generó
en otro servidor) se utiliza el comando `pdnsutil import-tsig-key` poniendo a
continuación el nombre que se le quiere dar, el algoritmo, y la clave
(codificada en Base64).

Por ejemplo, si nos pasan una clave de un servidor BIND en un archivo así:
```
key "cliente-secundario" {
          algorithm hmac-md5;
          secret "XM4NO7+Hlw6WlUB+qBVEBt7ybyvWf+rhew16ekYHfXM=";
};
```
es una clave con algoritmo **hmac-md5**, llamada **cliente-secundario** (aunque
ese es el nombre en el servidor de donde nos pasaron la clave, nosotros
podríamos ponerle cualquier otro nombre) y la clave en sí, codificada en Base64
es **`XM4NO7+Hlw6WlUB+qBVEBt7ybyvWf+rhew16ekYHfXM=`**.

Para cargarla en nuestro servidor con el nombre **primario-proveedor** usamos
el siguiente comando:
```
sudo --user=pdns pdnsutil import-tsig-key primario-proveedor hmac-md5 XM4NO7+Hlw6WlUB+qBVEBt7ybyvWf+rhew16ekYHfXM=
```

#### Borrar una clave del servidor

Para quitar una clave (que no se está utilizando) del servidor, se utiliza el
comando `pdnsutil delete-tsig-key` poniendo a continuación el nombre de la
clave:
```
sudo --user=pdns pdnsutil delete-tsig-key primario-proveedor
```

#### Listar todas las claves cargadas en el servidor

Para ver todas las claves cargadas en el servidor se utiliza el comando 
`pdnsutil list-tsig-keys`. Esto lista todas las claves poniendo primero el
nombre, luego el algoritmo y finalmente la clave codificada en Base64 (este 
formato es el mismo que utiliza el comando `pdnsutil import-tsig-key`).
```
$ sudo --user=pdns pdnsutil list-tsig-keys 
clave-de-prueba-0. hmac-sha256. y0HnBSTZEAPVWzvAGHP0u82T/XM2dfZrqvUw2qQa9gu3KnA4GQcsciiUc9o0xTYnZ3jNPxTK9eDoIN+G5kX8Tw==
primario-proveedor. hmac-md5. XM4NO7+Hlw6WlUB+qBVEBt7ybyvWf+rhew16ekYHfXM=
```

## Configuración del primario

https://doc.powerdns.com/authoritative/modes-of-operation.html#primary-operation

El soporte del modo primario viene desactivado por default en pdns. Para
activarlo, editar el archivo de configuración `/etc/powerdns/pdns.conf`
y activarlo:

```
#################################
# primary       Act as a primary
#
primary=yes
```

Para que tome la nueva configuración (si es que hubo que cambiarla):
```
sudo systemctl restart pdns.service
```

Para crear una zona vacía y configurarle los _name servers_ usamos:
```
sudo --user=pdns pdnsutil create-zone example.com
sudo --user=pdns pdnsutil set-kind example.com primary
sudo --user=pdns pdnsutil add-record example.com '' ns ns1.example.net
sudo --user=pdns pdnsutil add-record example.com '' ns ns2.example.net
```

Creamos una [clave 
TSIG](#notificaciones-y-transferencias-de-zona-autenticadas-con-tsig) para 
configurar entre este primario y uno o más secundarios para esta zona:
```
sudo --user=pdns pdnsutil generate-tsig-key example-com-prim-sec hmac-sha256
```
Exportar la clave TSIG para depués importarla en el secundario:
```
sudo --user=pdns pdnsutil list-tsig-keys | grep example-com-prim-sec > example-com-prim-sec_tsig.key
```
y asociarla a la zona example.com usada como primaria (es decir, que el
servidor la va a utilizar para aceptar solicitudes de transferencia `AXFR` para
esa zona):
```
sudo --user=pdns pdnsutil activate-tsig-key example.com example-com-prim-sec primary
```

## Configuración del secundario

https://doc.powerdns.com/authoritative/modes-of-operation.html#secondary-operation

El soporte del modo secundario viene desactivado por default en pdns. Para
activarlo, editar el archivo de configuración `/etc/powerdns/pdns.conf`
y activarlo:

```
#################################
# secondary     Act as a secondary
#
secondary=yes
```

Para que tome la nueva configuración (si es que hubo que cambiarla):
```
sudo systemctl restart pdns.service
```

Para crear una zona secundaria, indicándole la dirección IP del (o de los) 
servidor(es) primario(s):
```
sudo --user=pdns pdnsutil create-secondary-zone example.com 10.10.22.1 10.30.7.8
```

Creamos una [clave 
TSIG](#notificaciones-y-transferencias-de-zona-autenticadas-con-tsig) que
habíamos creado en el primario (y tenemos en un archivo que se llama
`example-com-prim-sec_tsig.key`):
```
cat example-com-prim-sec_tsig.key | xargs sudo --user=pdns pdnsutil import-tsig-key
```
y asociarla a la zona example.com usada como secundaria (es decir, que el
servidor la va a utilizar para realizar solicitudes de transferencia `AXFR` para
esa zona):
```
sudo --user=pdns pdnsutil activate-tsig-key example.com example-com-prim-sec secondary
```

Luego de uno o dos minutos, la zona debería haberse transferido desde el
servidor primario:
```
sudo --user=pdns pdnsutil show-zone example.com
sudo --user=pdns pdnsutil list-zone example.com
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
