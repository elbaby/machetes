# Actualizar el firewall dinámicamente con fail2ban

[fail2ban](https://github.com/fail2ban/fail2ban) es un utilitario que lee los
archivos de log y en base a los "errores" detectados allí puede bloquear
direcciones IP utilizando el firewall del equipo (usualmente `iptables`).

fail2ban **no es** un IDS, un IPS ni un WAF. Es una herramienta simple y básica
que sólo recorre logs y bloquea direcciones con reglas predefinidas.

La documentación está en la [wiki en el proyecto de
github](https://github.com/fail2ban/fail2ban/wiki).

## Instalación

En Debian y Ubuntu hay versiones razonablemente actuales en los repositorios, lo
más conveniente es instalar ese paquete:
```
sudo apt install fail2ban
```
Luego hay que habilitar el servicio e iniciarlo por primera vez:
```
sudo systemctl enable fail2ban.service
sudo systemctl start fail2ban.service
```
### nftables

En los kernels modernos, Netfilter utiliza el _framework_
[nftables](https://es.wikipedia.org/wiki/Nftables) que es más eficiente que el
viejo [xtables](https://es.wikipedia.org/wiki/Iptables).

El comando `iptables` que puede estar disponible es en realidad un link al
comando `iptables-nft`.

Para poder ver las tablas completas de nftables conviene instalar el paquete
`nftables` que contiene el comando `nft`.
```
sudo apt install nftables
```


## Configuración

Las configuraciones de fail2ban están en el directorio **`/etc/fail2ban`**.

**NO MODIFICAR LOS ARCHIVOS DE CONFIGURACIÓN QUE VIENEN EN LA DISTRIBUCIÓN**

Las configuraciones deben hacerse, o bien en archivos nuevos en
`/etc/fail2ban/fail2ban.d` o bien copiando cada archivo `xxxx.conf` que se
quiere modificar a un archivo `xxxx.local`.

Los archivos con extensión `.local` _siempre_ toman precedencia por sobre los
archivos con extensión `.conf`.

Para hacer las configuraciones generales, se pueden copiar los archivos
`fail2ban.conf` y `jail.conf` a `fail2ban.local` y `jail.local` respectivamente
y hacer allí las modificaciones:
```
sudo cp -p /etc/fail2ban/fail2ban.conf /etc/fail2ban/fail2ban.local
sudo cp -p /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Parámetros que puede ser interesante modificar en `/etc/fail2ban/jail.local`:
* `ignoreip`: lista de direcciones IP que nunca se deben bloquear. Por default
aquí sólo van las direcciones `localhost`. Se pueden agregar redes o direcciones
propias que se consideran _seguras_.
* `bantime`: el tiempo de _banneo_ por default de una IP (esto se puede
configurar en cada _jail_ específica. De todos modos, el _default_ de 10 minutos
me parece muy corto. Puede ser conveniente que esto sea entre una hora y una
semana.
* `maxretry`: la cantidad de "errores/fallas/ataques" admitidos _antes_ de
_bannear_ un host. **`3`** me parece un buen valor.
* `findtime`: el tiempo durante el cual se miden los "maxretries". Esto es, un
host se _bannea_ si se producen _maxretry_ errores en _findtime_ o menos tiempo.
* `banaction` y `banaction_allports` son los comandos que se usan para _bannear_
direcciones. Al menos hasta la versión 1.0.2, el default viene configurado
para usar `iptables`. Conviene modificarlo para que utilize `nft`:
```
[DEFAULT]
banaction = nftables-multiport
banaction_allports = nftables-allports
```
(OBSOLETO: Si se está usando
[Shorewall](https://github.com/elbaby/machetes/blob/master/Linux/Shorewall.md#soporte-en-fail2ban)
en lugar de `iptables` _standalone_, en estos campos hay que especificar
`shorewall`).

La configuración básica podría ser la siguiente:
```
[DEFAULT]
banaction = nftables-multiport
banaction_allports = nftables-allports
# no bloquear conexiones locales y de redes privadas (suponiendo que confiamos
# en las redes privadas locales)
ignoreip = 127.0.0.1/8 ::1 10.0.0.0/8 172.16.0.0/12 192.168.0.0/16 100.64.0.0/10
bantime  = 180d
findtime = 5m
maxretry = 3

[sshd]
# En general los ataques ssh pueden intentar atacar otros puertos,
# por lo que bloqueamos todos los puertos desde esas IPs
action = nftables-allports
```

Luego de modificar la configuración, hay que recargar el servidor para que la
tome:
```
sudo systemctl reload fail2ban.service
```

## Habilitación de un jail

En Ubuntu y Debian el único jail que viene habilitado es el `ssh` (en el
archivo `/etc/fail2ban/jail.d/defaults-debian.conf`.

Para habilitar un jail hay que poner los parámetros específicos junto con
`enabled = true` en un archivo en el directorio `/etc/fail2ban/jail.d`.

En `/etc/fail2ban/jail.conf` hay varios ejemplos de parámetros para jails
específicos.

Las definiciones de los filtros que usan los jails están en el directorio
`/etc/fail2ban/filter.d`. Como en el resto de los casos, si se quiere modificar
un filtro, hay que copiarlo con la extensión `.local` y modificar ese.

Por ejemplo, se podrían habilitar varios filtros para el server apache poniendo
un archivo `/etc/fail2ban/jail.d/apache.local` con un contenido similar al
siguiente:
```
[apache-badbots]
# Ban hosts which agent identifies spammer robots crawling the web
# for email addresses. The mail outputs are buffered.
enabled  = true
port     = http,https
logpath  = %(apache_access_log)s
bantime  = 48h
maxretry = 1

[apache-fakegooglebot]
enabled  = true
port     = http,https
logpath  = %(apache_access_log)s
maxretry = 1
ignorecommand = %(ignorecommands_dir)s/apache-fakegooglebot <ip>

[apache-nohome]
enabled  = true
port     = http,https
logpath  = %(apache_error_log)s
maxretry = 2

[apache-shellshock]
enabled  = true
port    = http,https
logpath = %(apache_error_log)s
maxretry = 1
```

Para ver los filtros específicos, hay que revisar los archivos
`apache-badbots.conf`, `apache-fakegooglebot.conf`, `apache-nohome.conf` y
`apache-shellshock.conf` en `/etc/fail2ban/filter.d`.

## Uso

## Ver listado de todas las jails
```
sudo fail2ban-client status
```
Ejemplo de la salida:
```
Status
|- Number of jail:	9
`- Jail list:	apache-auth, apache-badbots, apache-botsearch, apache-nohome, apache-noscript, apache-overflows, apache-shellshock, sshd, wordpress
```

### Ver estado de un jail en particular (e.g: sshd)
```
sudo fail2ban-client status
```
Ejemplo de la salida:
```
Status for the jail: sshd
|- Filter
|  |- Currently failed:	1
|  |- Total failed:	2
|  `- File list:	/var/log/auth.log
`- Actions
   |- Currently banned:	7
   |- Total banned:	3456
   `- Banned IP list:	1.194.218.133 1.214.197.163 1.238.106.229 1.55.33.86 1.71.9.130 1.82.220.20 1.9.107.43
```

### Ver todas las direcciones banneadas en todas las jaulas
```
sudo fail2ban-client banned
```
Ejemplo de la salida (es una sola línea):
```
[{'sshd': ['1.194.218.133', '1.214.197.163', '1.238.106.229', '1.55.33.86', '1.71.9.130', '1.82.220.20', '1.9.107.43']}, {'apache-auth': []}, {'apache-badbots': []}, {'apache-noscript': []}, {'apache-overflows': []}, {'apache-nohome': []}, {'apache-botsearch': []}, {'apache-shellshock': []}, {'wordpress': []}]
```

### Bannear una IP manualmente para una jaula
```
sudo fail2ban-client set sshd banip 1.2.3.4
```
Ejemplo de la salida (la cantidad de IPs nuevas banneadas):
```
1
```

### Desbannear una IP manualmente para una jaula
```
sudo fail2ban-client set sshd unbanip 1.2.3.4
```
Ejemplo de la salida (la cantidad de IPs desbanneadas):
```
1
```

### Ver todos los comandos disponibles
```
sudo fail2ban-client --help
```
Ejemplo de la salida:
```
Usage: fail2ban-client [OPTIONS] <COMMAND>

Fail2Ban v1.0.2 reads log file that contains password failure report
and bans the corresponding IP addresses using firewall rules.

Options:
    -c, --conf <DIR>        configuration directory
    -s, --socket <FILE>     socket path
    -p, --pidfile <FILE>    pidfile path
    --pname <NAME>          name of the process (main thread) to identify instance (default fail2ban-server)
    --loglevel <LEVEL>      logging level
    --logtarget <TARGET>    logging target, use file-name or stdout, stderr, syslog or sysout.
    --syslogsocket auto|<FILE>
    -d                      dump configuration. For debugging
    --dp, --dump-pretty     dump the configuration using more human readable representation
    -t, --test              test configuration (can be also specified with start parameters)
    -i                      interactive mode
    -v                      increase verbosity
    -q                      decrease verbosity
    -x                      force execution of the server (remove socket file)
    -b                      start server in background (default)
    -f                      start server in foreground
    --async                 start server in async mode (for internal usage only, don't read configuration)
    --timeout               timeout to wait for the server (for internal usage only, don't read configuration)
    --str2sec <STRING>      convert time abbreviation format to seconds
    -h, --help              display this help message
    -V, --version           print the version (-V returns machine-readable short format)

Command:
                                             BASIC
    start                                    starts the server and the jails
    restart                                  restarts the server
    restart [--unban] [--if-exists] <JAIL>   restarts the jail <JAIL> (alias
                                             for 'reload --restart ... <JAIL>')
    reload [--restart] [--unban] [--all]     reloads the configuration without
                                             restarting of the server, the
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
