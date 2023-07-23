# Adaptador USB - serial

Conectando un adaptador USB a serial port me aparecieron mensajes raros en el
`journalctl`:

```
Sep 22 12:56:33 popovich kernel: usb 3-7: new full-speed USB device number 5 using xhci_hcd
Sep 22 12:56:33 popovich kernel: usb 3-7: New USB device found, idVendor=1a86, idProduct=7523, bcdDevice= 2.54
Sep 22 12:56:33 popovich kernel: usb 3-7: New USB device strings: Mfr=0, Product=2, SerialNumber=0
Sep 22 12:56:33 popovich kernel: usb 3-7: Product: USB2.0-Ser!
Sep 22 12:56:33 popovich kernel: ch341 3-7:1.0: ch341-uart converter detected
Sep 22 12:56:33 popovich kernel: ch341-uart ttyUSB0: break control not supported, using simulated break
Sep 22 12:56:33 popovich kernel: usb 3-7: ch341-uart converter now attached to ttyUSB0
Sep 22 12:56:33 popovich mtp-probe[38291]: checking bus 3, device 5: "/sys/devices/pci0000:00/0000:00:14.0/usb3/3-7"
Sep 22 12:56:33 popovich mtp-probe[38291]: bus: 3, device: 5 was not an MTP device
Sep 22 12:56:33 popovich systemd[1]: Starting Braille Device Support...
Sep 22 12:56:33 popovich brltty[38294]: BRLTTY 6.4 rev BRLTTY-6.4 [https://brltty.app/]
Sep 22 12:56:33 popovich brltty[38294]: executing as the invoking user: root
Sep 22 12:56:33 popovich brltty[38294]: BRLTTY 6.4 rev BRLTTY-6.4 [https://brltty.app/]
Sep 22 12:56:33 popovich brltty[38294]: brltty: executing as the invoking user: root
Sep 22 12:56:33 popovich mtp-probe[38305]: checking bus 3, device 5: "/sys/devices/pci0000:00/0000:00:14.0/usb3/3-7"
Sep 22 12:56:33 popovich mtp-probe[38305]: bus: 3, device: 5 was not an MTP device
Sep 22 12:56:34 popovich brltty[38294]: CLDR open error: No such file or directory: /usr/share/unicode/cldr/common/annotations/en.xml
Sep 22 12:56:34 popovich kernel: input: BRLTTY 6.4 Linux Screen Driver Keyboard as /devices/virtual/input/input17
Sep 22 12:56:34 popovich brltty[38294]: brltty: CLDR open error: No such file or directory: /usr/share/unicode/cldr/common/annotations/en.xml
Sep 22 12:56:34 popovich brltty[38294]: brltty: possible cause: the package that defines the CLDR annotations directory is not installed
Sep 22 12:56:34 popovich brltty[38294]: brltty: emoji substitutiion won't be performed
Sep 22 12:56:34 popovich brltty[38294]: brltty: BrlAPI Server: release 0.8.3
Sep 22 12:56:34 popovich brltty[38294]: brltty: Linux Screen Driver:
Sep 22 12:56:34 popovich systemd[1]: Started Braille Device Support.
Sep 22 12:56:34 popovich brltty[38294]: possible cause: the package that defines the CLDR annotations directory is not installed
Sep 22 12:56:34 popovich brltty[38294]: emoji substitutiion won't be performed
Sep 22 12:56:34 popovich brltty[38294]: BrlAPI Server: release 0.8.3
Sep 22 12:56:34 popovich brltty[38294]: Linux Screen Driver:
Sep 22 12:56:34 popovich systemd-logind[940]: Watching system buttons on /dev/input/event16 (BRLTTY 6.4 Linux Screen Driver Keyboard)
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (II) config/udev: Adding input device BRLTTY 6.4 Linux Screen Driver Keyboard (/dev/input/event16)
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (**) BRLTTY 6.4 Linux Screen Driver Keyboard: Applying InputClass "libinput keyboard catchall"
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (II) Using input driver 'libinput' for 'BRLTTY 6.4 Linux Screen Driver Keyboard'
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (II) systemd-logind: got fd for /dev/input/event16 13:80 fd 98 paused 0
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (**) BRLTTY 6.4 Linux Screen Driver Keyboard: always reports core events
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (**) Option "Device" "/dev/input/event16"
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (II) event16 - BRLTTY 6.4 Linux Screen Driver Keyboard: is tagged by udev as: Keyboard
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (II) event16 - BRLTTY 6.4 Linux Screen Driver Keyboard: device is a keyboard
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (II) event16 - BRLTTY 6.4 Linux Screen Driver Keyboard: device removed
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (**) Option "config_info" "udev:/sys/devices/virtual/input/input17/event16"
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (II) XINPUT: Adding extended input device "BRLTTY 6.4 Linux Screen Driver Keyboard" (type: KEYBOARD, id 12)
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (**) Option "xkb_layout" "latam"
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (II) event16 - BRLTTY 6.4 Linux Screen Driver Keyboard: is tagged by udev as: Keyboard
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (II) event16 - BRLTTY 6.4 Linux Screen Driver Keyboard: device is a keyboard
Sep 22 12:56:34 popovich brltty[38294]: USB configuration set error 16: Device or resource busy
Sep 22 12:56:34 popovich brltty[38294]: USB interface in use: 0 (ch341)
Sep 22 12:56:34 popovich brltty[38294]: brltty: USB configuration set error 16: Device or resource busy
Sep 22 12:56:34 popovich brltty[38294]: brltty: USB interface in use: 0 (ch341)
Sep 22 12:56:34 popovich brltty[38294]: brltty: USB control transfer error 32: Broken pipe
Sep 22 12:56:34 popovich kernel: usb 3-7: usbfs: interface 0 claimed by ch341 while 'brltty' sets config #1
Sep 22 12:56:34 popovich kernel: ch341-uart ttyUSB0: ch341-uart converter now disconnected from ttyUSB0
Sep 22 12:56:34 popovich kernel: ch341 3-7:1.0: device disconnected
Sep 22 12:56:34 popovich brltty[38294]: USB control transfer error 32: Broken pipe
Sep 22 12:56:34 popovich ModemManager[971]: <info>  [base-manager] port ttyUSB0 released by device '/sys/devices/pci0000:00/0000:00:14.0/usb3/3-7'
Sep 22 12:56:34 popovich ModemManager[971]: <info>  [base-manager] couldn't check support for device '/sys/devices/pci0000:00/0000:00:14.0/usb3/3-7': Operation was cancelled
Sep 22 12:56:42 popovich kernel: usb 3-7: USB disconnect, device number 5
Sep 22 12:56:42 popovich systemd[1]: Stopping Braille Device Support...
Sep 22 12:56:49 popovich rtkit-daemon[1167]: Supervising 10 threads of 7 processes of 1 users.
Sep 22 12:56:49 popovich rtkit-daemon[1167]: Supervising 10 threads of 7 processes of 1 users.
Sep 22 12:56:52 popovich systemd[1]: brltty-udev.service: State 'stop-sigterm' timed out. Killing.
Sep 22 12:56:52 popovich systemd[1]: brltty-udev.service: Killing process 38294 (brltty) with signal SIGKILL.
Sep 22 12:56:52 popovich acpid[913]: input device has been disconnected, fd 20
Sep 22 12:56:52 popovich /usr/libexec/gdm-x-session[2310]: (II) event16 - BRLTTY 6.4 Linux Screen Driver Keyboard: device removed
Sep 22 12:56:52 popovich /usr/libexec/gdm-x-session[2310]: (II) config/udev: removing device BRLTTY 6.4 Linux Screen Driver Keyboard
Sep 22 12:56:52 popovich /usr/libexec/gdm-x-session[2310]: (**) Option "fd" "98"
Sep 22 12:56:52 popovich /usr/libexec/gdm-x-session[2310]: (II) UnloadModule: "libinput"
Sep 22 12:56:52 popovich /usr/libexec/gdm-x-session[2310]: (II) systemd-logind: releasing fd for 13:80
Sep 22 12:56:52 popovich systemd[1]: brltty-udev.service: Main process exited, code=killed, status=9/KILL
Sep 22 12:56:52 popovich systemd[1]: brltty-udev.service: Failed with result 'timeout'.
Sep 22 12:56:52 popovich systemd[1]: Stopped Braille Device Support.
```

