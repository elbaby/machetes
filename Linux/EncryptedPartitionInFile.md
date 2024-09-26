# Crear una partición encriptada dentro de un archivo

Referencias:
* https://askubuntu.com/questions/58935/how-do-i-create-an-encrypted-filesystem-inside-a-file#answer-877597

* Instalar prerrequisitos
```
sudo apt install cryptsetup
```


* Crear el archivo para meter adentro el filesystem:
```
FILENAME=<nombre del archivo a crear>
FILESIZE=<tamaño del archivo a crear>
# Si se usa fallocate, vale usar KB, GB, TB, PB, EB, ZB, YB (1KB = 1000 bytes) 
# o también KiB, GiB, TiB, PiB, EiB, ZiB, YiB (1KiB = 1024 bytes)
# Si se usa dd, poner el valor en MiB

fallocate --verbose --length ${FILESIZE} ${FILENAME}
# fallocate no funciona en filesystems tipo FAT o exFAT
#dd if=/dev/zero of=${FILENAME} bs=1M count=${FILESIZE}
```

* Crear un container encriptado con una _passphrase_:
```
cryptsetup --verify-passphrase luksFormat ${FILENAME}
```
Esto va a mostrar algo similar a lo siguiente, pidiendo una confirmación
explícita y luego una _passphrase_ que hay que repetir dos veces:
```
WARNING!
========
This will overwrite data on /media/user/usbdrive/.cryptofs irrevocably.

Are you sure? (Type 'yes' in capital letters): YES
Enter passphrase for /media/user/usbdrive/.cryptofs:
Verify passphrase:
```

* _Abrir_ el container encriptado (va a pedir la passphrase)
```
FILENAME=<nombre del archivo a crear>
VOLNAME=<nombre que se le quiere dar al volumen en el device mapper>

sudo cryptsetup open ${FILENAME} ${VOLNAME}
```
Esto va a pedir la passphrase y si es correcta, va a abrir el volumen y agregar
el dispositivo en el _device mapper_.

Se puede verificar el estado con el comando
```
sudo cryptsetup status ${VOLNAME}
```

* Crear el filesystem en el dispositivo:
```
sudo mkfs.ext4 /dev/mapper/${VOLNAME}
```

* Crear el punto de montaje y montar el filesystem:
```
VOLNAME=<nombre del volumen en el device mapper>
MOUNTPOINT=<path donde se quiere montar el filesystem> (debe estar vacío si ya existe)

sudo mkdir --parents --verbose ${MOUNTPOINT}
sudo mount /dev/mapper/${VOLNAME} ${MOUNTPOINT}
```

* Para desmontar el filesystem y _cerrar_ el dispositivo:
```
sudo umount ${MOUNTPOINT}
sudo cryptsetup close ${VOLNAME}
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
