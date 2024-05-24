# Exportar profiles de Thunderbird sin mensajes IMAP cacheados

Todas las herramientas, plugins y artículos que encontré para hacer un backup de
los profiles de Thunderbird terminan haciendo una copia completa de la carpeta
`~/.thunderird`.

Esto no es práctico cuando uno tiene configurada una o varias cuentas IMAP que
contienen miles o millones de mensajes.

En mi caso particular, mi profile de Thunderbird, conectado a mi cuenta de Gmail
y a un par de cuentas donde guardo muchos de mis mails históricos pre-Gmail,
ocupa más de 90Gb.

Si bien no me molesta ocupar ese espacio en el disco, no quiero backupearlo cada
vez que quiero exportar mi configuración para ponerla en otro equipo.

Con esto, me puse a ver cómo hacer para exportar mis profiles _sin_ guardar los
mails que están en los servidores (y que se irán cacheando con el tiempo una
vez que levante esa configuración en otra máquina).

Mozilla Thunderbird guarda los mensajes que va cacheando de los servidores IMAP
junto con toda la estructura de carpetas, debajo del directorio
`ImapMail/<nombre-del-servidor>`.

El problema es que es difícil identificar los archivos y carpetas utilizados
para almacenar los mensajes en sí (que preferimos no backupear) de la los que
se utilizan para guardar la metainformación de las carpetas y la estructura.

En el caso de que las carpetas de todos los servers estén guardadas en
Thunderbird utilizando el formato **`maildir`** (que utiliza un archivo para
cada mensaje), me aproveché del formato `maildir` y encontré una forma fácil
de hacerlo (suponiendo que no tengas, en ningún servidor IMAP, alguna carpeta
que se llame **tmp** o que se llame **cur** (en minúsculas).

En este caso, alcanza con hacer lo siguiente:
```
# COMPLETAR VARIABLES DE ENTORNO

# el nombre de la carpeta con los perfiles de Thunderbird
# (normalmente es .thunderbird)
DIRECTORIO_PROFILES=".thunderbird"

# el directorio desde donde cuelga la carpeta con los perfiles de Thunderbird
# (normalmente es el home linux del usuario)
DIRECTORIO_BASE=${HOME}

# el nombre que se le dará al backup (a gusto del consumidor)
BACKUP_FILE="thunderbird-backup_$(date +"%Y-%m-%d")"

# crear el backup (en el directorio donde estamos parados)
tar --create --verbose --bzip2 --file=${BACKUP_FILE}.tar.bz2 \
    --exclude=cur/* --exclude=tmp/* --exclude=global-messages-db.sqlite \
    --directory=${DIRECTORIO_BASE} ${DIRECTORIO_PROFILES}

if [ $? -eq 0 ] ; then
    echo "El backup quedó en ${PWD}/${BACKUP_FILE}.tar.bz2"
else
    echo "Algo falló"
fi
```

**OJO** Esto no va a funcionar para las carpetas en formato **`mbox`** (que,
lamentablemente, es el default que usa Thunderbird). Es decir, el backup se va
a hacer, pero incluirá las carpetas en formato `mbox` con todos los mensajes
cacheados.

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
