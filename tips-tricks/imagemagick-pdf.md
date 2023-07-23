# Soporte para archivos PDF en ImageMagik en Linux nuevos

A partir de 2018, por problemas de seguridad, el procesamiento de archivos .PDF
está bloqueado por política en muchas distros. Para habilitarlo hay que editar
el archivo `/etc/ImageMagick-6/policy.xml`, buscar la línea que dice:
```
<policy domain="coder" rights="none" pattern="PDF" />
```
y cambiar el `none` por **`read|write`**:
```
<policy domain="coder" rights="read|write" pattern="PDF" />
```

[Esta es
una](https://stackoverflow.com/questions/52998331/imagemagick-security-policy-pdf-blocking-conversion)
de muchas fuentes.

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
