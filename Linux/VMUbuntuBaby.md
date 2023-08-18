# Configurar una VM con Ubuntu recién instalado

2023-07-03: Esto está probado con una VM Ubuntu 22.04 en Linode.

## Primer login como `root`

```
# Todavía estoy _afuera_ del linode
IP_DEL_LINODE=<poner acá la IP que me da Linode>

# Entro como  (ya puse mi clave publica ssh para entrar ahí cuando creé la VM)
ssh root@${IP_DEL_LINODE}

# VERIFICAR QUE ESTOY ADENTRO DE LA NUEVA MAQUINA

# Creo usuario baby (primero que todo, para que quede con UID=1000)
useradd -c "Mariano Absatz" -s /bin/bash -m baby

# Copio el ~/.ssh de root (que ya me deja entrar) al usuario baby (y cambio el 
# ownership)
cp -rv ~root/.ssh ~baby
chown -Rv baby.baby ~baby/.ssh

# Lo agrego al grupo sudo
adduser baby sudo

# Le seteo la password (para que la pueda usar con sudo)
passwd baby


```
Ahora ya puedo salir (apretando ^D) y loguearme como baby

## Login como `baby`

```
# Todavía estoy _afuera_ del linode
IP_DEL_LINODE=<poner acá la IP que me da Linode>

# Entro como baby (copié la clave pública desde el usuario root más arriba)
ssh baby@${IP_DEL_LINODE}

# VERIFICAR QUE ESTOY ADENTRO DE LA NUEVA MAQUINA

# Setear el hostname (se va a ver después de desloguearse/reloguearse)
HOST_NAME=<poner acá el FQDN del host>
sudo hostnamectl set-hostname $HOST_NAME

# Habilitar los locales es_AR.UTF-8 y en_GB.UTF-8
# (por default sólo viene habilitado en_US.UTF-8)
sudo sed -i.BAK -e 's/^# es_AR.UTF-8/es_AR.UTF-8/' \
    -e 's/^# en_GB.UTF-8/en_GB.UTF-8/'  /etc/locale.gen
sudo locale-gen

# Configurar el timezone
sudo timedatectl set-timezone America/Argentina/Buenos_Aires

# Es un buen momento para actualizar los paquetes
sudo apt-get update
sudo apt-get dist-upgrade

# Finalmente, rebootear para que los logs empiecen a generarse con la hora local
sudo reboot
```

## Instalación paquetes básicos
```
sudo apt-get install build-essential subversion git git-filter-repo vim grip \
    p7zip-full p7zip-rar net-tools nmap ucspi-tcp-ipv6 keychain imagemagick \
    openssh-server openssh-client openvpn
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

# Creamos el directorio ~/.gnupg si no existe
mkdir -pv ~/.gnupg
# Copiamos archivos del cliente gpg 
cp -v ~/MOVEME_2_.gnupg/* ~/.gnupg

# El directorio ~/.subversion se creó durante el svn checkout
# Copiamos archivos del cliente subversion 
cp -v ~/MOVEME_2_.subversion/* ~/.subversion
```

## Algunas cosas más

* [Instalar y configurar etckeeper para trackear cambios en 
/etc](Etckeeper.md)
* [Instalar y configurar el firewall Shorewall](Shorewall.md)

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
