# SSH - Negociación de algoritmo de firma mutuo

Los nuevos clientes SSH piden password al conectarse a un server viejo aun
cuando el servidor tiene la clave pública del cliente.

El error que se ve al mandar la opción `-v` es:
```
debug1: send_pubkey_test: no mutual signature algorithm
```

La solución la encontré en el [AskUbuntu de 
StackExchange](https://askubuntu.com/questions/1404049/ssh-without-password-does-not-work-after-upgrading-from-18-04-to-22-04):

Para un uso muy eventual, agregar la opción `-o PubkeyAcceptedKeyTypes=+ssh-rsa`
en la invocación del cliente:
```
ssh -o PubkeyAcceptedKeyTypes=+ssh-rsa user@server
```

La otra opción es agregarla en el archivo de configuración personal del cliente
`~/.ssh/config`:
```
# https://askubuntu.com/questions/1404049/ssh-without-password-does-not-work-after-upgrading-from-18-04-to-22-04
PubkeyAcceptedKeyTypes +ssh-rsa
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
