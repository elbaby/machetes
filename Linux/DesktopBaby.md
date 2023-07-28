# Instalación de Linux (Pop! OS 22.04)

([acá está la versión anterior para Pop! OS 
20.04](DesktopBaby_PopOS-20.04.md))

* Bajamos e instalamos con las [instrucciones oficiales](
https://support.system76.com/articles/install-pop/)
  * Usamos "_Full Disk Encryption_" con una clave distinta a la de login

En una ventana de terminal, instalamos **Gnome Tweaks** que permite configurar
algunas cosas que no están en las configuraciones generales (**_Settings_**) de
gnome shell y la interfaz **Synaptic** de apt:
```
sudo apt install gnome-tweaks synaptic
```

## _Settings_ de gnome shell

* Entramos (en la UI gráfica) en **Settings** &rarr; **About** y configuramos el
**Device Name** con el nombre que le queremos poner al equipo (hostname)

![About](img/pop-settings-about.png)
* En **Settings** &rarr; **Region and Language** terminamos de instalar el
_Language Support_:

![Language Support](img/pop-settings-language_support.png)
* En **Settings** &rarr; **Desktop** &rarr; **Desktop Options** poner **Date &
Time and Notifications Position** en **_Right_**

![Destkop/Options](img/pop-settings-desktop-options.png)
* En **Settings** &rarr; **Desktop** &rarr; **Background** elegir la imagen de
fondo

![Desktop/Background](img/pop-settings-desktop-background.png)
* En **Settings** &rarr; **Desktop** &rarr; **Appearance** elegir **_Light_**

![Desktop/Appearence](img/pop-settings-desktop-appearance.png)
* En **Settings** &rarr; **Desktop** &rarr; **Dock** configurar las siguientes
opciones:

![Desktop/Dock](img/pop-settings-desktop-dock_1.png)

![Desktop/Dock](img/pop-settings-desktop-dock_2.png)
* En **Settings** &rarr; **Desktop** &rarr; **Workspaces** en **Multi-monitor
Behavior** elegir **_Workspaces on Primary Display Only_**

![Desktop/Workspaces](img/pop-settings-desktop-workspaces.png)
* En **Settings** &rarr; **Privacy** &rarr; **Screen** poner **Blank Screen
Delay** en **_10 minutes_** y **Automatic Screen Lock Delay** en **_30
seconds_**

![Privacy/Screen](img/pop-settings-privacy-screen.png)
* En **Settings** &rarr; **Sound** en **System Volume** encender 
**_Over-Amplification_**

![Sound](img/pop-settings-sound.png)
* En **Settings** &rarr; **Power** en **Automatic Suspend** encender **On
Battery Power** y poner el **Delay** en **_20 minutes_** y en **Suspend & Power 
Button** encender **_Show Battery Percentage_**

![Power](img/pop-settings-power.png)
* En **Settings** &rarr; **Accessibility** &rarr; **Seeing** encender **_Large
Text_**

![Accessibility](img/pop-settings-accessibility.png)
* En **Settings** &rarr; **Date & Time** poner **Time Format** en **_24-hour_**

![Date & Time](img/pop-settings-date_time.png)


## _Tweaks_ de gnome shell
Abrir **Gnome Tweaks** y hacer algunos ajustes:

![Top Bar](img/tweaks-top_bar.png)

![Window Titlebars](img/tweaks-window_titlebars.png)


## Instalación paquetes básicos
```
# repositorio de drivers de System76 (esto ya debería estar)
#sudo apt-add-repository ppa:system76-dev/stable

# paquetes headless
sudo apt-get install build-essential subversion git vim p7zip-full p7zip-rar \
    net-tools nmap ucspi-tcp-ipv6 grip keychain imagemagick \
    openssh-server openssh-client openvpn

# paquetes UI grafica
sudo apt-get install vim-gtk3 network-manager-openvpn-gnome speedcrunch \
    gimp-help-en gimp-help-es gimp-data-extras libwmf0.2-7-gtk

# paquetes vía flatpak (sólo para el usuario)
flatpak install com.github.tchx84.Flatseal org.mozilla.Thunderbird \
    com.bitwarden.desktop org.kpassxc.KeePassXC org.ksnip.ksnip \
    org.telegram.desktop im.riot.Riot org.signal.Signal us.zoom.Zoom \
    com.mastermindzh.tidal-hifi com.spotify.Client \
    com.stremio.Stremio org.kde.kpat com.calibre_ebook.calibre

```

* Configurar `vim` como el editor preferido del sistema:
```
sudo update-alternatives --set editor /usr/bin/vim.basic
```

## Entorno `/home/baby`:
```
# backup de los archivos que vienen "de fábrica" (para que no falle el checkout)
mkdir -pv ~/.00-ENV-BACKUP
mv -v ~/.bash* ~/.profile ~/.pam_environment ~/.vim* ~/.caff* ~/.gitconfig \
    ~/.hgrc ~/.msmtp* ~/.00-ENV-BACKUP

# hacemos checkout del entorno
svn checkout http://svn.ybab.net/baby/conf/baby/home_env/ .

# Si queremos usar LaTeX, descomentar la próxima línea que mantiene fonts
# para tener en ~/texmf
#svn checkout http://svn.ybab.net/baby/conf/baby/texmf

# creamos el ~/.bash_USUARIO
make ~/.bash_${LOGNAME}

# Creamos el directorio ~/.ssh si no existe
mkdir -pv ~/.ssh
# Copiamos archivos del cliente ssh 
cp -v ~/MOVEME_2_.ssh/* ~/.ssh
# Esto ya debería estar así, pero por si acaso:
chmod -v 700 ~/.ssh

# Autorizamos la conexión vía ssh con mis claves públicas
cp -v /dev/null ~/.ssh/authorized_keys
for key in ed25519 ecdsa rsa ; do
  cat ~/.ssh/id_${key}.pub >> ~/.ssh/authorized_keys
done
chmod -v 644 ~/.ssh/authorized_keys

# Si el equipo es seguro, hay que agregarle los ~/.ssh/id_${key} desde 
# otro equipo

# Creamos el directorio ~/.gnupg si no existe
mkdir -pv ~/.gnupg
# Copiamos archivos del cliente gpg 
cp -v ~/MOVEME_2_.gnupg/* ~/.gnupg

# El directorio ~/.subversion se creó durante el svn checkout
# Copiamos archivos del cliente subversion 
cp -v ~/MOVEME_2_.subversion/* ~/.subversion
```
## _Bookmarks_ para gnome shell
Esto en general se configura desde _Files_ o el navegador de carpetas y archivos
que sea, pero es más simple clavarlo directamente en el archivo de configuración
correspondiente:
```
mkdir -pv ~/Documents/ZZ-temp ~/Pictures/Screenshots
cat >> ~/.config/gtk-3.0/bookmarks <<EOF
file:///home/baby/Documents/Cuentas Cuentas
file:///home/baby/Pictures/Screenshots Screenshots
file:///tmp /tmp
file:///home/baby/Documents/ZZ-temp ZZ-temp
EOF
```

## Extensiones de Gnome

La mayoría de las configuraciones que se podían hacer antes en Gnome Shell ahora
está en diversas extensiones que se instalan y configuran individualmente.

La página oficial de las extensiones es https://extensions.gnome.org/

Pop OS ya viene con algunas instaladas. Para activar y configurar las
extensiones instaladas hay que ir a https://extensions.gnome.org/local/

Para que eso funcione hay que instalar [esta extensión (del navegador)
](https://addons.mozilla.org/firefox/addon/gnome-shell-integration/) en Firefox
o [esta extensión
](https://chrome.google.com/webstore/detail/gnome-shell-integration/gphhapmejobijbbhgpjhcjognlahblep)
en Google Chrome o Chromium.

En [esta página de soporte de System76 (Pop OS!)
](https://support.system76.com/articles/customize-gnome/) hay recomendaciones de
varias extensiones.

Las que instalamos son las siguientes:

* [Lock Keys](https://extensions.gnome.org/extension/36/lock-keys/) muestra el
estado de las teclas `NumLock` y `CapsLock` en el panel
* [Sound Input & Output Device Chooser
](https://extensions.gnome.org/extension/906/sound-output-device-chooser/) 
muestra el listado de dispositivos de salida y entrada de sonido en el menú de
status debajo del control de volumen
* [Extension List](https://extensions.gnome.org/extension/3088/extension-list/)
permite gestionar estas extensiones de Gnome desde el panel
* [Tray Icons: Reloaded
](https://extensions.gnome.org/extension/2890/tray-icons-reloaded/) vuelve a 
mostrar los íconos de la bandeja en el panel

Otras que se pueden instalar:

* [Desktop Icons NG (DING)
](https://extensions.gnome.org/extension/2087/desktop-icons-ng-ding/) agrega
íconos al escritorio
* [User Themes](https://extensions.gnome.org/extension/19/user-themes/) permite
cargar _themes_ del usuario desde `~/.themes/gnome-shell`
* [Window List](https://extensions.gnome.org/extension/602/window-list/) es una
lista de ventanas abiertas en la parte inferior de la pantalla (como en el viejo
Gnome o MS Windows) - **OJO** si usamos el **Cosmic Dock** abajo, no se puede 
usar esta extensión porque se pisan


## `ksnip` para capturar pantallas (screenshots) 
La página oficial es la de [github](https://github.com/ksnip/ksnip).
En Pop OS! 22.04 se puede instalar con apt o vía `flatpak`.

Si no lo instalamos antes con apt, usar flatpak:
```
# instalar paquete flatpak 
flatpak install flathub org.ksnip.ksnip
```
Carpeta para capturas
```
# crear una carpeta (dentro de ~/Pictures) para guardar las capturas
mkdir -pv ~/Pictures/Screenshots
```
Abrir la interfaz y cambiar algunas configuraciones (tocando `Alt+F7` o a través
del menú _Options_ &rarr; _Settings_):

![Application Settings](img/ksnip-settings-application.png)

![Saver Settings](img/ksnip-settings-application-saver.png)

**Capture save location and filename:**
```
home/baby/Pictures/Screenshots/$Y$M$D-$T.png
```

![Tray Icon Settings](img/ksnip-settings-application-tray_icon.png)


En la ventana principal también conviene agregar un _delay_ para la captura:

![Screenshot Capture Delay](img/ksnip-editor-capture_delay.png)

Finalmente, configurarlo en las **Startup applications** para que arranque
automáticamente al iniciar la sesión:

![Launch Startup Applications](img/ksnip-startup_apps-launch.png)

![Add ksnip launcher in Startup Applications](img/ksnip-startup_apps-config.png)

## Algunas cosas más

* [Instalar y configurar etckeeper para trackear cambios en 
/etc](InstalarEtckeeper.md)
* [Habilitar procesamiento de archivos PDF en 
ImageMagick](../tips-tricks/imagemagick-pdf.md)

## Cliente de mail Mozilla Thunderbird

[Configurar Thunderbird](ConfigurarThunderbirdFlatpak.md)

## Foxit PDF Reader

[Instalar Foxit PDF Reader](Foxit.md)

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
