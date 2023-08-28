# Backup de router MikroTik

## Backup de configuración

* Referencia: https://help.mikrotik.com/docs/display/ROS/Configuration+Management#ConfigurationManagement-ConfigurationExportandImport

Para hacer un backup de la configuración se usa el comando `export` (vía línea
de comandos/ssh). El export es relativo al menú en el que se está. Desde `/`
hace el backup de toda la configuración.

Vía ssh se puede hacer el backup y redirigirlo a un archivo:
```
#USUARIO=admin
#MIKROTIK=<ip o nombre de host del equipo>
#FILENAME=<nombre del archivo donde dejar el backup>

ssh ${USUARIO}@${MIKROTIK} export > ${FILENAME}.rsc
# convertirlo a formato de texto linux (sólo LF, no CRLF)
fromdos ${FILENAME}.rsc
```

### Información sensible
Si se quiere hacer backup también de información sensible (passwords, claves
privadas, etc), se debe usar la opción `show-sensitive` en RouterOS 7.x.
```
export show-sensitive
```

En RouterOS 6.x, la información sensible se muestra por defecto. Para evitarlo
hay que utilizar la opción `hide-sensitive`.
```
export hide-sensitive
```

### Incluir _defaults_
Por defecto, el comando `export` utiliza implícitamente la opción `compact` que
sólo muestra la configuración modificada de los valores _default_, y no estos
valores por _default_. Se puede utilizar la opción `verbose` para que se
muestren también las configuraciones por _default_.

### Líneas "dobladas"
El comando `export` (excepto cuando se usa la opción `terse`) "recorta" las
líneas para que no ocupen más de 80 caracteres poniendo una **`\`** en la
posición 79 o anterior y continuando en la línea siguiente con algunos espacios
al principio (lo que usualmente se llama _line folding_).

Por otra parte, la opción `terse` recién apareció en la versión 6.40 de
RouterOS, con lo cual no se puede usar en versiones anteriores.

Para hacer que no se "recorten" las líneas, se puede exportar vía `ssh` y
utilizar el comando `sed` para juntar las líneas:
```
#USUARIO=admin
#MIKROTIK=<ip o nombre de host del equipo>
#FILENAME=<nombre del archivo donde dejar el backup>

ssh ${USUARIO}@${MIKROTIK} export | sed ':x; /\\\r$/ { N; s/\\\r\n    //; tx }' > ${FILENAME}.rsc
# convertirlo a formato de texto linux (sólo LF, no CRLF)
fromdos ${FILENAME}.rsc
```

### Exportación concisa
A partir de la versión 6.40 de RouterOS el comando `export` permite utilizar
una opción `terse` que exporta comandos completos línea por línea, sin agrupar
por secciones.

Esto es conveniente para mantener en un repositorio `git` y comparar versiones
para ver cambios.
```
#USUARIO=admin
#MIKROTIK=<ip o nombre de host del equipo>
#FILENAME=<nombre del archivo donde dejar el backup>

ssh ${USUARIO}@${MIKROTIK} export terse > ${FILENAME}.rsc
# convertirlo a formato de texto linux (sólo LF, no CRLF)
fromdos ${FILENAME}.rsc
```

## Backup binario completo

* Referencia: https://help.mikrotik.com/docs/display/ROS/Backup
```
#USUARIO=admin
#MIKROTIK=<ip o nombre de host del equipo>
#FILENAME=<nombre del archivo donde dejar el backup>

