# _Logging_

En los Ubuntu y Debian nuevos, Caddy utiliza las facilidades de _logging_ de 
`systemd`, llamada 
[`journald`](https://www.server-world.info/en/note?os=Debian_11&p=journald).

Los logs se manejan en un formato binario y se consultan con la herramienta
[**`journalctl`**](https://manpages.debian.org/systemd/journalctl.1.en.html).

Sin ningún parámetro, `journalctl` muestra todo el log del sistema dentro de
un _pager_.

En general conviene filtrar para ver los mensajes que uno quiere.

Para ver los mensajes de `caddy` se puede filtrar por el nombre de la `unit`:
```
sudo journalctl --unit caddy.service
sudo journalctl -u caddy.service
```
o filtrar por el identificador de syslog:
```
sudo journalctl --identifier caddy
sudo journalctl -t caddy
```
La ventaja de filtrar por `unit` es que se muestran también mensajes producidos
por `systemd` respecto de esa `unit` (además de los que produce la `unit` en 
sí).

También podemos buscar un _pattern_ en particular con la opción `--grep`. Si
el _pattern_ es todo en minúsculas, la búsqueda es _case insensitive_. Si se
quiere hacer una búsqueda _case sensitive_ con un _pattern_ que sólo tiene
minúsculas hay que agregar la opción `--case-sensitive=y`.
```
sudo journalctl --unit caddy.service --grep fail
sudo journalctl --unit caddy.service -g fail
```

Se puede acotar el rango de tiempo de la búsqueda usando las opciones `--since`
y `--until`:
```
sudo journalctl --unit caddy.service --since "2022-01-11 00:00:00" --until "2022-01-13 23:59:59"
sudo journalctl --unit caddy.service -S "2022-01-11 00:00:00" -U "2022-01-13 23:59:59"
```

Para "seguir" un log a medida que avanza (como si se siguiera un log de texto
utilizando el comando `tail -f`) se puede utilizar la opción `--follow`:
```
sudo journalctl --unit caddy.service --follow
sudo journalctl --unit caddy.service -f
```
alternativamente, estando dentro del _pager_, apretar la tecla `F` mayúscula y
el _pager_ irá hasta el final del log y comenzará a "seguirlo".

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
