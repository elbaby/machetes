[[_TOC_]]

**[UniFi 
Network](https://help.ui.com/hc/en-us/sections/6582310816535-Network)** 
es la aplicación que gestiona todos los dispositivos de red de **[Ubiquiti 
UniFi](https://ui.com/)**.

Normalmente se vende en una [UniFi OS
Console](https://store.ui.com/collections/unifi-network-unifi-os-consoles) que 
se conecta a internet y es accesible desde el [UniFi 
Portal](https://account.ui.com/) o desde una [aplicación para
móviles](https://www.ui.com/download-software/).

Sin embargo, es posible [instalar UniFi Network en modo _self 
hosted_](https://help.ui.com/hc/en-us/articles/360012282453) en Windows, Linux
y MacOS.

## Documentación

La Guía del Usuario del Server (Controller) aparentemente no se publica más.
Aparentemente la última versión en PDF es la [UniFi Controller V5 User
Guide](https://dl.ui.com/guides/UniFi/UniFi_Controller_V5_UG.pdf) para la
versión 5.6.2 de lo que entonces se llamaba **UniFi Enterprise System
Controller**. Dejo una [copia local](uploads/docs/UniFi_Controller_V5_UG.pdf)
por si Ubiquiti decide borrarlo del server.

Para la versión actual (**UniFi Network Application** 7.1.68 a mediados de 2022)
hay una versión en línea [**UniFi Network - Use the UniFi Network 
Application**](https://help.ui.com/hc/en-us/articles/1500012237441-UniFi-Network-Use-the-UniFi-Network-Application).


* [Tom Lawrence](https://lawrencesystems.com/) tiene una empresita que brinda
servicios IT y tiene mucha experiencia con pfSense.
  * El foro de soporte (abierto) de Lawrence Systems tiene una [sección sobre firewalls y networking](https://forums.lawrencesystems.com/c/network-firewalls-vlan-subnets-blaming-dns/7) que brinda muy buen soporte (en inglés) sobre UniFi (entre muchas otras plataformas)
  * Lawrence también hace muchos videos y tiene armada una [**playlist** que mantiene actualizada sobre todo lo relacionado con **UniFi**] https://lawrence.video/unifi). Algunos videos que nos interesan:
    * [Self Hosted UniFi Network Application Controller Install Tutorial](https://youtu.be/lkUhWnDPutg) (es medio viejo, pero sirve)
    * [How To Setup VLANs With pfsense & UniFi 2023](https://youtu.be/WMyz7SVlrgc)
	* [Reviewing UniFi 7.4.156: OpenVPN Server, Big VLAN Port Management Changes, and Other New Features!](https://youtu.be/5eTR3m_ixa8)
	* [UniFi Controller Tutorial: Managing Multiple Sites & Migrations with Ease!](https://youtu.be/KKU6eJN4bVU)

## Instalación

Seguir las instrucciones para hacer la [instalación en modo _self 
hosted_](UniFi_Network-Instalacion.md).

También hay 
[instrucciones](https://support.hostifi.com/en/articles/3272609-unifi-ios-android-app-setup-direct-access)
para conectar las apps nativas IoS y Android a una consola _self hosted_.

## Configuración

### Configuraciones generales del sistema

Seguir las instrucciones para hacer las [configuraciones generales del
sistema](UniFi_Network-Configuracion_general.md)

## Configuración de las redes

### Configurar la red default

* **Dashboard** :arrow_right: **Settings**

![Elegir settings en dashboard](img/unifi/netapp-dashboard2settings.png)

* **Settings** :arrow_right: **Networks**

![Elegir networks en settings](img/unifi/netapp-settings2networks.png)

* Seleccionar la red **Default** (si se acaba de instalar, es la única que
existe).

![Seleccionar red Default en networks](img/unifi/netapp-networks-default.png)

* Elegir un nombre para la red (también puede quedar **_Default_**, pero en
general es más claro poner nombres representativos)

* Configurar la red (_host address_/_netmask_) con una red en la que esté
conectada directamente el servidor donde está instalado **UniFi Network**.

* La configuración de DHCP que aparece en esta pantalla sólo funciona si se
utiliza un router u otro dispositivo de UniFi que tenga un servidor DHCP (como
ser el UniFi Security Gateway). Por eso, salvo que se cuente con un router
UniFi, no hay que configurarlo en esta pantalla.

![Editar datos de la red default](img/unifi/netapp-networks-edit.png)


![Nueva red default](img/unifi/netapp-networks-defaultmod.png)

La configuración del ruteo y el servidor DHCP hay que hacerlo en forma 
**externa** al _UniFi Network_, ya que estas opciones sólo están soportadas si 
se tiene un _UniFi Security Gateway_ en la red.

Esto hay que hacerlo **antes** de conectar los _access points_

### Configurar la red WiFi

Aun cuando durante la [configuración 
inicial](UniFi_Network-Instalacion.md#user-content-setup-de-wifi) se indicó que 
_no_ combine los nombres de las redes de 2 y 5GHz en uno solo, al finalizar el 
proceso, están combinadas.

#### Cambiar red WiFi inicial para que sea sólo de 2.4GHz

* **Settings** :arrow_right: **WiFi**

![Elegir wifi en settings](img/unifi/netapp-settings2wifi.png)

* Seleccionar la red WiFi que se va a modificar (si se acaba de instalar, es la
única que existe).

![Seleccionar red wifi a modificar](img/unifi/netapp-wifi-2+5.png)

* Cambiarle el nombre para reconocerla como la red de 2.4GHz

* En **WiFi Band** _apagar_ la opción de 5GHz (y verificar que esté encendida
la opción de 2.4GHz)

* Aplicar los cambios

![Modificar red wifi de 2.4](img/unifi/netapp-wifi-edit2ghz.png)

#### Crear nueva red WiFi sólo de 5.8GHz sobre la misma red IP

* En **Settings** :arrow_right: **WiFi**

* Apretar **:heavy_plus_sign: Create New WiFi Network**

![Crear nueva red wifi](img/unifi/netapp-wifi-create-new.png)

* Ponerle el nombre para reconocerla como la red de 5.8GHz

* En **WiFi Band** _apagar_ la opción de 2.4GHz (y verificar que esté encendida
la opción de 5GHz)

* Aplicar los cambios

![Crear red wifi de 5.8](img/unifi/netapp-wifi-create5ghz.png)

### Conectar dispositivo (_access point_)

Es **importante** que el Access Point esté en la misma red _layer 2_ que el
servidor, y que esa red cuente con un servicio DHCP y ruteo a internet

* Ir a :arrow_right: **UniFi Devices**

![Elegir unify devices en dashboard](img/unifi/netapp-dashboard2devices.png)

![No devices found](img/unifi/netapp-devices-empty.png)

* Conectar el _access point_ a la misma red que el server y esperar a que
aparezca en esta pantalla.

![Managed by another console](img/unifi/netapp-devices-managedby.png)

* Si aparece con el status **_Managed by Another Console_** entonces hay que
[resetear el AP a los valores de fábrica, manteniendo apretado con un click el
botón de _reset_ durante 5 segundos o 
más](https://lazyadmin.nl/home-network/reset-unifi-ap-to-factory-defaults/):

* Si en el status aparece **_Click to Adopt_** entonces se puede "adoptar" el 
Access Point:

![Click to adopt](img/unifi/netapp-devices-click2adopt.png)

![Click to adopt](img/unifi/netapp-devices-adoptDevice.png)

* En el listado el status primero pasa a **_Adopting_**:

![Adopting](img/unifi/netapp-devices-adopting.png)

* Luego (si es necesario un _update_ de _firmware_) el status pasa a 
**_Updating_**:

![Updating](img/unifi/netapp-devices-updating.png)

* Finalmente pasa el status a **_Online_**:

![Online](img/unifi/netapp-devices-online.png)

### Configurar el _Access Point_

Al seleccionar un _Access Point_, se lo puede configurar:

* Cambiar el nombre para identificarlo

![AP settings](img/unifi/netapp-devices-APsettings1.png)

* Ponerle una dirección IP fija y configurar el _gateway_ y servidores DNS:

![AP settings](img/unifi/netapp-devices-APsettings2.png)

![AP settings](img/unifi/netapp-devices-APsettings3.png)


