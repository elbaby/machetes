# rclone

<!--ts-->
   * [Instalación](#instalación)
      * [Actualización automática](#actualización-automática)
   * [Remotos](#remotos)
   * [Uso básico](#uso-básico)
      * [Configuración inicial / creación y modificación del archivo de configuración (rclone.conf)](#configuración-inicial--creación-y-modificación-del-archivo-de-configuración-rcloneconf)
      * [Configuración de remotos por <em>provider</em>](#configuración-de-remotos-por-provider)

<!-- Added by: baby, at: Sun 15 Aug 19:00:15 -03 2021 -->

<!--te-->

[Rclone](https://rclone.org/) es una herramienta de línea de comandos que permite el acceso consistente a [decenas de proveedores de almacenamiento _cloud_ (**_backends_**)](https://rclone.org/#providers) incluyendo [Nextcloud](https://nextcloud.com/), [Google Drive](https://www.google.com/drive/), [Microsoft OneDrive](https://onedrive.live.com/), etc.

La [documentación](https://rclone.org/docs), es muy completa, aunque requiere de paciencia para seguirla.

## Instalación

Rclone es un programa Go, con lo cual es un solo binario autocontenido que se puede copiar directamente en un directorio dentro del _path_.

En https://rclone.org/install hay instrucciones de instalación utilizando distintos métodos en varios sistemas operativos, incluyendo instrucciones para instalar en docker o desde los fuentes.

En https://rclone.org/downloads están todas las versiones precompiladas.

Yo preferí instalar el package para Debian/Ubuntu directamente:

```
curl --output /tmp/rclone-current-linux-amd64.deb  https://downloads.rclone.org/rclone-current-linux-amd64.deb && sudo dpkg --install /tmp/rclone-current-linux-amd64.deb 
```

### Actualización automática

Rclone tiene un comando [`selfupdate`](https://rclone.org/commands/rclone_selfupdate/) que le permite actualizarse a sí mismo. Como lo tengo instalado desde el paquete `.deb` agregué un archivo al cron para que diariamente se actualice (si es que hay una versión nueva):

**`/etc/cron.daily/rclone-update`**
```
#!/bin/bash

# por más que le pongo --quiet, manda "rclone is up to date" a stdout

/usr/bin/rclone selfupdate --package deb --quiet > /dev/null
```
(no olvidarse de hacer `chmod +x /etc/cron.daily/rclone-update`)

## Remotos

En rclone, se llama **remoto** (_**remote**_) a una especificación de acceso a un **_backend_**.

Un remoto tiene un _nombre_ que consiste en letras (mayúsculas y minúsculas), dígitos, guión y _underscore_ (no puede empezar con guión). También acepta espacios en blanco, pero la gente de bien _jamás_ los utiliza en un nombre.

En la línea de comandos y archivos de configuración, a los nombres de los remotos se les agrega un "**:**" al final (ver la [sintaxis de los _paths_ remotos](https://rclone.org/docs/#syntax-of-remote-paths)).

Los remotos se definen en un archivo de configuración que, por default, está en **`~/.config/rclone/rclone.conf`** (se puede especificar uno distinto en las opciones del comando `rclone`).

## Uso básico

La sintáxis del comando `rclone` es:

**`rclone`** **_`subcomando`_** _`argumentos y opciones del subcomando`_

El comando **`rclone help`** da un listado de todos los [subcomandos disponibles](https://rclone.org/commands/)

### Configuración inicial / creación y modificación del archivo de configuración (`rclone.conf`)

El comando [**`rclone config`**](https://rclone.org/commands/rclone_config) permite crear y modificar un archivo de configuración (por default `~/.config/rclone/rclone.conf`), tanto en forma interactiva como automática (si se le dan suficientes argumentos). 
La forma más simple de crear y modificar el archivo de configuración es usar el comando **`rclone config`** en forma interactiva.

**ATENCIÓN:** El archivo de configuración contiene información **_muy sensible_** (claves, tokens de acceso, etc.) que están _levemente_ ofuscadas, en el sentido de que no están en _plain text_ pero **no están encriptadas**. Es decir, cualquiera que tenga acceso a ese archivo, tendrá acceso a los almacenamientos remotos definidos allí.

El comando `rclone config` permite encriptar este archivo (ver la opción `Set configuration password`), _peeeeeeerooooo_ una vez encriptado, será necesario introducir la clave de encripción **_cada vez que se utilice el comando_ `rclone`** (lo cual no es enteramente práctico).

```
$ rclone config
2021/08/13 16:56:28 NOTICE: Config file "/home/baby/.config/rclone/rclone.conf" not found - using defaults
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
```

### Configuración de remotos por _provider_

En https://rclone.org/#providers está la lista de proveedores de almacenamiento soportado. Cada uno con un link a su _home page_ y a la página que explica cómo configurarlo (con `rclone config`).

Por ahora voy a ir poniendo algunas notas sobre lo que yo configuré, pero las instrucciones hay que tomarlas de [allí](https://rclone.org/#providers).

* [Google Drive (drive)](remote-drive.md)
* [Microsoft OneDrive (onedrive)](remote-onedrive.md)
* [Crypt (crypt)](remote-crypt.md)

___
<!-- LICENSE -->
___
<a rel="licencia" href="http://creativecommons.org/licenses/by-sa/4.0/deed.es">
<img alt="Creative Commons License" style="border-width:0" 
src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />
Este documento está licenciado en los términos de una <a rel="licencia" 
href="http://creativecommons.org/licenses/by-sa/4.0/deed.es">
Licencia Atribución-CompartirIgual 4.0 Internacional de Creative Commons</a>.

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/deed.en">
<img alt="Creative Commons License" style="border-width:0" 
src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />
This document is licensed under a <a rel="license" 
href="http://creativecommons.org/licenses/by-sa/4.0/deed.en">
Creative Commons Attribution-ShareAlike 4.0 International License</a>.
<!-- END --> 
