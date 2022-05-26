# Copiar archivos de un repositorio git a otro manteniendo la historia

Esto lo encontré gugleando en
[stackoverflow](https://stackoverflow.com/questions/1365541/how-to-move-some-files-from-one-git-repo-to-another-not-a-clone-preserving-hi)

Supongamos que tenemos el archivo o directorio `path1/files1` dentro del
repositorio git `repo1` y queremos que eso (`files1`) aparezca en el repositorio
`repo2` bajo el path `path2` manteniendo toda la historia de los archivos.

Ambos repositorios deben estar "limpios" y actualizados.

Vamos a crear un _patch_ en el primer repositorio y aplicarlo en el segundo.

```
# entramos al primer repositorio
cd repo1
# generamos el patch
git log --pretty=email --patch-with-stat --reverse --full-index --binary -- path1/files1 > ../repo1-path1-files1.patch
cd ..

# entramos al segundo repositorio
cd repo2
# vamos al directorio donde queremos "copiar" las cosas
cd path2
# aplicamos el patch
git am --committer-date-is-author-date < ../repo1-path1-files1.patch
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
