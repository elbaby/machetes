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

![Create Profile Wizard](img/thunderbird-03-create_profile.png)

![Create Profile Wizard](img/thunderbird-04-exit_profile.png)

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
![Create Profile Wizard](img/thunderbird-05-profile_restart.png)


A continuación, ofrece crear una nueva cuenta, pero cancelamos para configurar
algunas cosas _antes_ de crear la primera cuenta:

![Account Setup Cancel](img/thunderbird-06-account_setup_cancel.png)

![Account Setup Cancel](img/thunderbird-07-account_setup_cancel-exit.png)

## Configuraciones de correo electrónico

### Configuraciones Generales

![Settings](img/thunderbird-08-settings.png)

![Settings/Language](img/thunderbird-09-settings_language.png)

![Settings/Language](img/thunderbird-10-settings_language.png)

![Settings/Language](img/thunderbird-11-settings_language.png)

![Settings/Language/Restart](img/thunderbird-12-settings_language-restart.png)

![Settings/Alerts](img/thunderbird-13-settings_alerts.png)

![Settings/Reading & Display](img/thunderbird-14-settings_reading.png)

![Settings/Return Receipts](img/thunderbird-15-settings_return_receipts.png)

Utilizar **maildir** en lugar de **mbox** para las carpetas de mail:

![Settings/maildir](img/thunderbird-16-settings_maildir.png)

Hacer que las carpetas abran por default con orden cronológico descendente (el
mail más nuevo arriba):

![Settings/Config Editor/Default Sort Order](img/thunderbird-17-settings_confedit_default_sort.png)

![Settings/Config Editor/Default Sort Order](img/thunderbird-18-settings_confedit_default_sort.png)

#### OPCIONAL

Este cambio sólo hay que hacerlo si nos vamos a conectar a un servidor que no
soporta versiones de TLS nuevas:

![Settings/Config Editor/TLS Minimum Version (optional)](img/thunderbird-19-settings_confedit_tls_minversion.png)

![Settings/Config Editor/TLS Minimum Version (optional)](img/thunderbird-20-settings_confedit_tls_minversion.png)

### Configuraciones para la Redacción de Mensajes

![Settings/Composition](img/thunderbird-21-settings_dl_more_dicts.png)

![Settings/Composition](img/thunderbird-22-settings_dl_langpack-es-ar.png)

![Settings/Composition](img/thunderbird-23-settings_inst_langpack-es-ar.png)

![Settings/Composition](img/thunderbird-24-settings_composition.png)

![Settings/Attachments Keywords](img/thunderbird-25-settings_attach_keywords.png)

![Settings/Attachments Keywords](img/thunderbird-26-settings_attach_keywords.png)

![Settings/Attachments Keywords](img/thunderbird-27-settings_attach_keywords.png)

Para finalizar, cerramos el tab de preferencias:

![Settings/Close Tab](img/thunderbird-28-settings_close.png)

## Configurar cuenta de correo de GMail

![New Account](img/thunderbird-29-newaccount.png)

![New Account/Existing Email](img/thunderbird-30-newaccount.png)

![New Account/Existing Email](img/thunderbird-31-newaccount.png)

Seleccionar IMAP

![New Account/Gmail IMAP](img/thunderbird-32-accountsetup_imap.png)

### Autenticarse con Google (OAuht) y permitir el acceso

![Google Sign in](img/thunderbird-33-google_sign_in_email.png)

![Google Sign in](img/thunderbird-34-google_sign_in_password.png)

A continuación, seleccionar el método de autenticación por 2° factor que
prefieras:

![Google Sign in](img/thunderbird-35-google_sign_in_2fa.png)

![Google Sign in](img/thunderbird-36-google_sign_in_2fa.png)

Una vez validado el 2° factor aparecerá el pedido de permisos necesarios:

![Account Setup/Allow Google Account Access](img/thunderbird-37-google_oauth_allow.png)

### Conectar libreta de direcciones y calendario de Google

![Account Setup/Google Address Books & Calendars](img/thunderbird-38-google_linked_services.png)

![Account Setup/Google Calendar Refresh Period](img/thunderbird-39-google_linked_services-calendar_refresh.png)

![Account Setup/Google Address Books & Calendars](img/thunderbird-40-google_linked_services.png)

## Configuraciones de calendarios

![Settings](img/thunderbird-08-settings.png)

![Settings/Calendar](img/thunderbird-41-settings_calendar.png)

![Settings/Calendar](img/thunderbird-42-settings_calendar.png)


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
