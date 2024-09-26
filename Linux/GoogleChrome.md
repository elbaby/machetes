## Instalar Google Chrome sin darle la llave de tu computadora a Google

Cuando se instala [Google Chrome](https://google.com/chrome) el instalador
configura los [repositorios oficiales](https://dl.google.com/linux/chrome/deb/)
pero agrega la [clave pública de firma de software de
Google](https://dl.google.com/linux/linux_signing_key.pub) en el directorio
`/etc/apt/trusted.gpg.d` lo que hace que automáticamente nuestro equipo confíe
en cualquier cosa que firme Google, lo cual no es una buena política.

Para peor, periódicamente y en cada actualización, hay un proceso que _insiste_
en poner la clave pública allí aunque la hayamos puesto en el lugar correcto
(el directorio `/etc/apt/keyrings`) y configurado el repositorio en APT para que
sólo confíe en esa firma para el repositorio de Google Chrome.

Por suerte [James Renken](https://github.com/jprenken) armó un script simple
para evitar que esto ocurra en una forma elegante.

[Yo](https://github.com/elbaby) lo adapté mínimamente para evitar un posible
mensaje de error y que muestre un montón de caracteres binarios en la consola
y lo dejé en [este
gist](https://gist.github.com/elbaby/fa18ad2fe34cfa212dd5c0303d980fe0).

Para instalar Google Chrome, simplemente hay que bajar el script y ejecutarlo:

```
curl --silent --show-error --location \
    https://gist.github.com/elbaby/fa18ad2fe34cfa212dd5c0303d980fe0/raw/99844bb8126aee074cde8a47fabebc89f0b09bc3/install-chrome.sh \
    sudo bash
```

Esto va a:
* Bajar la [clave pública de firma de software de
Google](https://dl.google.com/linux/linux_signing_key.pub) y ponerla en
`/etc/apt/keyrings`
* Agregar a la configuración el [repositorio de Google
Chrome](https://dl.google.com/linux/chrome/deb/) en
`/etc/apt/sources.list.d/google-chrome.list` indicándole que valide las firmas
con la clave pública guardada en `/etc/apt/keyrings/google.gpg`
* Crear el archivo `/etc/default/google-chrome` con configuraciones para que
_no_ pise la configuración del repositorio cuando se ejecute un `full-upgrade`
* Poner en `/etc/apt/trusted.gpg.d/google-chrome.gpg` un link simbólico a
`/dev/null` para que cada vez que los scripts de instalación de Google intenten
poner allí nuevamente la clave pública, en lugar de hacerlo, la borre.
* Ejecutar un `apt update` para actualizar la información de los
repositorios
* Instalar el paquete `google-chrome-stable`

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
