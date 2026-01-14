# Renombrar los screenshots de GNOME 3

Las versiones nuevas de [GNOME Shell](https://www.gnome.org/) (a partir de la
42) tienen la utilidad de captura de pantalla integrada (ya no se hace más con
[GNOME Screenshot](https://gitlab.gnome.org/Archive/gnome-screenshot).

Pero de acuerdo a la _filosofía_ de GNOME, no es configurable ni la ubicación
ni el nombre de los archivos donde se guardan dichas capturas.

En GNOME 48 tuve problemas para utilizar [ksnip](Ksnip.md), pero con algo de
[ayuda de
ChatGPT](https://chatgpt.com/share/6966f6fe-2a4c-8006-85e9-38185dc875ee)
conseguí un método simple para al menos renombrar los archivos creados al
formato que me gusta.

Primero instalamos `inotify-tools` para poder monitorear los archivos que la
utilidad de screenshot va creando.

```bash
sudo apt install inotify-tools
```

Luego creamos un script para monitorear el directorio y renombrar los archivos
```bash
# creamos el directorio si no existe
mkdir -pv ~/bin

# creamos el script
cat > ~/bin/rename-screenshots.sh <<EOF
#!/bin/sh

WATCHDIR="\${HOME}/Pictures/Screenshots"

inotifywait -m -e close_write --format "%f" "\${WATCHDIR}" | while read FILE; do
    # Match GNOME's default filename pattern
    # Screenshot From YYYY-MM-DD HH-SS-SS.png
    if echo "\${FILE}" | grep -Eq '^Screenshot From [0-9]{4}-[0-9]{2}-[0-9]{2} [0-9]{2}-[0-9]{2}-[0-9]{2}\.png$'; then
        # Extract timestamp
        TS=\$(echo "\${FILE}" | sed -E 's/Screenshot From ([0-9]{4})-([0-9]{2})-([0-9]{2}) ([0-9]{2})-([0-9]{2})-([0-9]{2})\.png/\1\2\3-\4\5\6/')
        mv -- "\${WATCHDIR}/\${FILE}" "\${WATCHDIR}/\${TS}.png"
    fi
done
EOF

# hacer el script ejecutable
chmod +x ~/bin/rename-screenshots.sh
```
Creamos un servicio `systemd` de usuario para ejecutar automáticamente el script
y lo configuramos.

```bash
# creamos el directorio si no existe
mkdir -pv ~/.config/systemd/user

# armamos el servicio systemd
cat > ~/.config/systemd/user/rename-screenshots.service <<EOF
[Unit]
Description=Rename GNOME screenshots automatically

[Service]
ExecStart=%h/bin/rename-screenshots.sh
Restart=always

[Install]
WantedBy=default.target
EOF


# habilitamos el servicio y lo arrancamos
systemctl --user daemon-reload
systemctl --user enable --now rename-screenshots.service
```

A partir de ahora, cada vez que se guarde un archivo en `~/Pictures/Screenshots`
con el formato `Screenshot From YYYY-MM-DD hh-mm-ss.png` lo va a renombrar
inmediatamente a `YYYYMMDD-hhmmss.png`


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
