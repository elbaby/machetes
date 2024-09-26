# asdf

[asdf](https://asdf-vm.com/) (la herramienta con el nombre más fácil de tipear
del mundo) es un [manejador de versiones de herramientas en tiempo de
ejecución](https://asdf-vm.com/guide/introduction.html).

## Instalación

Seguimos las [instrucciones de
instalación](https://asdf-vm.com/guide/getting-started.html):

```
# Elegir la versión a instalar (ver https://github.com/asdf-vm/asdf/tags)
asdfVersion=v0.14.0

# Prerrequisitos
sudo apt install curl git

# Instalar asdf (es simplemente clonar el branch desde github)
git clone https://github.com/asdf-vm/asdf.git ~/.asdf --branch ${asdfVersion}

# Configurar la ejecución del script cada vez que ingresamos al shell 
# y el "completion" de los comandos para bash
cat >> ${HOME}/.bashrc <<EOF
#asdf
if [ -f ${HOME}/.asdf/asdf.sh ] ; then
	. "${HOME}/.asdf/asdf.sh"
	if [ -f ${HOME}/.asdf/completions/asdf.bash ] ; then
		. "${HOME}/.asdf/completions/asdf.bash"
	fi
fi
EOF

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
