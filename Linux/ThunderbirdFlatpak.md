# Instalación y configuración del cliente de mail Mozilla Thunderbird (usando el paquete de Flatpak)

## Instalación

Instalar el thunderbird en modo usuario
```
flatpak install org.mozilla.Thunderbird
```
Ahora vamos a hacer una configuración personalizada.

## Profile Manager

Para eso, lanzamos el _Profile Manager_ **antes** de arrancarlo por primera vez.
Desde una terminal (dentro del entorno gráfico), tipear:
```
flatpak run --branch=stable --arch=x86_64 --env=LC_ALL=${LANG} --command=thunderbird --file-forwarding org.mozilla.Thunderbird -ProfileManager
```
Esto va a lanzar el _Profile Manager_.

### Crear profile nuevo con nombre y carpeta en ubicación personalizada

![Create Profile](img/thunderbird-01-create_profile.png)

![Create Profile Wizard](img/thunderbird-02-create_profile.png)

![Create Profile Wizard](img/thunderbird-07-create_profile.png)

![Create Profile Wizard](img/thunderbird-07-create_profile_a-exit.png)

### Renombrar (y eventualmente mover) el directorio de perfil

Ahora vamos a renombrar el directorio del perfil para que quede fijo:
```
cd ~/.var/app/org.mozilla.Thunderbird/.thunderbird
NOMBREDIR=`echo *.baby`
mv -v ${NOMBREDIR} baby.profile
sed -i.BACKUP -e s/${NOMBREDIR}/baby.profile/g profiles.ini
```

Opcionalmente, lo podemos mover a otro filesystem (para que no ocupe petabytes
en el `home`):
```
# El ${DESTDIR} NO DEBE EXISTIR!! (y debe ser un path absoluto)
DESTDIR=/d/baby/thunderbird-profiles


cd ~/.var/app/org.mozilla.Thunderbird/
mv -v .thunderbird ${DESTDIR}
ln -sv ${DESTDIR} .thunderbird
```

## Configuraciones previas a la creación de la cuenta
Volvemos a arrancar el Thunderbird
```
flatpak run --branch=stable --arch=x86_64 --env=LC_ALL=${LANG} --command=thunderbird --file-forwarding org.mozilla.Thunderbird -ProfileManager
```
![Create Profile Wizard](img/thunderbird-07-create_profile_b-start.png)


A continuación, ofrece crear una nueva cuenta, pero cancelamos para configurar
algunas cosas _antes_ de crear la primera cuenta:

![Account Setup Cancel](img/thunderbird-08-account_setup_cancel.png)

![Account Setup Cancel](img/thunderbird-08-account_setup_cancel_exit-setup.png)

## Configuraciones de correo electrónico

### Configuraciones Generales

![Settings](img/thunderbird-09-preferences.png)

![Settings/Language](img/thunderbird-10-preferences_language.png)

![Settings/Language](img/thunderbird-11-preferences_language.png)

![Settings/Language](img/thunderbird-12-preferences_language.png)

![Settings/Language/Restart](img/thunderbird-13-preferences_language-restart.png)

![Settings/Alerts](img/thunderbird-14-preferences_alerts.png)

![Settings/Reading & Display](img/thunderbird-15-preferences_reading.png)

![Settings/Return Receipts](img/thunderbird-16-preferences_return_receipts.png)

Utilizar **maildir** en lugar de **mbox** para las carpetas de mail:

![Settings/maildir](img/thunderbird-17-preferences_maildir.png)

Hacer que las carpetas abran por default con orden cronológico descendente (el
mail más nuevo arriba):

![Settings/Config Editor/Default Sort Order](img/thunderbird-18-preferences_confedit_default_sort.png)

![Settings/Config Editor/Default Sort Order](img/thunderbird-19-preferences_confedit_default_sort.png)

Este cambio sólo hay que hacerlo si nos vamos a conectar a un servidor que no
soporta versiones de TLS nuevas:

![Settings/Config Editor/TLS Minimum Version (optional)](img/thunderbird-20-preferences_confedit_tls_version.png)

![Settings/Config Editor/TLS Minimum Version (optional)](img/thunderbird-21-preferences_confedit_tls_version.png)

### Configuraciones para la Redacción de Mensajes

![Settings/Composition](img/thunderbird-22-preferences_composition_a_download_dics.png)

![Settings/Composition](img/thunderbird-22-preferences_composition_b_langpac-es-ar_dl.png)

![Settings/Composition](img/thunderbird-22-preferences_composition_c_langpac-es-ar_inst.png)

![Settings/Composition](img/thunderbird-22-preferences_composition.png)

![Settings/Attachments Keywords](img/thunderbird-23-preferences_attach_keywords.png)

![Settings/Attachments Keywords](img/thunderbird-24-preferences_attach_keywords.png)

![Settings/Attachments Keywords](img/thunderbird-25-preferences_attach_keywords.png)

Para finalizar, cerramos el tab de preferencias:

![Settings/Close Tab](img/thunderbird-26-preferences_close.png)

## Configurar cuenta de correo de GMail

![Account Settings/Email](img/thunderbird-27-accountsettings_email.png)

![Account Settings/Set Up GMail](img/thunderbird-28-accountsettings_gmail.png)

Utilizamos IMAP

![Account Settings/Gmail IMAP](img/thunderbird-29-accountsettings_gmail_imap.png)

Autenticarse con Google y permitir el acceso

![Account Settings/Gmail Allow Google Account Access](img/thunderbird-30-accountsettings_gmail_allow.png)

Conectar libreta de direcciones y calendario de Google:

![Account Settings/Google Address Books & Calendars](img/thunderbird-31-accountsettings_gmail_addressbook_calendar.png)

![Account Settings/Google Calendar Refresh Period](img/thunderbird-32-accountsettings_gmail_calendar_refresh.png)

![Account Settings/Google Address Books & Calendars](img/thunderbird-33-accountsettings_gmail_addressbook_calendar.png)

Configurar las preferencias del Calendario:

![Preferences](img/thunderbird-09-preferences.png)

![Preferences/Calendar](img/thunderbird-34-preferences_calendar.png)

![Preferences/Calendar](img/thunderbird-35-preferences_calendar.png)


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
