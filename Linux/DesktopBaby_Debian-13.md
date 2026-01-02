# Instalación de Linux (Debian 13 - Trixie)

## Obtener la imagen
* Bajamos usando las [instrucciones oficiales]( https://www.debian.org/distrib/)
  * Se puede bajar la [versión
netinst](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd) que es la
más pequeña pero requiere conexión a internet durante la instalación
  * La más completa es la [versión
DVD](https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/) que permite
instalar la gran mayoría de los paquetes sin conexión a internet
  * También hay varias [versiones
**Live CD**](https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/)
que permiten bootear el sistema operativo desde un pendrive o DVD e instalarlo
desde el sistema ya funcionando. Las distintas versiones corresponden a
distintos entornos de escritorio (GNOME, KDE, Xfce, Mate, etc, incluso una
versión con una interfaz de texto solo: standard).
  * El ejemplo de instalación lo hacemos con el [**Live CD GNOME** versión
13.2.0](https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/debian-live-13.2.0-amd64-gnome.iso)

## Copiar la imagen a un dispositivo USB
* Para copiar la imagen bajada a un pendrive seguimos [las instrucciones
oficiales](https://www.debian.org/releases/stable/amd64/ch04s03.es.html)

![Instrucciones USB drive](img/db13-instrucciones-usbdrive.png)

* Identificar el nombre que le asignó el sistema al pendrive (en el ejemplo,
usamos `gparted` y vemos que el nombre es **`sdb`** (ojo **no es** `sdb1` que
es el nombre de la (primera) partición, el dispositivo en sí es _solamente_
`sdb`).

![Identificar dispositivo USB](img/db13-gparted-showdevice.png)

* Usando una ventana de terminal, copiar la imagen al dispositivo identificado
(el paso `sudo cp` tarda _mucho_):
```bash
DEVICE=/dev/sdb
LIVEIMAGE=debian-live-13.2.0-amd64-gnome.iso
sudo cp ${LIVEIMAGE} ${DEVICE}
sudo sync
```

![Copiar imagen a dispositivo USB](img/db13-copy-iso-usb.png)

## Bootear en el equipo a instalar con el dispositivo USB

Esto depende de la computadora, pero usualmente, en cuanto se enciende el
equipo con el pendrive conectado hay que apretar **`<ESC>`** o **`F2`** o
**`F10`** o alguna otra tecla (consultar la documentación o mirar atentamente
la pantalla _splash_ cuando arranca)>

Cuando carga desde el pendrive, se verá el menú de boot:

![Boot menu](img/db13-boot-menu.png)

Elegir **Live system (amd64)**. Allí arranca Debian desktop (tarda un rato,
porque carga desde el pendrive) y ofrece el tour de bienvenida (que se puede
saltear apretando **Skip**:

![Welcome tour](img/db13-live-welcome.png)

## Instalación de Debian

Luego, elegir del dash el ícono de **Install Debian** (si no aparece el dash,
apretar la tecla _Super_ (o _Windows_) para que aparezca:

![Install Debian](img/db13-dash-install-debian.png)

### Ubicación / zona horaria

En **Region** dejar **America** y en **Zone** elegir **Argentina/Buenos Aires**:

![Location](img/db13-location.png)

### Distribución de teclado

Si el teclado es el norteamericano estándar (sin eñe ni acentos), elegir la
distribución base **English (US)** y la distribución específica **English (US,
intl., with dead keys)** que permite utilizar la comilla simple como acento, la
doble como diéresis y el tilde (`~`) antes de la `n` para generar la `ñ` y
también permite utilizar las vocales con la tecla `Alt` derecha para generar
acentos y la `n` con `Alt` derecha para generar la `ñ`:

![Keyboard](img/db13-keyboard.png)

Si el teclado es en español puede ser de dos tipos. En los equipos y teclados
"de marca" (Microsoft, Dell, IBM, etc) en general se utiliza la distribución
de español latinoamericano: **Español (LA)** que tiene el acento agudo `´` a la
derecha de la `P`.
Si es un equipo más genérico o un teclado Genius o Logitech, lo más posible es
que utilice la distribución de español de España **Español (ES)**.

### Particionar el disco

Seleccionar **Erase disk** para borrar todo lo que hay en el disco. Marcar
el _checkbox_ **Encrypt system** y poner una _passphrase_ larga y segura que va
a pedir al arrancar el equipo para desncriptar los filesystems.

![Partitions](img/db13-partitions.png)

### Usuario principal

Cargar los datos del usuario principal (que automáticamente tendrá permiso
para hacer `sudo` y poder administrar el equipo) y el nombre del equipo.
Utilizar una [_password_ segura](https://clueless.ar/pasguord-guat-pasguord):

![User](img/db13-user.png)

### Chequeo final pre-instalación

Luego se mostrará una ventana con todos los datos configurados:

![Summary](img/db13-summary.png)

Verificarlos y presionar el botón **Install**.

### Instalación

Allí comienza la instalación que lleva un rato:

![Install](img/db13-install.png)

Si se presiona el botón **Toggle log** en la ventana se puede ver en tiempo real
el log de la instalación:

![Install log](img/db13-install-log.png)

Al finalizar, dejar seleccionado el _checkbox_ **Restart now** y presionar el
botón **Done**:

![Install finished](img/db13-install-finished.png)

## Primer inicio

Al reiniciar, lo primero que muestra es una pantalla de texto simple y queda a
la espera de que se tipee la _passphrase_ con la que se encriptaron los 
filesystems. Al tipearla no hay echo en la pantalla (presionar `<Enter>` al
finalizar):

![Boot encryption passphrase](img/db13-boot-passphrase.png)

![Decrypting passphrase](img/db13-decrypt-passphrase.png)

Luego aparece el menú de GRUB. Esperar unos segundos o apretar `<Enter>`
estando seleccionado **Debian GNU/Linux**:

![Boot menu](img/db13-grub-menu.png)

![Cryptsetup successfully decrypted file system](img/db13-cryptsetup.png)

Finalmente, debería aparecer la pantalla de login del _desktop manager_ (GDM)
con el primer usuario seleccionado:

![GDM login](img/db13-gdm-login.png)

![GDM password](img/db13-gdm-password.png)

Luego de seleccionar el usuario e ingresar la _password_, si es la primera vez
que ingresa este usuario, mostrará el tour de bienvenida:

![Welcome tour](img/db13-welcome.png)

## Configuración básica GNOME

Ingresar a **Settings**:

![GNOME settings](img/db13-gnome-settings.png)

### Nombre del equipo

Ir a **System** (al final) &rarr; **About**:

![system settings](img/db13-settings-system-about.png)

Acá se puede cambiar el nombre del dispositivo (_hostname_, _device name_):

![hostname](img/db13-settings-system-hostname.png)

### Fecha y hora

En **System** &rarr; **Date & Time**:

![system settings](img/db13-settings-system-datetime.png)

* Encender **Automatic Time Zone**
* Poner en **Time Format**: **24-hour** (para que funcione hay que habilitar la
[_Geolocalización_](#geolocalización) más abajo)
* En **Clock & Calendar**:
  * Encender **Week Day**
  * Encender **Seconds**

![date and time settings](img/db13-settings-datetime.png)

### Secure Shell

En **System** &rarr; **Secure Shell**:

![system settings](img/db13-settings-system-ssh.png)

Encender **Secure Shell** para acceso remoto:

![enable ssh](img/db13-system-ssh-enable.png)

Esto pide autenticación para habilitar el acceso remoto via ssh (es la clave del
usuario que está ejecutando):

![enable ssh - authenticate](img/db13-system-ssh-enable.png)

Y queda habilitado el acceso remoto con ssh:

![enable ssh](img/db13-system-ssh-enabled.png)

### Bloqueo de pantalla automático

En **Privacy & Security** &rarr; **Screen Lock**:

![privacy and security settings](img/db13-settings-privacy-screenlock.png)

* En **Blank Screen Delay** poner **12 minutes**
* En **Automatick Screen Lock Delay** poner **30 seconds**

![screen lock delays](img/db13-settings-screenlock.png)

### Geolocalización

En **Privacy & Security** &rarr; **Location**:

![privacy and security settings](img/db13-settings-privacy-location.png)

Encender **Automatic Device Location** (esto es necesario para que funcione
[_Automatic Time Zone_ más arriba](#fecha-y-hora):

![location](img/db13-settings-location.png)

## GNOME Tweaks

Ingresar a **Tweaks**:

![Tweaks](img/db13-gnome-tweaks.png)

### Ventanas

Ir a **Windows** y seleccionar **Minimize** para habilitar el botón para
minimizar en las ventanas

![windows settings](img/db13-tweaks-windows.png)



<!--


## _Settings_ de gnome shell

* Entramos (en la UI gráfica) en **Settings** &rarr; **About** y configuramos el
**Device Name** con el nombre que le queremos poner al equipo (hostname)

![About](img/pop-settings-about.png)


* En **Settings** &rarr; **Region and Language** apretar el botón **_Manage
Installed Languages_** y terminar de instalar el _Language Support_:

![Language Support](img/pop-settings-language_support.png)

* En la misma ventana, seleccionar **Regional Formats**, elegir el formato para
usar para números, fechas y monedas y apretar el botón **_Apply system-wide_**.

![Regional Formats](img/pop-settings-regional_formats.png)

* En **Settings** &rarr; **Desktop** &rarr; **Background** elegir la imagen de
fondo

![Desktop/Background](img/pop-settings-desktop-background.png)

* En **Settings** &rarr; **Desktop** &rarr; **Dock** configurar las siguientes
opciones:
  * En **Dock Options** en la opción **Icon Click Action** seleccionar
"**_Launch, Minimize, or Preview Windows_**"
  * En **Dock Visibility** 
    * seleccionar **_Intelligently hide_**
    * en la opción **Show Dock on Display** seleccionar "**_All Displays_**"
  * En **Dock Size** seleccionar **_Small (36px)_**

![Desktop/Dock](img/pop-settings-desktop-dock.png)

* En **Settings** &rarr; **Desktop** &rarr; **Workspaces** en **Multi-monitor
Behavior** elegir **_Workspaces on Primary Display Only_**

![Desktop/Workspaces](img/pop-settings-desktop-workspaces.png)

* En **Settings** &rarr; **Privacy** &rarr; **Screen** poner **Blank Screen
Delay** en **_12 minutes_** y **Automatic Screen Lock Delay** en **_30
seconds_**

![Privacy/Screen](img/pop-settings-privacy-screen.png)

* En **Settings** &rarr; **Sound** en **System Volume** encender 
**_Over-Amplification_**

![Sound](img/pop-settings-sound.png)

* En **Settings** &rarr; **Power** &rarr; **Power Saving Options** en
**Automatic Suspend** apagar **_Plugged In_** y en **Suspend & Power Button**
encender **_Show Battery Percentage_**

![Power](img/pop-settings-power.png)

* En **Settings** &rarr; **Accessibility** &rarr; **Seeing** encender **_Large
Text_**

![Accessibility](img/pop-settings-accessibility.png)

* En **Settings** &rarr; **Date & Time** poner **Time Format** en **_24-hour_**

![Date & Time](img/pop-settings-date_time.png)


## _Tweaks_ de gnome shell

En una ventana de terminal, instalamos **Gnome Tweaks** que permite configurar
algunas cosas que no están en las configuraciones generales (**_Settings_**) de
gnome shell:

```
sudo apt install gnome-tweaks
```

Abrir **Gnome Tweaks** para hacer ajustes que no están accesibles desde la
configuración estándar.

* En **Gnome Tweaks** &rarr; **Top Bar** &rarr; **Clock** encender **_Weekday_**
y **_Seconds_**

![Top Bar](img/tweaks-top_bar.png)

## Instalación paquetes básicos

```
# repositorio de drivers de System76 (esto ya debería estar)
#sudo apt-add-repository ppa:system76-dev/stable

# repositorio GitHub CLI
curl --fail --silent --show-error --location \
    https://cli.github.com/packages/githubcli-archive-keyring.gpg | \
    sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg >/dev/null
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" |\
    sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null

# actualización de repositorios y de todo lo que ya está instalado
sudo apt update
sudo apt full-upgrade

# paquetes headless
sudo apt install build-essential vim keychain tofrodos plocate \
    net-tools tcptraceroute openssh-server openssh-client openvpn nmap whois \
    ucspi-tcp-ipv6 bind9-dnsutils ipcalc tidy libxml2-utils \
    p7zip-full p7zip-rar git git-filter-repo gh grip subversion \
    neofetch direnv imagemagick

# paquetes UI grafica
sudo apt install gparted exfatprogs exfat-fuse synaptic libwmf-0.2-7-gtk \
    network-manager-openvpn-gnome speedcrunch vim-gtk3 pdfarranger \
    gimp-help-en gimp-help-es gimp-data-extras gnome-sound-recorder \
    pavucontrol

# paquetes vía flatpak (sólo para el usuario)
flatpak remote-add --user --if-not-exists flathub \
    https://flathub.org/repo/flathub.flatpakrepo
flatpak install --user --assumeyes flathub com.github.tchx84.Flatseal \
    com.bitwarden.desktop org.keepassxc.KeePassXC org.ksnip.ksnip \
    org.telegram.desktop im.riot.Riot org.signal.Signal us.zoom.Zoom \
    com.mastermindzh.tidal-hifi com.spotify.Client org.pipewire.Helvum \
    org.audacityteam.Audacity com.stremio.Stremio com.calibre_ebook.calibre \
    org.mozilla.Thunderbird com.jgraph.drawio.desktop org.kde.kpat \
    org.remmina.Remmina com.mikrotik.WinBox


# configuración de flatpak
flatpak --user config --set languages 'en;es'
# actualizacion de Locales
flatpak update --subpath=en,es org.gnome.Platform.Locale
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
make .bash_${LOGNAME}

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
# Cambiamos los permisos de los directorios y archivos en ~/.gnupg
find ~/.gnupg -type d -exec chmod 700 {} \;
find ~/.gnupg -type f -exec chmod 600 {} \;

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
file:///home/baby/Documents/Cuentas.git Cuentas
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

## Aplicaciones en el _Dock_

Estas son las aplicaciones que dejo configuradas en el _Dock_:

![Dock icons](img/pop-dock-icons.png)

## Configuración de Gnome Terminal

Abrir Gnome Terminal, seleccionar el "botón de hamburguesa" (**≡**) y
seleccionar _Preferences_:

![Terminal Preferences](img/gnome-terminal-preferences.png)

Seleccionar el perfil default llamado **_Pop_** e ir a **Colors**:

![Profile Colors](img/gnome-terminal-profile-colors.png)

Apagar la opción **_Use transparency from system theme_**, encender la opción
**_Use transparent background_** y ajustar el _slider_ con la transparencia
deseada (usualmente alrededor de un 15% es razonable):

![Terminal transparent
background](img/gnome-terminal-transparent-background.png)

## Configuración de KeePassXC

Abrir KeePassXC y cambiar algunas configuraciones (tocando `Alt+,` o a través
del menú _Tools_ &rarr; _Settings_):

![Basic Settings](img/keepassxc-application-basic-settings.png)

## Algunas cosas más

* [Instalar y configurar **ksnip** para capturar y editar screenshots](Ksnip.md)
* [Instalar y configurar **etckeeper** para trackear cambios en /etc](Etckeeper.md)
* [Instalar el cliente **NordVPN**](NordVPN.md)
* [Instalar el navegador **Google Chrome**](GoogleChrome.md)
* [Habilitar procesamiento de archivos **PDF** en 
**ImageMagick**](../tips-tricks/imagemagick-pdf.md)
* [Instalar **Foxit PDF Reader**](Foxit.md)
* [Instalar la extensión **Video DownloadHelper** en los
navegadores y la _CoApp_](VideoDownloadHelper.md)

## Cliente de mail Mozilla Thunderbird

[Configurar **Thunderbird**](ThunderbirdFlatpak.md)

También se puede [recuperar la configuración de un backup de los
perfiles](ThunderbirdBackupProfile.md)

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
