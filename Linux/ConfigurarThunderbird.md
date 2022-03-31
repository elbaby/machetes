# Instalación y configuración del cliente de mail Mozilla Thunderbird

## Instalación

Instalar el thunderbird
```
sudo apt install thunderbird
```
Ahora vamos a hacer una configuración personalizada.

## Profile Manager

Para eso, lanzamos el _Profile Manager_ **antes** de arrancarlo por primera vez.
Desde una terminal (dentro del entorno gráfico), tipear:
```
thunderbird -ProfileManager
```
Esto va a lanzar el _Profile Manager_.

### Crear profile nuevo con nombre y carpeta en ubicación personalizada

![Create Profile](img/thunderbird-01-create_profile.png)
![Create Profile Wizard](img/thunderbird-02-create_profile.png)
![Choose Folder](img/thunderbird-03-profile_choose_folder.png)
![Create Folder](img/thunderbird-04-profile_create_folder.png)
![Create Folder](img/thunderbird-05-profile_create_folder.png)
![Create Folder](img/thunderbird-06-profile_create_folder.png)
![Create Folder](img/thunderbird-07-profile_create_folder.png)

A continuación, ofrece crear una nueva cuenta, pero cancelamos para configurar
algunas cosas _antes_ de crear la primera cuenta:

![Account Setup Cancel](img/thunderbird-08-account_setup_cancel.png)

## Preferencias de correo electrónico

![Preferences](img/thunderbird-09-preferences.png)
![Preferences/Language](img/thunderbird-10-preferences_language.png)
![Preferences/Language](img/thunderbird-11-preferences_language.png)
![Preferences/Language](img/thunderbird-12-preferences_language.png)
![Preferences/Language/Restart](img/thunderbird-13-preferences_language-restart.png)
![Preferences/Alerts](img/thunderbird-14-preferences_alerts.png)
![Preferences/Reading & Display](img/thunderbird-15-preferences_reading.png)
![Preferences/Return Receipts](img/thunderbird-16-preferences_return_receipts.png)

Utilizar **maildir** en lugar de **mbox** para las carpetas de mail:
![Preferences/maildir](img/thunderbird-17-preferences_maildir.png)

Hacer que las carpetas abran por default con orden cronológico descendente (el
mail más nuevo arriba):
![Preferences/Config Editor/Default Sort Order](img/thunderbird-18-preferences_confedit_default_sort.png)
![Preferences/Config Editor/Default Sort Order](img/thunderbird-19-preferences_confedit_default_sort.png)

Este cambio sólo hay que hacerlo si nos vamos a conectar a un servidor que no
soporta versiones de TLS nuevas:
![Preferences/Config Editor/TLS Minimum Version (optional)](img/thunderbird-20-preferences_confedit_tls_version.png)
![Preferences/Config Editor/TLS Minimum Version (optional)](img/thunderbird-21-preferences_confedit_tls_version.png)

Configuraciones para la redacción de mensajes:
![Preferences/Composition](img/thunderbird-22-preferences_composition.png)
![Preferences/Attachments Keywords](img/thunderbird-23-preferences_attach_keywords.png)
![Preferences/Attachments Keywords](img/thunderbird-24-preferences_attach_keywords.png)
![Preferences/Attachments Keywords](img/thunderbird-25-preferences_attach_keywords.png)

Para finalizar, cerramos el tab de preferencias:
![Preferences/Close Tab](img/thunderbird-26-preferences_close.png)

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
<a rel="licencia" href="http://creativecommons.org/licenses/by-sa/4.0/deed.es">
<img alt="Creative Commons License" style="border-width:0"
src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a>
<br /><br />
Este documento está licenciado en los términos de una <a rel="licencia"
href="http://creativecommons.org/licenses/by-sa/4.0/deed.es">
Licencia Atribución-CompartirIgual 4.0 Internacional de Creative Commons</a>.
<br /><br />
This document is licensed under a <a rel="license" 
href="http://creativecommons.org/licenses/by-sa/4.0/deed.en">
Creative Commons Attribution-ShareAlike 4.0 International License</a>.
<!-- END --> 
