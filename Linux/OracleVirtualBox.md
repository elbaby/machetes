# Instalar VirtualBox desde los repositorios de Oracle

La versión en los [repositorios de 
Oracle](https://www.virtualbox.org/wiki/Linux_Downloads#Debian-basedLinuxdistributions)
tienen más funcionalidad que la que viene en Ubuntu/PopOS!
Además, permite seleccionar la versión con más facilidad cuando hay varias
disponibles.

Bajamos las dos claves públicas que se usan para firmar paquetes de esos
repositorios, pero, en vez de agregarlas en `/etc/apt/truted.gpg.d` las ponemos
en un keyring en `/etc/apt/keyrings` y lo referenciamos específicamente en la 
descripción del repositorio:

```
# Generar archivo temporal, bajar primera clave y guardarla allí y luego bajar la segunda clave y agregarla allí mismo
FILE=`mktemp --tmpdir=/tmp oracle-vbox.XXXXXXXX` && curl --no-progress-meter https://www.virtualbox.org/download/oracle_vbox_2016.asc > ${FILE} && curl --no-progress-meter https://www.virtualbox.org/download/oracle_vbox.asc >> ${FILE}
# Cambiar el keyring temporal a formato binario y copiarlo en /etc/apt/keyrings
gpg --dearmor < ${FILE} | sudo tee /etc/apt/keyrings/oracle-vbox-archive-keyring.gpg > /dev/null
# Borrar el archivo temporal
rm ${FILE}
```

Armamos la descripción del repositorio con el nuevo [DEB822 Source
Format](https://repolib.readthedocs.io/en/latest/deb822-format.html) multilínea:
```
sudo tee /etc/apt/sources.list.d/oracle-virtualbox.sources <<EOF
X-Repolib-Name: Oracle VirtualBox
Enabled: yes
Types: deb
URIs: https://download.virtualbox.org/virtualbox/debian
Suites: jammy
Components: contrib
Architectures: amd64
Signed-by: /etc/apt/keyrings/oracle-vbox-archive-keyring.gpg
EOF
```

Actualizamos los repositorios e instalamos la versión 6.1:
```
sudo apt update
sudo apt install virtualbox-6.1
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
