# Instalar etckeeper para mantener la historia de modificaciones en /etc

Instalar el paquete (y git para control de versiones si no estaba instalado):
```
sudo apt install git etckeeper
```

Esto inicializa un repositorio git en `/etc`. 

En el futuro, si queremos ver todos los cambios que hubo en las configuraciones
basta con hacer:
```
sudo git log /etc
```

# Algunos ajustes


## Commit automático

etckeeper permite usar _hooks_ para evitar que se puedan modificar paquetes 
utilizando **`apt`**, lo cual nos permite ver cambios que se hayan producido
(o que hayamos hecho y no recordemos) dentro de `/etc`.

### Evitar commit automático antes de ejecutar APT

Para habilitar esto hay que evitar que etckeeper haga un commit automáticamente
_antes_ de instalar paquetes:

```
sudo sed --in-place=.orig-0 -e 's/^#AVOID_COMMIT_BEFORE_INSTALL=1/AVOID_COMMIT_BEFORE_INSTALL=1/' /etc/etckeeper/etckeeper.conf
```

A partir de ahora, si hay cambios en algún archivo de '/etc' sin commitear y se
intenta instalar un paquete, aparecera un mensaje similar a este y no se podrá
instalar hasta hacer un `commit` manualmente de los archivso cambiados:
```
** etckeeper detected uncommitted changes in /etc prior to apt run
** Aborting apt run. Manually commit and restart.
```

Cada vez que se manejan paquetes usando **apt**, al finalizar, se ejecuta un
`commit` de los cambios que se hubieran producido en la instalación, remoción o
modificación de paquetes.

### Evitar commit automático diario

Por otra parte, por _default_, etckeeper hace un `commit` diario. Para evitar
esto hay que hacer otro cambio en `etckeeper.conf`
```
sudo sed --in-place=.orig-1 -e 's/^#AVOID_DAILY_AUTOCOMMITS=1/AVOID_DAILY_AUTOCOMMITS=1/' /etc/etckeeper/etckeeper.conf
```

Además, hay que deshabilitar timer de `systemd`:
```
sudo systemctl disable etckeeper.timer
```

## Archivos que se modifican en forma automática con frecuencia

Hay algunos archivos que las aplicaciones modifican con frecuencia en forma
automática y hacen que se generen errores cada vez que se intenta instalar
paquetes cuando se configuró que _no_ se haga un `commit` automático antes
de ejecutar apt. En particular, algunas configuraciones de `cups`.

Para evitar esto hay que agregar los nombres de esos archivos en 
`/etc/.gitignore` y quitarlos del repositorio actual (sin borrarlos):

```
sudo tee -a /etc/.gitignore << EOF
# archivos que se modifican automáticamente en CUPS
cups/subscriptions.conf
cups/subscriptions.conf.O
cups/printers.conf
cups/printers.conf.O
EOF

# quitar los archivos del repositorio sin borrarlos
sudo git rm --cached /etc/cups/subscriptions.conf /etc/cups/subscriptions.conf.O /etc/cups/printers.conf /etc/cups/printers.conf.O
```

Configurar el usuario en el repositorio:
```
cd /etc
sudo git config user.email scm+${HOSTNAME}@baby.com.ar
sudo git config user.name "${USER} (${HOSTNAME} admin)"
```

Agregar y hacer commit de los cambios que hicimos
```
cd /etc
sudo git add .gitignore .etckeeper etckeeper/etckeeper.conf systemd/system/multi-user.target.wants/etckeeper.timer
sudo git commit -m "ajustes de configuración inicial etckeeper"
```

# Problema si se usa `snapd`

Si hay paquetes instalados utilizando `snap`, las actualizaciones de estos
paquetes las realiza el daemon `snapd` automáticamente sin utilizar APT, con lo
cual `etckeeper` no se entera de estos cambios.

Si al hacer un `git status /etc` aparecen cambios debajo del directorio
`/etc/systemd/system` que impiden realizar acciones con APT, se puede utilizar
el siguiente comando:
```
sudo git add /etc/systemd/system && \
    sudo git commit -m 'actualizaciones de snaps que etckeeper no maneja automáticamente' \
    /etc/systemd/system && sudo git add /etc/.etckeeper
```

Opcionalmente, se puede crear un alias en `.bashrc` como este:
```
alias gitsnap="sudo git add /etc/systemd/system && sudo git commit -m 'actualizaciones de snaps que etckeeper no maneja automáticamente' /etc/systemd/system && sudo git add /etc/.etckeeper"
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
