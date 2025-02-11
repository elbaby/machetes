## Restauración inicial

Al entrar por primera vez a una consola recién instalada, si se cuenta con un
backup de una instalación anterior, se puede restaurar directamente sin hacer
la configuración inicial.

En la primera pantalla que aparece, en lugar de configurar el nombre de la red,
país, etc., seleccionar directamente la opción **Restore Server from a Backup**
y luego **Upload Backup File**

![Restore Server from a Backup](img/unifi/netapp-restoreServerFromBackup.png)

![Restore Server / Upload Backup
File](img/unifi/netapp-restoreServerUploadBackupFile.png)

  * Seleccionar (en la máquina local) el archivo que contiene el backup a
restaurar (son archivos con extensión **`.unf`**)
  * Una vez seleccionado, aparece un mensaje indicando que el archivo está
subiendo al servidor (no interrumpir este proceso porque podría cortarse la
subida del archivo)
  * Cuando termina de subir, aparece una ventana **Restore from backup** para 
confirmar. Hacerlo apretando el botón **Confirm**

![Restore from backup / Confirm](img/unifi/netapp-restoreFromBackupConfirm.png)

  * Luego aparecerá un mensaje mientras se está restaurando (esto puede durar
varios minutos)
  * Finalmente, se reiniciará la aplicación y aparecerá la pantalla de login
  * Ingresar con las credenciales de la instalación restaurada

![Restore in process...](img/unifi/netapp-restoreInProcess.png)

![First login after restore](img/unifi/netapp-restoreFirstLogin.png)
