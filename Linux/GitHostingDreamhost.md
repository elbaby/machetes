# Migrando mis viejos repositorios subversion en Dreamhost a git

Voy a tomar lo que dice
[acá](https://help.dreamhost.com/hc/en-us/articles/216445197-Pushing-your-local-Git-repository-to-a-DreamHost-server-Linux-Mac-OS-X)
como base para migrar los repositorios subversion que tengo en Dreamhost a
repositorios git hosteados allí mismo.

## Nuevo subdominio y usuario

* Creo (en el [panel de gestión de
Dreamhost](https://panel.dreamhost.com/index.cgi?tree=domain.dashboard)) un
nuevo sudominio para hostear mis repositorios (supongamos **git.example.com**
donde el dominio example.com ya lo tengo hosteado allí).
  * El tipo de hosting que le pongo es "_Fully hosted_".
  * Agregarle un certificado gratuito LetsEncrypt para utilizar acceso https en
lugar de http
* Creo un [nuevo usuario
SFTP](https://panel.dreamhost.com/index.cgi?tree=users.dashboard) (digamos
**gitexamplecom**) y le habilito el acceso ssh (con bash).
* Vuelvo a editar el "website" que creé 
[https://panel.dreamhost.com/index.cgi?tree=domain.dashboard#!/site/**git.example.com**](https://panel.dreamhost.com/index.cgi?tree=domain.dashboard#!/site/git.example.com)
  * En la opción **Paths** aprieto **Modify** ycambio el **Web directory** para
que en lugar de ser `/home/username/` **`example.com`** sea `/home/username/`
**`www/example.com`**.
  * En la opción **Manage Files** aprieto **Login Info** y luego **Switch user**
para cambiar el usuario por el que acabo de crear (luego de eso el usuario que
se creó automáticamente al crear del subdominio se puede borrar).

## Crear un subdirectorio para poner los repositorios

* Creamos un subdirectorio del directorio expuesto en la web para poner allí los
repositorios
```
cd ~/www/git.example.com
mkdir -pv gitrepos
```

### Proteger el acceso https a ese subdirectorio con usuario y clave

Fuente: https://help.dreamhost.com/hc/en-us/articles/216363187-Password-protecting-your-site-with-an-htaccess-file

* Loguearse con el usuario sftp/ssh creado para el sitio
* Crear el archivo **`.htpasswd`** con el primer usuario y su clave en un
directorio que no está accesible via web:
```
htpasswd -c ~/www/.htpasswd usuario1
```
  (esto va a pedir la clave y confirmación en el teclado)
* Para agregar usuarios, el comando es igual pero _sin_ la opción `-c` que es la
que crea el archivo
```
htpasswd ~/www/.htpasswd usuario2
```
* Agregar un archivo con el nombre **`.htaccess`** (el nombre _debe_ ser
`.htaccess`) en el directorio web que queremos proteger (en nuestro ejemplo, en
`/home/gitexamplecom/www/git.example.com/gitrepos`), con el siguiente contenido:
```
# Proteger el directorio
Options -Indexes
AuthName "¿Quién sos?"
AuthType Basic
AuthUserFile /home/gitexamplecom/www/.htpasswd
Require valid-user
```
* Ajustar el nombre del archivo `/home/gitexamplecom/www/.htpasswd` a la
realidad.

## Crear un repositorio git vacío

* Ir al subdirectorio de repositorios y crear un subdirectorio para el nuevo
repositorio. El nombre del subdirectorio **_debe_** terminar en **.git**.
```
cd ~/www/git.example.com/gitrepos
mkdir -pv miproyecto.git
```

* Crear en el nuevo directorio un repositorio vacío
```
NOMBREREPO=nombre_del_repositorio
cd ~/www/git.example.com/gitrepos/${NOMBREREPO}.git
git init --bare
git symbolic-ref HEAD refs/heads/main
```

## Convertir el viejo repositorio subversion a git

Referencias:
* https://git-scm.com/book/en/v2/Git-and-Other-Systems-Migrating-to-Git
* https://john.albin.net/git/convert-subversion-to-git

* Loguearse vía ssh al usuario que tiene asociado el repositorio subversion.
* Obtener los nombres de usuario de los commits del repositorio subversion (en
subversion, los nombres de usuario sólo tienen un string simple, no tienen el
nombre completo ni el mail que es lo que necesita git).
```
SVNREPO=nombre_del_repositorio

mkdir -pv ${HOME}/tmp

svn log -q file://${HOME}/svn/${SVNREPO}|awk -F '|' '/^r/ {sub("^ ", "", $2); \
  sub(" $", "", $2); print $2" = "$2" <"$2">"}' \
  | sort -u > ${HOME}/tmp/authors_${SVNREPO}_svn2git.txt
```
Esto dejó en el archivo `${HOME}/tmp/authors_${SVNREPO}_svn2git.txt` una lista
de nombres con el formato siguiente
```
baby = baby <baby>
juan = juan <juan>
...
```
Ese archivo hay que convertirlo al formato requerido para la opción `-A` (o
`--authors-file`) del comando `git svn`:
```
baby = Mariano Absatz <baby@example.net>
juan = Juan Pérez <juan.perez@example.com>
...
```

* Clonar el repositorio usando `git svn`:
```
SVNREPO=nombre_del_repositorio
git svn clone file://${HOME}/svn/${SVNREPO} --no-metadata \
  --authors-file ${HOME}/tmp/authors_${SVNREPO}_svn2git.txt \
  ${HOME}/tmp/${SVNREPO}.git
```

* Pasar las propiedades `svn:ignore` a `.gitignore`
```
cd ${HOME}/tmp/${SVNREPO}.git
git svn show-ignore > .gitignore
git add .gitignore
git commit --message='Convertir propiedades svn:ignore a .gitignore'
```

## "Subir" (_push_) el repositorio al servidor

* Loguearse vía ssh al usuario donde clonamos el repositorio svn a git
* Crear un par de claves ssh (sin _passphrase_) para conectarse vía ssh desde
este usuario al usuario que tiene los repositorios git.
```
mkdir -pvm 0755 ~/.ssh
ssh-keygen -C "svn2git@${HOSTNAME}" -f ~/.ssh/svn2git-key -t rsa -b 4096 -N ""
```
* Copiar la parte pública de la clave recién creada al usuario sftp/ssh creado
para el sitio de los repositorios git (en nuestro ejemplo, **gitexamplecom**).
Supongamos que el nombre del host es **server.dreamhost.com** (si es el mismo
hosting compartido entre los dos sitios, se puede usar simplemente localhost):
```
ssh-copy-id -i ~/.ssh/svn2git-key giteaxamplecom@server.dreamhost.com
# si es en el mismo host:
#ssh-copy-id -i ~/.ssh/svn2git-key giteaxamplecom@localhost
```
* Ir al repositorio git que ya clonamos y agregarle un `remote` que apunte al
repositorio que creamos en el servidor:
```
SVNREPO=nombre_del_repositorio
USUARIOREMOTO=gitexamplecom
SERVERREMOTO=server.dreamhost.com
GITREPOREMOTO='www/git.example.com/gitrepos/miproyecto.git'
PWDIRREMOTO=www
cd ${HOME}/tmp/${SVNREPO}.git

# el nombre del branch (podría ser trunk, master, main u otro)
MAINBRANCH=`git rev-parse --abbrev-ref HEAD`
# renombrémoslo para que quede main como en el repositorio vacío creado en el server
git branch -m ${MAINBRANCH} main

# agregar el remoto
git remote add origin ${USUARIOREMOTO}@${SERVERREMOTO}:${GITREPOREMOTO}
# configurar el comando ssh que usa git para este repo, para que use la clave generada
git config --add --local core.sshCommand 'ssh -i ~/.ssh/svn2git-key'

# push al remoto por primera vez:
git push --set-upstream origin main
```
* Opcionalmente, podemos copiar los archivos de permisos y claves subversion a
un subdirectorio (no accesible desde la web) del servidor remoto:
```
scp -i ~/.ssh/svn2git-key ${HOME}/svn/${SVNREPO}.access \
  ${HOME}/svn/${SVNREPO}.passwd ${USUARIOREMOTO}@${SERVERREMOTO}:${PWDIRREMOTO}/
```

## Clonar el repositorio desde otro lado

En cualquier otro equipo ahora se puede clonar el repositorio desde el servidor.

Los pasos son:
* Copiar la clave pública ssh al servidor
* Hacer un git clone via ssh
```
# elegir la clave ssh que se quiere usar
CLAVESSH=~/.ssh/id_rsa.pub
USUARIO=gitexamplecom
SERVER=server.dreamhost.com
REPOREMOTO='www/git.example.com/gitrepos/miproyecto.git'

# Copiar la clave pública al server
ssh-copy-id -i ${CLAVESSH} ${USUARIO}@${SERVER}

# Clonar el repositorio remoto a un repositorio local
git clone ${USUARIO}@${SERVER}:${REPOREMOTO}
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
