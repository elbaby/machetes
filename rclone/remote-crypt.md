# Crypt ([crypt](https://rclone.org/crypt/)) (pseudo-remoto para encriptar _otros_ remotos)

**[Crypt](https://rclone.org/crypt/)** no es un _backend_ remoto, sino que se 
usa como una **capa** _encima_ de _otro_ "remoto".

Cuando accedés a un remoto a través del remoto _crypt_ arriba del mismo, rclone 
hace encripción/desencripción del lado del cliente, de modo tal que lo que se 
transmite y se almacena en el remoto que está debajo del _crypt_ está encriptado 
y no se puede ver.

La encripción es _simétrica_ (misma clave para encriptar y desencriptar). La 
password la podés elegir vos o dejar que la genere rclone. De cualquiera de 
ambos modos, la clave se va a guardar en el archivo de configuración 
**`~/.config/rclone/rclone.conf`** en una forma ofuscada. **Ojo**, esta 
ofuscación sólo sirve para que una vista casual del archivo no revele una clave
fácilmente legible, pero **_no está encriptada ni hasheada_**.

Es **MUY IMPORTANTE** que **_nadie_** tenga acceso a ese archivo, ya que, 
en particular, la información que está allí es suficiente para desencriptar el
contenido guardado en el remoto debajo de la capa _crypt_. (de hecho, podríamos 
copiar ese pedazo a otra computadora y utilizar rclone en la otra computadora
para tener acceso al remoto encriptado).

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
