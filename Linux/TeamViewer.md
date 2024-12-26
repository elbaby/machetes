# Instalación de TeamViewer en linux

La primera vez hay que bajar el `.deb` e instalarlo:
```
curl --location --remote-name --output-dir /tmp \
    https://download.teamviewer.com/download/linux/teamviewer_amd64.deb

sudo apt install /tmp/teamviewer_amd64.deb
```
Esto instaló el paquete y además agregó la configuración del repositorio en
`/etc/apt/sources.list.d/teamviewer.list` y la clave pública con la que está
firmado en `/usr/share/keyrings/teamviewer-keyring.gpg`.

Para mayor consistencia, vamos a mover la clave pública a `/etc/apt/keyrings` y
ajustar la configuración del repositorio.
```
sudo mv -v /usr/share/keyrings/teamviewer-keyring.gpg /etc/apt/keyrings

sudo sed --in-place=.save --expression='s#/usr/share#/etc/apt#' \
    /etc/apt/sources.list.d/teamviewer.list

# actualizar paquetes
sudo apt update
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
