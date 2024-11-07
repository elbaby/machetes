# Repositorio git para acceso remoto

En el equipo que funcionará como servidor, crear un repositorio "_bare_":
```
# Usuario con el que se conecta via ssh
# REMOTE_ACCESS_USER=git

# Directorio HOME de ${REMOTE_ACCESS_USER}
USER_HOME=$( getent passwd "${REMOTE_ACCESS_USER}" | cut -d: -f6 )

# Path donde se almacenan los repositorios
# REPOSITORY_BASE=${USER_HOME}/repo

# Nombre del repositorio a crear
# REPO_NAME=mi_repositorio

# Crear el repositorio
sudo -u ${REMOTE_ACCESS_USER} mkdir -pv ${REPOSITORY_BASE}
sudo -u ${REMOTE_ACCESS_USER} git init ${REPOSITORY_BASE}/${REPO_NAME}.git
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
