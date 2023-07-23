# TODO MAL
No termino de entender cómo usar postgres (salvo una pequeña receta para
redhat/centos usando docker).

Además, el autor del paquete lo está abandonando:
https://github.com/ngoduykhanh/PowerDNS-Admin/issues/1032

# Instalación de PowerDNS Admin

En https://github.com/PowerDNS/pdns/wiki/WebFrontends hay varios frontends web
para administrar un servidor autoritativo PowerDNS.

El más completo y actualizado parece ser [PowerDNS
Admin](https://github.com/ngoduykhanh/PowerDNS-Admin), escrito en Python y JS y
se comunica con el server a través de la API de pdns (no escribe directamente en
la base de datos).

Las [instrucciones](https://github.com/ngoduykhanh/PowerDNS-Admin/wiki/Running-PowerDNS-Admin-on-Ubuntu-or-Debian)
están medio desordenadas y presumen que usás un motor MySQL Community Server. 
Intentaremos adecuarlo para utilizar PostgreSQL.

## Instalación de paquetes requeridos:

* Python 3 development:
```
sudo apt install python3-dev python3-venv
```

* PostgreSQL 13 development:
```
sudo apt install postgresql-server-dev-13
```

* Otros paquetes requeridos
```
sudo apt install libsasl2-dev libldap2-dev libssl-dev libxml2-dev libxslt1-dev libxmlsec1-dev libffi-dev pkg-config apt-transport-https virtualenv build-essential

```

* NodeJS LTS

(las instrucciones originales eran para instalar la versión 10 que ya estaba
deprecada hace tiempo, cuando lo instalé (2021-10) la versión LTS era la 16)
```
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install nodejs
```

* yarn
```
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update
sudo apt install yarn
```

## Clonar el código fuente y crear un virtualenv

Vamos a instalar en **`/opt/powerdns-admin`**:
```
sudo git clone https://github.com/ngoduykhanh/PowerDNS-Admin.git /opt/powerdns-admin
cd /opt/powerdns-admin
python3 -mvenv ./venv
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
