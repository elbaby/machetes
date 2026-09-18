A partir de la versión 2.4.4, Caddy permite [agregar módulos al binario
instalado](https://caddyserver.com/docs/command-line#caddy-add-package).

Si el paquete está instalado con el paquete `caddy` de Debian o Cloudsmith, los
cambios se perderían con un upgrade. Para evitarlo se puede utilizar
`dpkg-divert` y `update-alternatives`

## Usar `dpkg-divert` para que APT no actualice la versión con módulos agregados
del ejecutable de Caddy

```
# Mover el binario original a /usr/bin/caddy.default
CADDYVERSION=`/usr/bin/caddy --version | cut -d ' ' -f 1`
sudo dpkg-divert --divert /usr/bin/caddy.default --rename /usr/bin/caddy
```

## Hacer una copia del ejecutable de Caddy para agregarle los módulos

```
# Hacer una copia en /usr/bin/caddy.custom.vXX.YY.Z
sudo cp /usr/bin/caddy.default /usr/bin/caddy.custom.${CADDYVERSION}

# Actualizar `caddy.custom.vX.YY.Z` con los módulos deseados:
sudo /usr/bin/caddy.custom.${CADDYVERSION} add-package github.com/caddyserver/transform-encoder
```

## Configurar _alternatives_ para que use uno u otro ejecutable de Caddy

```
# Configurar el binario original como alternativa con baja prioridad (10)
sudo update-alternatives --install /usr/bin/caddy caddy /usr/bin/caddy.default 10

# Configurar el binario con los módulos agregados como alternativa con alta prioridad (50)
sudo update-alternatives --install /usr/bin/caddy caddy /usr/bin/caddy.custom.${CADDYVERSION} 50
```

Si más adelante se desea volver a usar el ejecutable original se puede
reconfigurar usando:
```
sudo update-alternatives --config caddy
```
y seleccionando la opción que corresponde a `caddy.default`.

## Upgrades del Caddy

Cuando se actualiza ahora el caddy usando APT, se actualizará el binario
en `/usr/bin/caddy.default` y no el que se modificó en
`/usr/bin/caddy.custom.vX.YY.Z`.

Si se actualizó el Caddy y se desea agregar los módulos y utilzar la nueva
versión hay que hacer lo siguiente:
```
# obtener el nuevo número de versión para ponerlo en un binario separado
CADDYVERSION=`/usr/bin/caddy --version | cut -d ' ' -f 1`

# Hacer la copia (del nuevo) binario en /usr/bin/caddy.custom.vX.YY.ZZ (con la nueva versión)
sudo cp -v /usr/bin/caddy.default /usr/bin/caddy.custom.${CADDYVERSION}

# Actualizar el nuevo `caddy.custom.vX.YY.ZZ` con los módulos deseados:
sudo /usr/bin/caddy.custom.${CADDYVERSION} add-package github.com/caddyserver/transform-encoder

# Configurar el binario con los módulos agregados como alternativa con prioridad más alta que la versión anterior
PRIORITY=$((`update-alternatives --query caddy | awk '/^Best:/{best=$2} /^Alternative:/{alt=$2} /^Priority:/{if(alt==best) print $2}'`+5))
sudo update-alternatives --install /usr/bin/caddy caddy /usr/bin/caddy.custom.${CADDYVERSION} ${PRIORITY}
# La versión anterior queda como alternativa pero con menos prioridad

# Reiniciar el caddy (va a iniciar la nueva versión)
sudo systemctl restart caddy.service
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
