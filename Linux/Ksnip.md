## `ksnip` para capturar pantallas (screenshots) 
La página oficial es la de [github](https://github.com/ksnip/ksnip).
En Pop OS! 22.04 se puede instalar con apt o vía `flatpak`.

Si no lo instalamos antes con apt, usar flatpak:
```
# instalar paquete flatpak 
flatpak install --user flathub org.ksnip.ksnip
```
Carpeta para capturas
```
# crear una carpeta (dentro de ~/Pictures) para guardar las capturas
mkdir -pv ~/Pictures/Screenshots
```
Abrir la interfaz y cambiar algunas configuraciones (tocando `Alt+F7` o a través
del menú _Options_ &rarr; _Settings_):

![Application Settings](img/ksnip-settings-application.png)

![Saver Settings](img/ksnip-settings-application-saver.png)

**Capture save location and filename:**
```
/home/baby/Pictures/Screenshots/$Y$M$D-$T.png
```

![Tray Icon Settings](img/ksnip-settings-application-tray_icon.png)


En la ventana principal también conviene agregar un _delay_ para la captura:

![Screenshot Capture Delay](img/ksnip-editor-capture_delay.png)

Finalmente, configurarlo en las **Startup applications** para que arranque
automáticamente al iniciar la sesión:

```
cat <<EOF > ~/.config/autostart/ksnip.desktop
[Desktop Entry]
Name=ksnip
Comment=Capture and edit screenshots
Exec=flatpak run org.ksnip.ksnip
Type=Application
Terminal=false
Hidden=false
EOF
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
