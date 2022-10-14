A partir de la versión 2.4.4, Caddy permite [agregar módulos al binario
instalado](https://caddyserver.com/docs/command-line#caddy-add-package).

Si el paquete está instalado con el paquete `caddy` de Debian, los cambios se 
perderían con un upgrade. Para evitarlo se puede utilizar `dpkg-divert` y
`update-alternatives`:

```
# Mover el binario original a /usr/bin/caddy.default
sudo dpkg-divert --divert /usr/bin/caddy.default --rename /usr/bin/caddy
# Hacer una copia en /usr/bin/caddy.custom
sudo cp /usr/bin/caddy.default /usr/bin/caddy.custom
```

Ahora se puede actualizar `caddy.custom` con los módulos deseados:
```
sudo /usr/bin/caddy.custom add-package github.com/caddyserver/transform-encoder
```

Configurar las alternativas:

```
# Configurar el binario original como alternativa con baja prioridad (10)
sudo update-alternatives --install /usr/bin/caddy caddy /usr/bin/caddy.default 10
# Configurar el binario compilado como alternativa con alta prioridad (50)
sudo update-alternatives --install /usr/bin/caddy caddy /usr/bin/caddy.custom 50
```

Si más adelante se desea volver al original se puede reconfigurar usando

```
sudo update-alternatives --config caddy
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
