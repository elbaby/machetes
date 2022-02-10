# PowerDNS

[PowerDNS](https://www.powerdns.com/) es uno de los servidores DNS que andan
bien y son open source (GPLv2).

Es usual la confusión respecto de qué hace (y especialmente cómo se configura)
un "_servidor DNS_" ya que hay **dos cosas distintas** a las que llamamos
"_servidor DNS_".
Esto está acentuado porque algunos productos (notoriamente [ISC BIND](
https://www.isc.org/bind/) y [Microsoft DNS](
https://docs.microsoft.com/en-us/windows-server/networking/dns/dns-top)) son
servidores monolíticos que brindan ambos servicios.

Los dos servicios son:

* **Servidor de nombres autoritativo de zona** que se utiliza para publicar
información acerca de zonas por parte del titular de dichas zonas. Esta
publicación es, en principio, para que la consuma toda la internet.

* **Servidor iterativo** (mal llamado **_recursivo_**) **de resolución de
nombres** (o simplemente **_resolver_**) que se utiliza para obtener la
información de _cualquier_ zona en internet. Esta publicación es, en principio,
para que la consuman los clientes de ese servidor (normalmente, dentro de la
misma organización o de clientes de un proveedor) aunque existen también 
_resolvers_ públicos que dan servicio de resolución para cualquiera que lo 
desee utilizar.

En [NIC Argentina](https://nic.ar) hay una [explicación de cómo funcionan estos
servicios](https://nic.ar/es/novedades/noticias/como-funciona-el-dns).

La mayoría de los servidores modernos (al menos los open source que conozco),
brindan ambos servicios por separado (a través de distintos servidores).

En el caso de [PowerDNS](https://www.powerdns.com/software.html), el [PowerDNS
Authoritative Server](https://www.powerdns.com/auth.html) es un **servidor de
nombres autoritativo** y el
[PowerDNS Recursor](https://www.powerdns.com/recursor.html) es un **servidor
iterativo de resolución** (**_resolver_**). PowerDNS tiene otro producto llamado
[PowerDNS dnsdist](https://www.powerdns.com/dnsdist.html) que es un _load
balancer_ para DNS.

## Servidores autoritativos con _PowerDNS Authoritative Server_ (pdns_server)

**[pdns_server](https://doc.powerdns.com/authoritative/)** soporta múltiples 
back-ends.
Es decir, la información de las zonas que publica puede estar en bases de datos
relacionales, en archivos de zona tipo BIND o inclusive ser accedidos a traves
de un _pipe_ desde otro proceso o inclusive a través de una API desde otro tipo
de servidor.

<!-- Esto por ahora no lo estamos explicando 
Si bien pdns puede actuar como un servidor primario o secundario y transferir
zonas a través del mismo protocolo DNS usando NOTIFY, AXFR e IXFR, se recomienda
realizar la sincronización entre servidores autoritativos _fuera de banda_. Esto
es usualmente simple utilizando los mecanismos de replicación de las bases de
datos (especialmente si todos los servidores son administrados por la misma
organización). Esto es lo que la documentación de PowerDNS llama [replicación
nativa](https://doc.powerdns.com/authoritative/modes-of-operation.html#native-replication).
--> 


* **[Instalación de servidor PowerDNS 
autoritativo](pdns_server-instalacion.md)**
* **[Configuracion de primario y 
secundario](pdns_server-primario-secundario.md)**
* **[Habilitación de servidor HTTP para monitoreo y 
API](pdns_server-http-server-api.md)**
* **[Configuración de múltiples instancias de PowerDNS 
autoritativo](pdns_server-multiples-instancias.md)**
* **[Instalación de gui web PowerDNS WebUI](pdns_webui-instalacion.md)**

## Servidores iterativos (resolvers) con _PowerDNS Recursor_ (pdns_recursor)

**[pdns_recursor](https://doc.powerdns.com/recursor/)** se utiliza normalmente
para brindar servicio a una organización o a clientes de una organización.
En general **_no_** es un servicio que se brinde públicamente ya que hacerlo
implica un potencial consumo de recursos altísimo (ejemplos de servidores 
iterativos públicos son el [**1.1.1.1** de Cloudflare](https://1.1.1.1/dns/),
el [**8.8.8.8** de Google](https://developers.google.com/speed/public-dns/) o
el [**9.9.9.9** de Quad9](https://www.quad9.net/)).

Si bien probablemente pdns_recursor se pueda utilizar (probablemente en
combinación con [dnsdist](https://dnsdist.org/)) para brindar un servicio 
público, acá se explicará cómo configurar un servidor para una red de una 
organización pequeña o mediana.

* **[Instalación de servidor iterativo PowerDNS 
Recursor](pdns_recursor-instalacion.md)**

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
