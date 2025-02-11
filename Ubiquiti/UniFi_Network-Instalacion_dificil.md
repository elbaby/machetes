[[_TOC_]]

# Instalación difícil (y desactualizada) de [UniFi Network](UniFi_Network.md)

Esta versión difícil es la que utilizó el Baby para instalar las cosas
manualmente. En su mayor parte fueron tomadas del [script de Glenn Rietveld
(a.k.a _UI Glenn_) para instalar la versión 7.1.68 bajo Debian
11](https://get.glennr.nl/unifi/install/unifi-7.1.68.sh) tomados del [posteo en
los foros de UniFi de UI 
Glenn](https://community.ui.com/questions/UniFi-Installation-Scripts-or-UniFi-Easy-Update-Script-or-UniFi-Lets-Encrypt-or-UniFi-Easy-Encrypt-/ccbc7530-dd61-40a7-82ec-22b17f027776).

Es decir, por si acaso me puse a leer el script y seguir los pasos y
documentarlos acá.

Con el diario del lunes, lo más razonable es simplemente [instalarlo con el
script de UI Glenn](UniFi_Network-Instalacion.md#instalación-fácil)


Estas son instrucciones para instalar [UniFi Network](UniFi_Network.md) en 
[modo _self hosted_](https://help.ui.com/hc/en-us/articles/220066768-UniFi-How-to-Install-and-Update-via-APT-on-Debian-or-Ubuntu)
en Debian 11 (Bullseye). Acá hay un [video](https://youtu.be/lkUhWnDPutg) que
explica cómo instalarlo en Ubuntu 20.04.

## Repositorios y claves

Obtener las claves públicas con las que están firmados los paquetes y 
repositorios y ponerlas en `/usr/share/keyrings/unifi-archive-keyring.gpg`,
`/usr/share/keyrings/mongodb36-archive-keyring.gpg` y 
`/usr/share/keyrings/adoptium-archive-keyring.gpg`:

```
curl --location https://dl.ui.com/unifi/unifi-repo.gpg | gpg --dearmor | sudo tee /usr/share/keyrings/unifi-archive-keyring.gpg > /dev/null
curl --location https://www.mongodb.org/static/pgp/server-3.6.pub | sudo tee /usr/share/keyrings/mongodb36-archive-keyring.gpg > /dev/null
#curl --location https://www.mongodb.org/static/pgp/server-4.4.pub | sudo tee /usr/share/keyrings/mongodb44-archive-keyring.gpg > /dev/null
#curl --location https://download.bell-sw.com/pki/GPG-KEY-bellsoft | gpg --dearmor | sudo tee /usr/share/keyrings/bellsoft-archive-keyring.gpg > /dev/null
curl --location https://packages.adoptium.net/artifactory/api/gpg/key/public | sudo gpg --dearmor -o /usr/share/keyrings/adoptium-archive-keyring.gpg > /dev/null
```

Poner el repositorio de UniFi en un archivo 
`/etc/apt/sources.list.d/unifi.list`, el de mongodb en 
`/etc/apt/sources.list.d/mongodb.list` y el de Bellsoft (Oracle clone) Java 8 en 
`/etc/apt/sources.list.d/bellsoft-java.list:

```
sudo tee /etc/apt/sources.list.d/unifi.list <<EOF
# https://help.ui.com/hc/en-us/articles/220066768-UniFi-Network-Updating-Third-Party-non-Console-UniFi-Network-Applications-Linux-Advanced-#1
deb [arch=amd64 signed-by=/usr/share/keyrings/unifi-archive-keyring.gpg] https://www.ui.com/downloads/unifi/debian stable ubiquiti
EOF

sudo tee /etc/apt/sources.list.d/mongodb.list <<EOF
# Los repos de MongoDB 3.x sólo están para Stretch (ni Buster ni Bullseye). Unifi no soporta MongoDB 4 en adelante
deb [arch=amd64 signed-by=/usr/share/keyrings/mongodb36-archive-keyring.gpg] http://repo.mongodb.org/apt/debian stretch/mongodb-org/3.6 main
EOF

#sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list <<EOF
## Los repos de MongoDB 4.x sólo están para Stretch y Buster  no Bullseye). Unifi no soporta MongoDB más nuevo que 4.4
#deb [arch=amd64 signed-by=/usr/share/keyrings/mongodb44-archive-keyring.gpg] http://repo.mongodb.org/apt/debian buster/mongodb-org/4.4 main
#EOF

sudo tee /etc/apt/sources.list.d/adoptium.list <<EOF
# Repositorio adoptium / temurin JDP
# https://adoptium.net/installation/linux/
deb [signed-by=/usr/share/keyrings/adoptium-archive-keyring.gpg] https://packages.adoptium.net/artifactory/deb $(lsb_release -cs) main
EOF

#sudo tee /etc/apt/sources.list.d/bellsoft-java.list <<EOF
## https://bell-sw.com/pages/repositories/#apt
#deb [arch=amd64 signed-by=/usr/share/keyrings/bellsoft-archive-keyring.gpg] https://apt.bell-sw.com/ stable main
#EOF
```

Actualizar los datos de los repositorios:

```
sudo apt-get update
```

## Instalación

Instalar Temurin Java 17 y UniFi Network
```
sudo apt-get install temurin-17-jdk unifi
```

Configurar el `$JAVA_HOME` en UniFi para que encuentre el Java 17 de Temurin:
```
sudo tee /etc/default/unifi <<EOF
JAVA_HOME=/usr/lib/jvm/temurin-17-jdk-amd64
EOF
```

Arrancar la aplicación:
```
sudo systemctl start unifi.service
```

Los logs de control (arranque/parada) de la aplicación se pueden ver con
```
sudo journalctl -u 'unifi' -e
```

Los logs del servidor se mantienen en **`/var/log/unifi`**.

Los datos del servidor se almacenan en **`/var/lib/unifi`**.

Para usar la aplicación hay que ir a `https://<IP-DEL-EQUIPO>:8443` (Ubiquiti
recomienda utilizar **Chrome**).

Esto va a dar un **ERROR SSL** ya que el certificado que tiene la aplicación es
autofirmado.

Se puede instalar un [Caddy](Caddy.md) para usar de proxy reverso con 
certificados obtenidos vía LetsEncrypt.

## Setup inicial

Al entrar por primera vez hay que hacer un [**setup
inicial**](UniFi_Network-Setup_inicial.md)

## Configuración

Seguir con la [configuración](UniFi_Network.md#user-content-configuración)

## Upgrades

La instalación hecha vía APT no se actualiza desde dentro de la aplicación.

Hacer un backup dentro de la aplicación:
Settings -> System -> Backups

Los backups se mantienen en el servidor en `/var/lib/unifi/backups` (los
backups automáticos, si están configurados, se guardan en
`/var/lib/unifi/backups/autobackup`).

Para actualizarla, hay que ingresar al server y, primero, actualizar los
repositorios.
```
sudo apt-get update
```
Si hubiese cambiado el número de versión (más allá del último número, de
release), va a solicitar autorización para el _Codename_ del repositorio:
```
(...)
E: Repository 'https://dl.ui.com/unifi/debian stable InRelease' changed its 'Codename' value from 'unifi-7.3' to 'unifi-7.4'
N: This must be accepted explicitly before updates for this repository can be applied. See apt-secure(8) manpage for details.
Do you want to accept these changes and continue updating from this repository? [y/N]
```
acá hay que contestar **`y`**
```
Get:17 https://dl.ui.com/unifi/debian stable/ubiquiti amd64 Packages [712 B]
(...)
```

Ahora, actualizar el paquete `unifi`:
```
sudo apt-get install unifi
```

Esto va a consultar si hay un backup:
```
Package configuration

    ┌────────────────────────┤ Configuring unifi ├─────────────────────────┐
    │                                                                      │
    │ It is recommended that you create a backup before installing a new   │
    │ version.                                                             │
    │                                                                      │
    │ Do you have a backup?                                                │
    │                                                                      │
    │                   <Yes>                      <No>                    │
    │                                                                      │
    └──────────────────────────────────────────────────────────────────────┘

```
Contestar **`<Yes>`**.

Reiniciar la aplicación:
```
sudo systemctl restart unifi.service
```
<!--
### Upgrade de MongoDB

La primera versión que instalamos de Unifi Network sólo soportaba MongoDB 3.6.

A partir de la versión 8.0.7, también soporta MongoDB 4.4 (que es _menos vieja_).

* Detener tanto el servicio unifi como mongodb:
```
sudo systemctl stop unifi.service
```
-->
