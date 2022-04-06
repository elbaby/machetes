# OfflineIMAP

[OfflineIMAP](http://www.offlineimap.org/) es una herramienta para sincronizar
y mantener sincronizada una cuenta de correo IMAP con una cuenta local de 
correo, ya sea en formato **mbox** o **Maildir**. 
En realidad, también se puede usar para sincronizar dos cuentas IMAP remotas.
El repositorio fuente está [en github
](https://github.com/OfflineIMAP/offlineimap/blob/master/offlineimap.conf).

## (no more) imapsync
Anteriormente yo usaba [imapsync](https://imapsync.lamiral.info/), pero ahora,
si bien los fuentes siguen manteniéndose [en github
](https://github.com/imapsync/imapsync), y hay [algunas instrucciones para
instalar desde los mismos](https://tecadmin.net/use-imapsync-on-ubuntu/) (que 
yo no conseguí hacer funcionar), el paquete instalable [se vende por €60
](https://imapsync.lamiral.info/#buy_all). Lo que sí ofrece el autor es un
[servicio gratuito de sincronización via web](https://imapsync.lamiral.info/X/).

## Instalación

En Ubuntu, OfflineIMAP está en los repositorios estándar, con lo cual la 
instalación es simplemente:
```
sudo apt install offlineimap
```

La configuración se realiza con un archivo `.offlineimaprc` en el directorio
`home` del usuario.

La distribución trae un ejemplo de configuración ultrasimple que se puede copiar
y adaptar:
```
cp /usr/share/doc/offlineimap/examples/offlineimap.conf.minimal ~/.offlineimaprc
```
También hay un archivo de configuración completo y comentado en
[`/usr/share/doc/offlineimap/examples/offlineimap.conf`
](https://github.com/OfflineIMAP/offlineimap/blob/master/offlineimap.conf) que
funciona como documentación completa del formato.

Para ejecutar la sincronización, usar simplemente:
```
offlineimap
```

Y pedirá las claves para conectarse al servidor.


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