# Generar el backup binario dentro del MikroTik
ssh ${USUARIO}@${MIKROTIK} /system backup save dont-encrypt=yes name=${FILENAME}.backup
# Bajar el backup binario
scp ${USUARIO}@${MIKROTIK}:/${FILENAME}.backup ${FILENAME}.backup
# Opcionalmente, borrar el backup binario dentro del MikroTik
ssh ${USUARIO}@${MIKROTIK} /file remove \"${FILENAME}.backup\"
```

## Backup de licencia

* Referencia: https://openwrt.org/toh/mikrotik/common?s[]=backup&s[]=mikrotik&s[]=license#saving_mikrotik_routerboard_license_key_without_using_winbox

El hardware que vende MikroTik viene con una licencia perpetua. Pero si se lo
llega a formatear con herramientas externas (dd/fdisk/etc), esa licencia se
pierde. Para backupearla hay que obtenerla vía línea de comandos.

El siguiente comando genera un archivo `XXXX-YYYY.key` en el área de **Files**
que se puede bajar desde la interfaz:
```
/system license output
```

Si se conecta vía ssh, se puede enviar el comando para generar el archivo y
luego bajarlo:
```
#USUARIO=admin
#MIKROTIK=<ip o nombre de host del equipo>
#FILENAME=<prefijo del nombre del archivo donde dejar la licencia>

# Obtener el id de la licencia
KEYID=`ssh ${USUARIO}@${MIKROTIK} /system license print | grep software-id: | sed -e 's/^ *software-id: *//' -e 's/\r//'`
echo "KEYID=${KEYID}"

# Generar el archivo de licencia dentro del MikroTik
ssh ${USUARIO}@${MIKROTIK} /system license output

# Bajar el archivo con la licencia
scp ${USUARIO}@${MIKROTIK}:/${KEYID}.key ${FILENAME}_${KEYID}.key
# convertirlo a formato de texto linux (sólo LF, no CRLF)
fromdos ${FILENAME}_${KEYID}.key

# Opcionalmente, borrar el archivo de licencia dentro del MikroTik
ssh ${USUARIO}@${MIKROTIK} /file remove \"${KEYID}.key\"
```

## Todos los pasos juntos

Aquí para un gran copy/paste que haga todos los backups (configuración / binario
licencia):
```
#USUARIO=admin
#MIKROTIK=<ip o nombre de host del equipo>
#FILENAME=<nombre del archivo donde dejar el backup>
#La línea siguiente descomentarla SOLAMENTE si el RouterOS es 6.40 o más nuevo
#CONCISA=" terse "
#La línea siguiente descomentarla SOLAMENTE si el RouterOS es 7 o más nuevo
#SENSITIVE=" show-sensitive "

# Generar y bajar el backup de la configuración
if [ ${CONCISA} ] ; then
  ssh ${USUARIO}@${MIKROTIK} export ${SENSITIVE} ${CONCISA} > ${FILENAME}.rsc
else
  ssh ${USUARIO}@${MIKROTIK} export ${SENSITIVE} | sed ':x; /\\\r$/ { N; s/\\\r\n    //; tx }' > ${FILENAME}.rsc
fi
# convertirlo a formato de texto linux (sólo LF, no CRLF)
fromdos ${FILENAME}.rsc

# Generar el backup binario dentro del MikroTik
ssh ${USUARIO}@${MIKROTIK} /system backup save dont-encrypt=yes name=${FILENAME}.backup
# Bajar el backup binario
scp ${USUARIO}@${MIKROTIK}:/${FILENAME}.backup ${FILENAME}.backup
# Borrar el backup binario dentro del MikroTik
ssh ${USUARIO}@${MIKROTIK} /file remove \"${FILENAME}.backup\"

# Obtener el id de la licencia
KEYID=`ssh ${USUARIO}@${MIKROTIK} /system license print | grep software-id: | sed -e 's/^ *software-id: *//' -e 's/\r//'`
echo "KEYID=${KEYID}"
# Generar el archivo de licencia dentro del MikroTik
ssh ${USUARIO}@${MIKROTIK} /system license output
# Bajar el archivo con la licencia
scp ${USUARIO}@${MIKROTIK}:/${KEYID}.key ${FILENAME}_${KEYID}.key
# convertirlo a formato de texto linux (sólo LF, no CRLF)
fromdos ${FILENAME}_${KEYID}.key
# Borrar el archivo de licencia dentro del MikroTik
ssh ${USUARIO}@${MIKROTIK} /file remove \"${KEYID}.key\"
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
