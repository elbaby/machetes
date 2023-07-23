A partir de la versión 2.4.4, Caddy permite [agregar módulos al binario
instalado](https://caddyserver.com/docs/command-line#caddy-add-package).

Si el paquete está instalado con el paquete `caddy` de Debian, los cambios se 
perderían con un upgrade. Para evitarlo se puede utilizar `dpkg-divert` y
`update-alternatives`

## Usar `dpkg-divert` para que APT no actualice la versión con módulos agregados
del ejecutable de Caddy

```
# Mover el binario original a /usr/bin/caddy.default
sudo dpkg-divert --divert /usr/bin/caddy.default --rename /usr/bin/caddy
```

## Hacer una copia del ejecutable de Caddy para agregarle los módulos

```
# Hacer una copia en /usr/bin/caddy.custom
sudo cp /usr/bin/caddy.default /usr/bin/caddy.custom

# Actualizar `caddy.custom` con los módulos deseados:
sudo /usr/bin/caddy.custom add-package github.com/caddyserver/transform-encoder
```

## Configurar _alternatives_ para que use uno u otro ejecutable de Caddy

```
# Configurar el binario original como alternativa con baja prioridad (10)
sudo update-alternatives --install /usr/bin/caddy caddy /usr/bin/caddy.default 10

# Configurar el binario con los módulos agregados como alternativa con alta prioridad (50)
sudo update-alternatives --install /usr/bin/caddy caddy /usr/bin/caddy.custom 50
```

Si más adelante se desea volver a usar el ejecutable original se puede
reconfigurar usando:
```
sudo update-alternatives --config caddy
```
y seleccionando la opción que corresponde a `caddy.default`.

## Upgrades del Caddy

Cuando se actualiza ahora el caddy usando APT, se actualizará el binario
en `/usr/bin/caddy.default` y no el que se modificó en `/usr/bin/caddy.custom`.

Si se actualizó el Caddy y se desea agregar los módulos y utilzar la nueva
versión hay que hacer lo siguiente:
```
# Volver a hacer la copia (del nuevo) binario en /usr/bin/caddy.custom
sudo cp /usr/bin/caddy.default /usr/bin/caddy.custom

# Actualizar el nuevo `caddy.custom` con los módulos deseados:
sudo /usr/bin/caddy.custom add-package github.com/caddyserver/transform-encoder
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
