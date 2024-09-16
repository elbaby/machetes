## Instalar el cliente NordVPN sin autorizar a nordvpn para que firme cualquier paquete

Cuando se instala [NordVPN](https://nordvpn.com/) en Linux con las
[instrucciones oficiales](https://support.nordvpn.com/hc/en-us/articles/20196094470929-Installing-NordVPN-on-Linux-distributions)
el instalador configura los [repositorios
oficiales](https://repo.nordvpn.com//deb/nordvpn/debian)
pero agrega la [clave pública de firma de software de
NordVPN](https://repo.nordvpn.com/gpg/nordvpn_public.asc) en el directorio
`/etc/apt/trusted.gpg.d` lo que hace que automáticamente nuestro equipo confíe
en cualquier cosa que firme NordVPN, lo cual no es una buena política.

Con los siguientes pasos, podemos instalar NordVPN sin hacer esto
```
# crear el directorio para keyrings si no existe
sudo mkdir --parents --verbose /etc/apt/keyrings

# bajar la clave pública de NordVPN y guardarla en formato gpg binario
curl --fail --silent --show-error --location https://repo.nordvpn.com/gpg/nordvpn_public.asc \
    | gpg --dearmor \
    | sudo tee /etc/apt/keyrings/nordvpn.gpg > /dev/null

# configurar el repositorio para que valide con la clave pública que bajamos
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/nordvpn.gpg] https://repo.nordvpn.com//deb/nordvpn/debian stable main" \
    | sudo tee /etc/apt/sources.list.d/nordvpn.list >/dev/null

# instalar el cliente nordvp
sudo apt install nordvp

# Crear el grupo nordvpn
sudo groupadd nordvpn
# Agregar nuestro usuario al grupo nordvpn
sudo usermod -aG nordvpn $USER
```

Después de hacer esto es necesario rebootear el equipo. Luego hay que
loguearse a NordVPN
```
nordvpn login
```

Esto nos da un URL para loguearnos via web.

Si el equipo no tiene GUI, seguir [estas
instrucciones](https://support.nordvpn.com/hc/en-us/articlesa/20226600447633-How-to-log-in-to-NordVPN-on-Linux-devices-without-a-GUI).

Más información sobre los comandos
[acá](https://support.nordvpn.com/hc/en-us/articles/20196094470929-Installing-NordVPN-on-Linux-distributions#h_01HGZ1TXZPXTNBHJBZYBQCPWCY)

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
