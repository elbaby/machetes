# Instalación de AnyDesk en linux

```
# repositorio AnyDesk
curl --fail --silent --show-error --location \
    https://keys.anydesk.com/repos/DEB-GPG-KEY | \
    gpg --dearmor | \
    sudo tee /etc/apt/keyrings/anydesk-keyring.gpg >/dev/null
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/anydesk-keyring.gpg] http://deb.anydesk.com/ all main" |\
    sudo tee /etc/apt/sources.list.d/anydesk-stable.list > /dev/null

# actualizar paquetes
sudo apt update

# instalar AnyDesk
sudo apt install anydesk
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
