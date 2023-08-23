# WinBox en Linux

Referencias: https://youtu.be/2jjtK5Me29I

* Instalar [Wine](https://www.winehq.org/):
```
sudo apt-get install wine
```

* Bajar [WinBox](https://mikrotik.com/download) 
```
# Creo un directorio para "instalar" WinBox
sudo mkdir -pv /opt/MikroTikWinBox

# Bajar la última versión de WinBox 64 bits
sudo curl --location --output /opt/MikroTikWinBox/winbox64.exe https://mt.lv/winbox64
```

* Ejecutar manualmente
```
wine /opt/MikroTikWinBox/winbox64.exe
```

* Crear una entrada en el escritorio
  * [Buscar un ícono
apropiado](https://www.google.com/search?q=winbox+icon&tbm=isch) en formato PNG
y guardarlo en `/opt/MikroTikWinBox/winbox.png`
  * Crear un archivo para lanzar la aplicación
```
cat << EOF| sudo tee /opt/MikroTikWinBox/WinBox.desktop
#!/usr/bin/env xdg-open
[Desktop Entry]
Name=MikroTik WinBox
Exec=wine /opt/MikroTikWinBox/winbox64.exe
Icon=/opt/MikroTikWinBox/winbox.png
Terminal=false
Type=Application
EOF
```
   * Hacer una copia en el escritorio y darle permisos de ejecución
```
cp /opt/MikroTikWinBox/WinBox.desktop $(xdg-user-dir DESKTOP)
gio set $(xdg-user-dir DESKTOP)/WinBox.desktop metadata::trusted true
chmod +x $(xdg-user-dir DESKTOP)/WinBox.desktop
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
