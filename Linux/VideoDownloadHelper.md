# Video DownloadHelper

[Video DonwloadHelper](https://www.downloadhelper.net/) es una extensión para
navegadores
([Firefox](https://addons.mozilla.org/firefox/addon/video-downloadhelper),
[Chrome](https://chrome.google.com/webstore/detail/video-downloadhelper/lmjnegcaeklhafolokijcfjliaokphfk),
[Edge](https://microsoftedge.microsoft.com/addons/detail/video-downloadhelper/jmkaglaafmhbcpleggkmaliipiilhldn))
que permite bajar videos de muchos (no todos) sitios de streaming de videos.

Funciona en Linux, Windows y MacOS y en todos los casos necesita de una
aplicación instalada directamente en el sistema operativo que ellos llaman
[CoApp](https://github.com/aclap-dev/video-downloadhelper/wiki/CoApp-Installation)

La [Documentación está en esta
wiki](https://github.com/aclap-dev/video-downloadhelper/wiki).

La extensión se instala desde el mismo navegador.

La Companion Application (CoApp) se instala el Linux con [este
script](https://github.com/aclap-dev/vdhcoapp/releases/latest/download/install.sh).
La instalación es dentro del usuario que la ejecuta, con permisos para ese
usuario (no requiere permisos de `root`).

Para instalar (o actualizar) la CoApp hay que **apagar todos los navegadores
donde está instalada la extensión** y ejecutar lo siguiente:

```
curl -sSLf https://github.com/aclap-dev/vdhcoapp/releases/latest/download/install.sh | bash
```

Para desinstalar la CoApp hay que **apagar todos los navegadores donde está
instalada la extensión** y ejecutar lo siguiente:
```
~/.local/share/vdhcoapp/vdhcoapp uninstall
rm -rf ~/.local/share/vdhcoapp
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
