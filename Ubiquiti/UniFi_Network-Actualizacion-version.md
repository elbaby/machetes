# Actualización de versión de UniFi Network

## Actualización vía APT

Las actualizaciones de versión de la aplicación [UniFi
Network](UniFi_Network.md) instaladas como se indica
[acá](UniFi_Network-Instalacion.md#instalación-fácil) se pueden hacer
directamente con `APT`.

* Login en consola UniFi
  * Ir a **Settings** :arrow_right: **System** :arrow_right: **Backups**
:arrow_right: **Download Backup** :arrow_right: **"No Limit"** :arrow_right:
**Download**
  * Va a aparecer una ventanita (Preparing... Fetching/Downloading)
  * (tarda unos 30/40 segundos)
* Login en Proxmox
  * Stop VM 218
  * Snapshot VM 218
  * Start VM 218
* Login en `ep-it-018`
```
# Cambiar la versión del repositorio de Unifi
export OLDVERS=8.3
export NEWVERS=8.4
sudo sed --in-place=.save --expression="s/${OLDVERS}/${NEWVERS}/" /etc/apt/sources.list.d/100-ubnt-unifi.list

# Actualizar listado de paquetes
sudo apt update

# Detener el servicio unifi
sudo systemctl stop unifi.service

# Actualizar paquete unifi (el --assume-yes es para que no pregunte si ya tenemos un backup)
sudo apt-get install --assume-yes unifi

# Iniciar nuevamente el servicio unifi
sudo systemctl start unifi.service

# Opcionalmente, es un buen momento para hacer un upgrade de todos los paquetes
sudo apt full-upgrade

# Si se actualizó el kernel, también es recomendable reiniciar para que cargue el nuevo kernel
sudo reboot
```

* Si todo funcionó bien se puede borrar el snapshot del proxmox y el backup
completo hecho desde la gui UniFi

## Actualización con cambio de versión de MongoDB

Si se quiere actualizar la versión de **MongoDB**, es más fácil hacer un backup
(vía interfaz web) e instalar todo desde cero, ya que (al menos con las
versiones viejas de MongoDB) no había un _path_ de actualización razonable
porque las versiones 3.x y 4.x no estaban soportadas en los Debian nuevos.

Esto fue necesario al menos para pasar de MongoDB 3.x a 4.x cuando se actualizó
desde la versión 7.x a 8.0.x de la aplicación, y para pasar de la versión 4.x
a 7.0 de MongoDB cuando se pasó de la versión 8.0.x a la versión 8.1.x de la
aplicación.

Dado que la versión 7.0 _sí_ es soportada en Debian 12, es posible que a futuro
se pueda hacer un upgrade de la base de datos sin reinstalar todo.

Para hacer la instalación nueva, lo más fácil es crear una nueva máquina virtual
con el mismo nombre (internamente) que la vieja y una dirección IP interna
diferente provisoria.

Los pasos son los siguientes:
* Creación nueva VM con IP provisoria
* Hacer una [instalación fácil de UniFi
Network](UniFi_Network-Instalacion.md#instalación-fácil) en la nueva VM pero
**detenerse _antes_ del [setup inicial via
web](UniFi_Network-Instalacion.md#setup-inicial-via-web)**
* Hacer un [backup completo](UniFi_Network-Backup_completo.md) desde la interfaz
web
* Hacer el setup inicial [**restaurando** la instalación anterior del último
**backup**](UniFi_Network-Restore_inicial.md)
* Hacer que el caddy que resuelve el nombre público apunte a la IP provisoria
* Probar que todo funcione OK
* Apagar la VM vieja
* En la VM nueva, cambiar la IP provisoria por la IP fija que usaba la VM vieja
* Hacer que el caddy que resuelve el nombre público vuelva a apuntar a la IP
original (que ahora está en la VM nueva)