En particular las líneas sorprendentes son estas:
```
Sep 22 12:56:33 popovich brltty[38294]: BRLTTY 6.4 rev BRLTTY-6.4 [https://brltty.app/]
Sep 22 12:56:33 popovich brltty[38294]: BRLTTY 6.4 rev BRLTTY-6.4 [https://brltty.app/]
Sep 22 12:56:33 popovich brltty[38294]: brltty: executing as the invoking user: root
Sep 22 12:56:34 popovich kernel: input: BRLTTY 6.4 Linux Screen Driver Keyboard as /devices/virtual/input/input17
Sep 22 12:56:34 popovich brltty[38294]: brltty: CLDR open error: No such file or directory: /usr/share/unicode/cldr/common/annotations/en.xml
Sep 22 12:56:34 popovich brltty[38294]: brltty: possible cause: the package that defines the CLDR annotations directory is not installed
Sep 22 12:56:34 popovich brltty[38294]: brltty: emoji substitutiion won't be performed
Sep 22 12:56:34 popovich brltty[38294]: brltty: BrlAPI Server: release 0.8.3
Sep 22 12:56:34 popovich brltty[38294]: brltty: Linux Screen Driver:
Sep 22 12:56:34 popovich systemd-logind[940]: Watching system buttons on /dev/input/event16 (BRLTTY 6.4 Linux Screen Driver Keyboard)
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (II) config/udev: Adding input device BRLTTY 6.4 Linux Screen Driver Keyboard (/dev/input/event16)
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (**) BRLTTY 6.4 Linux Screen Driver Keyboard: Applying InputClass "libinput keyboard catchall"
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (II) Using input driver 'libinput' for 'BRLTTY 6.4 Linux Screen Driver Keyboard'
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (**) BRLTTY 6.4 Linux Screen Driver Keyboard: always reports core events
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (II) event16 - BRLTTY 6.4 Linux Screen Driver Keyboard: is tagged by udev as: Keyboard
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (II) event16 - BRLTTY 6.4 Linux Screen Driver Keyboard: device is a keyboard
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (II) event16 - BRLTTY 6.4 Linux Screen Driver Keyboard: device removed
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (II) XINPUT: Adding extended input device "BRLTTY 6.4 Linux Screen Driver Keyboard" (type: KEYBOAR>
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (II) event16 - BRLTTY 6.4 Linux Screen Driver Keyboard: is tagged by udev as: Keyboard
Sep 22 12:56:34 popovich /usr/libexec/gdm-x-session[2310]: (II) event16 - BRLTTY 6.4 Linux Screen Driver Keyboard: device is a keyboard
Sep 22 12:56:34 popovich brltty[38294]: brltty: USB configuration set error 16: Device or resource busy
Sep 22 12:56:34 popovich brltty[38294]: brltty: USB interface in use: 0 (ch341)
Sep 22 12:56:34 popovich brltty[38294]: brltty: USB control transfer error 32: Broken pipe
Sep 22 12:56:34 popovich kernel: usb 3-7: usbfs: interface 0 claimed by ch341 while 'brltty' sets config #1
Sep 22 12:56:52 popovich systemd[1]: brltty-udev.service: State 'stop-sigterm' timed out. Killing.
Sep 22 12:56:52 popovich systemd[1]: brltty-udev.service: Killing process 38294 (brltty) with signal SIGKILL.
Sep 22 12:56:52 popovich /usr/libexec/gdm-x-session[2310]: (II) event16 - BRLTTY 6.4 Linux Screen Driver Keyboard: device removed
Sep 22 12:56:52 popovich /usr/libexec/gdm-x-session[2310]: (II) config/udev: removing device BRLTTY 6.4 Linux Screen Driver Keyboard
Sep 22 12:56:52 popovich systemd[1]: brltty-udev.service: Main process exited, code=killed, status=9/KILL
Sep 22 12:56:52 popovich systemd[1]: brltty-udev.service: Failed with result 'timeout'.
```
ya que [BRLTTY](https://brltty.app/) es un daemon de actualización de consola 
para displays en braille.

Por suerte, encontré [esto en 
StackOverflow](https://stackoverflow.com/a/70123897/4102803) que me llevó a 
resolverlo.

En Pop! OS 22.04 las reglas de udev para brltty están en otro archivo, pero la
solución es simplemente deshabilitarlas. Esto debería andar en forma genérica:

```
for rulefile in /usr/lib/udev/rules.d/*brltty*.rules ; do
  sudo mv -v ${rulefile} ${rulefile}.disabled
done

sudo udevadm control --reload-rules
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
