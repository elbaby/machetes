# Instalación de Linux (POP OS 20.04)

* Bajamos e instalamos con las [instrucciones oficiales](
https://support.system76.com/articles/install-pop/)
  * Usamos "_Full Disk Encryption_" con una clave distinta a la de login
* Entramos (en la UI gráfica) en **Settings** -> **About** y configuramos el
**Device Name** con el nombre que le queremos poner al equipo (hostname)
* Instalamos algunos paquetes más
```
# repositorio de drivers de System76
sudo apt-add-repository ppa:system76-dev/stable

# paquetes headless
sudo apt install build-essential subversion git vim openssh-server openssh-client

# paquetes UI grafica
sudo apt install gnome-tweaks synaptic thunderbird vim-gtk3
```
* Bajamos el entorno `/home/baby` y lo configuramos:
```
# backup de los archivos que vienen "de fábrica" (para que no falle el checkout)
mkdir .00-ENV-BACKUP
mv .bash* .profile .00-ENV-BACKUP

# hacemos checkout del entorno
svn checkout http://svn.ybab.net/baby/conf/baby/home_env/ .

# Copiamos archivos del cliente ssh 
cp ~/MOVEME_2_.ssh/* ~/.ssh

# Copiamos archivos del cliente gpg 
cp ~/MOVEME_2_.gnupg/* ~/.gnupg

# Copiamos archivos del cliente subversion 
cp ~/MOVEME_2_.subversion/* ~/.subversion
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
