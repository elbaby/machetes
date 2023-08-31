# Segurización inicial

Referencia: [Manual: Protecting the
Router](https://help.mikrotik.com/docs/display/ROS/First+Time+Configuration#FirstTimeConfiguration-ProtectingtheRouter)

## Usuario para administración (que _no_ sea `admin`)

Si bien ya [configuramos la clave para el usuario
admin](IC_ResetearConfiguracion.md#password-del-administrador), como todo el
mundo sabe que el usuario administrador es `admin`, conviene crear un usuario
nuevo con los mismos permisos y _eliminar_ el usuario `admin`.

Para agregar un usuario con permisos de administrador usar una línea similar a
la siguiente en la consola. En lugar de `minombre` poner el nombre de usuario que
se quiere utilizar, en lugar de `mipassword` poner una _buena_ password y en
lugar de `Nombre Apellido` poner el nombre y apellido (o cualquier otro
comentario que se quiera agregar al usuario). La opción `group=full` es lo que
hace que el usuario tenga permisos de administración.
```conf
/user add name=minombre password=mipassword group=full comment="Nombre Apellido"
```
Esto también se puede hacer desde WebFig en **System** => **Users** apretando el
botón **Add New**.

### Verificar acceso con el nuevo usuario

Probar que se puede ingresar con el nuevo usuario y clave (ya sea con WebFig,
con WinBox o vía ssh) para verificar que se tiene la clave correcta.

* **ATENCIÓN** si se pierden las credenciales de autenticación del usuario con
permisos de administración, **_NO SE PODRÁ INGRESAR AL ROUTER_** y la única
forma de configurarlo va a ser **_RESETEARLO Y PERDER TODA LA CONFIGURACÓN_**.

### Acceso ssh sin password

Si se quiere ingresar al router con ssh sin utilizar password, hay que utilizar
un par de claves pública/privada. RouterOS sólo soporta claves en formato RSA.

Si ya tenemos una, la podemos utilizar. Normalmente, en linux los pares de
claves están en la carpeta `~/.ssh` del usuario. Por default, la clave privada
en formato RSA está en el archivo `~/.ssh/id_rsa` y la pública correspondiente
en `~/.ssh/id_rsa.pub`.

Si no tenemos, la podemos generar con el siguiente comando:
```bash
ssh-keygen -t rsa -b 4096
```

Una vez generada, copiamos la clave pública al router:
```bash
MIKROTIK=192.168.88.1
USUARIO=minombre

scp ~/.ssh/id_rsa.pub ${USUARIO}@${MIKROTIK}:/
```

También se puede subir el archivo `~/.ssh/id_rsa.pub` vía WebFig en **Files**.

Una vez copiada la clave pública en el router, se la agrega al usuario con el
siguiente comando CLI:
```conf
/user ssh-keys import public-key-file=id_rsa.pub user=minombre
```

También se lo puede configurar desde WebFig en **System** => **Users** =>
**SSH Keys** con el botón **Import SSH Key**.

Cuando un usuario tiene cargada una clave pública, el acceso con password vía
ssh se deshabilita. Si se quiere mantener el acceso con password vía ssh se
debe configurar así:
```conf
/ip ssh set always-allow-password-login=yes
```

### Deshabilitar el usuario `admin`

Una vez que se confirmó que el nuevo usuario funciona, deshabilitamos el usuario
`admin` con el siguiente comando:
```conf
/user disable admin
```

### Bloquear el acceso de WinBox via MAC de las interfaces que no sean seguras

Por default, el servidor mac corre en toedas las interfaces. Vamos a crear una
lista de interfaces "_seguras_", deshabilitar el acceso desde la lista `all` y
habilitar desde la lista que creamos.

La lista de interfaces podría desde tener sólamente la interfaz `ether2` hasta
tener todas _excepto_ la que se use para la WAN (normalmente `ether1`).

En el ejemplo, vamos a habilitar las interfaces `ether2`, `ether3`,  `ether4` y
`ether5`.
```conf
/interface list add name=listSafeMAC comment="lista de interfaces para atender server MAC"
/interface list member add list=listSafeMAC interface=ether2
/interface list member add list=listSafeMAC interface=ether3
/interface list member add list=listSafeMAC interface=ether4
/interface list member add list=listSafeMAC interface=ether5
```

Para hacer esto en WebFig, la lista se crea en **Interfaces** => **Interface
List** apretando el botón **Lists** y luego el botón **Add New**.

Para agregar las interfaces a la lista, apretar el botón **Close** (o ir desde
**Interfaces** => **Interface List**) y apretar el botón **Add New** para
agregar cada interfaz a la lsita.

Ahora configuramos el servidor MAC Telnet (que permite conexiones _layer 2_
desde un MikroTik a otro) para que sólo escuche en las interfaces de la lista
que creamos, y lo mismo para el servidor MAC WinBox (que permite conexiones
_layer 2_ desde WinBox):
```conf
# servidor MAC Telnet
tool mac-server set allowed-interface-list=listSafeMAC
# servidor MAC WinBox
tool mac-server mac-winbox set allowed-interface-list=listSafeMAC
```

Si se quiere deshabilitar el acceso WinBox por completo, se puede configurar
así:
```conf
tool mac-server mac-winbox set allowed-interface-list=none
```

Desde WebFig esto se configura en **Tools** => **MAC Server** con el botón **MAC
Telnet Server** y el botón **MAC WinBox Server**.

### Descubrimiento de MikroTiks vecinos

MikroTik utiliza un protocolo propio para descubrir otros dispositivos MikroTik
en la red. También es conveniente deshabilitar esto en la interfaz WAN y en
otras interfaces que no sean confiables.

Utilizamos la misma lista de interfaces "_seguras_" que creamos antes:
```conf
/ip neighbor discovery-settings set discover-interface-list=listSafeMAC
```

### Reforzar la seguridad del servidor SSH

Referencia: https://help.mikrotik.com/docs/display/ROS/SSH

Vamos a reforzar algunas opciones de seguridad del servidor SSH del router:
```conf
/ip ssh set strong-crypto=yes host-key-type=rsa host-key-size=4096
```

Y ahora hay que regenerar la clave ssh del host (el siguiente comando pide
confirmación):
```conf
/ip ssh regenerate-host-key
```

La próxima vez que se ingrese desde un equipo que ya se había conectado, dará
un error porque se cambió la clave del host:
```bash
$ ssh 192.168.88.1
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
SHA256:vU7MF9tzQmTAi4t2UEyKS8jSOG1ETbb1cqkV7XwnIhQ.
Please contact your system administrator.
Add correct host key in /home/baby/.ssh/known_hosts to get rid of this message.
Offending RSA key in /home/baby/.ssh/known_hosts:296
  remove with:
  ssh-keygen -f "/home/baby/.ssh/known_hosts" -R "192.168.88.1"
Host key for 192.168.88.1 has changed and you have requested strict checking.
Host key verification failed.
```

Para poder conectarse, hay que borrar la clave vieja de la lista de hosts
conocidos:
```bash
$ ssh-keygen -f "/home/baby/.ssh/known_hosts" -R "192.168.88.1"
# Host 192.168.88.1 found: line 296
/home/baby/.ssh/known_hosts updated.
Original contents retained as /home/baby/.ssh/known_hosts.old
```

Y ahora se podrá conectar nuevamente:
```bash
$ ssh 192.168.88.1
The authenticity of host '192.168.88.1 (192.168.88.1)' can't be established.
RSA key fingerprint is SHA256:vU7MF9tzQmTAi4t2UEyKS8jSOG1ETbb1cqkV7XwnIhQ.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.88.1' (RSA) to the list of known hosts.








  MMM      MMM       KKK                          TTTTTTTTTTT      KKK
  MMMM    MMMM       KKK                          TTTTTTTTTTT      KKK
  MMM MMMM MMM  III  KKK  KKK  RRRRRR     OOOOOO      TTT     III  KKK  KKK
  MMM  MM  MMM  III  KKKKK     RRR  RRR  OOO  OOO     TTT     III  KKKKK
  MMM      MMM  III  KKK KKK   RRRRRR    OOO  OOO     TTT     III  KKK KKK
  MMM      MMM  III  KKK  KKK  RRR  RRR   OOOOOO      TTT     III  KKK  KKK

  MikroTik RouterOS 7.11 (c) 1999-2023       https://www.mikrotik.com/

Press F1 for help

1970-01-02 07:34:45 ssh,critical SSH host key regenerated!

[baby@MikroTik] >
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
