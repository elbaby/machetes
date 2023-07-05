# Configurar una VM Linode con Ubuntu recién instalado

2023-07-03: Esto está probado con Ubuntu 22.04

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

# Habilitar los locales es_AR.UTF-8 y en_GB.UTF-8 (por default sólo viene habilitado en_US.UTF-8)
sudo sed -i.BAK -e 's/^# es_AR.UTF-8/es_AR.UTF-8/' -e 's/^# en_GB.UTF-8/en_GB.UTF-8/'  /etc/locale.gen
sudo locale-gen

# Es un buen momento para actualizar los paquetes
sudo apt update && sudo apt dist-upgrade
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
