# Actualizar el firewall dinámicamente con fail2ban

[fail2ban](https://github.com/fail2ban/fail2ban) es un utilitario que lee los
archivos de log y en base a los "errores" detectados allí puede bloquear
direcciones IP utilizando el firewall del equipo (usualmente `iptables`).

fail2ban **no es** un IDS, un IPS ni un WAF. Es una herramienta simple y básica
que sólo recorre logs y bloquea direcciones con reglas predefinidas.

La documentación está en la [wiki en el proyecto de
github](https://github.com/fail2ban/fail2ban/wiki).

# Instalación

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
## nftables

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


# Configuración

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

# Habilitación de un jail

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

# Uso

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

## Ver estado de un jail en particular (e.g: sshd)
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

## Ver todas las direcciones banneadas en todas las jaulas
```
sudo fail2ban-client banned
```
Ejemplo de la salida (es una sola línea):
```
[{'sshd': ['1.194.218.133', '1.214.197.163', '1.238.106.229', '1.55.33.86', '1.71.9.130', '1.82.220.20', '1.9.107.43']}, {'apache-auth': []}, {'apache-badbots': []}, {'apache-noscript': []}, {'apache-overflows': []}, {'apache-nohome': []}, {'apache-botsearch': []}, {'apache-shellshock': []}, {'wordpress': []}]
```

## Bannear una IP manualmente para una jaula
```
sudo fail2ban-client set sshd banip 1.2.3.4
```
Ejemplo de la salida (la cantidad de IPs nuevas banneadas):
```
1
```

## Desbannear una IP manualmente para una jaula
```
sudo fail2ban-client set sshd unbanip 1.2.3.4
```
Ejemplo de la salida (la cantidad de IPs desbanneadas):
```
1
```

## Ver todos los comandos disponibles
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
                                             option '--restart' activates
                                             completely restarting of affected
                                             jails, thereby can unban IP
                                             addresses (if option '--unban'
                                             specified)
    reload [--restart] [--unban] [--if-exists] <JAIL>
                                             reloads the jail <JAIL>, or
                                             restarts it (if option '--restart'
                                             specified)
    stop                                     stops all jails and terminate the
                                             server
    unban --all                              unbans all IP addresses (in all
                                             jails and database)
    unban <IP> ... <IP>                      unbans <IP> (in all jails and
                                             database)
    banned                                   return jails with banned IPs as
                                             dictionary
    banned <IP> ... <IP>]                    return list(s) of jails where
                                             given IP(s) are banned
    status                                   gets the current status of the
                                             server
    ping                                     tests if the server is alive
    echo                                     for internal usage, returns back
                                             and outputs a given string
    help                                     return this output
    version                                  return the server version

                                             LOGGING
    set loglevel <LEVEL>                     sets logging level to <LEVEL>.
                                             Levels: CRITICAL, ERROR, WARNING,
                                             NOTICE, INFO, DEBUG, TRACEDEBUG,
                                             HEAVYDEBUG or corresponding
                                             numeric value (50-5)
    get loglevel                             gets the logging level
    set logtarget <TARGET>                   sets logging target to <TARGET>.
                                             Can be STDOUT, STDERR, SYSLOG,
                                             SYSTEMD-JOURNAL or a file
    get logtarget                            gets logging target
    set syslogsocket auto|<SOCKET>           sets the syslog socket path to
                                             auto or <SOCKET>. Only used if
                                             logtarget is SYSLOG
    get syslogsocket                         gets syslog socket path
    flushlogs                                flushes the logtarget if a file
                                             and reopens it. For log rotation.

                                             DATABASE
    set dbfile <FILE>                        set the location of fail2ban
                                             persistent datastore. Set to
                                             "None" to disable
    get dbfile                               get the location of fail2ban
                                             persistent datastore
    set dbmaxmatches <INT>                   sets the max number of matches
                                             stored in database per ticket
    get dbmaxmatches                         gets the max number of matches
                                             stored in database per ticket
    set dbpurgeage <SECONDS>                 sets the max age in <SECONDS> that
                                             history of bans will be kept
    get dbpurgeage                           gets the max age in seconds that
                                             history of bans will be kept

                                             JAIL CONTROL
    add <JAIL> <BACKEND>                     creates <JAIL> using <BACKEND>
    start <JAIL>                             starts the jail <JAIL>
    stop <JAIL>                              stops the jail <JAIL>. The jail is
                                             removed
    status <JAIL> [FLAVOR]                   gets the current status of <JAIL>,
                                             with optional flavor or extended
                                             info

                                             JAIL CONFIGURATION
    set <JAIL> idle on|off                   sets the idle state of <JAIL>
    set <JAIL> ignoreself true|false         allows the ignoring of own IP
                                             addresses
    set <JAIL> addignoreip <IP>              adds <IP> to the ignore list of
                                             <JAIL>
    set <JAIL> delignoreip <IP>              removes <IP> from the ignore list
                                             of <JAIL>
    set <JAIL> ignorecommand <VALUE>         sets ignorecommand of <JAIL>
    set <JAIL> ignorecache <VALUE>           sets ignorecache of <JAIL>
    set <JAIL> addlogpath <FILE> ['tail']    adds <FILE> to the monitoring list
                                             of <JAIL>, optionally starting at
                                             the 'tail' of the file (default
                                             'head').
    set <JAIL> dellogpath <FILE>             removes <FILE> from the monitoring
                                             list of <JAIL>
    set <JAIL> logencoding <ENCODING>        sets the <ENCODING> of the log
                                             files for <JAIL>
    set <JAIL> addjournalmatch <MATCH>       adds <MATCH> to the journal filter
                                             of <JAIL>
    set <JAIL> deljournalmatch <MATCH>       removes <MATCH> from the journal
                                             filter of <JAIL>
    set <JAIL> addfailregex <REGEX>          adds the regular expression
                                             <REGEX> which must match failures
                                             for <JAIL>
    set <JAIL> delfailregex <INDEX>          removes the regular expression at
                                             <INDEX> for failregex
    set <JAIL> addignoreregex <REGEX>        adds the regular expression
                                             <REGEX> which should match pattern
                                             to exclude for <JAIL>
    set <JAIL> delignoreregex <INDEX>        removes the regular expression at
                                             <INDEX> for ignoreregex
    set <JAIL> findtime <TIME>               sets the number of seconds <TIME>
                                             for which the filter will look
                                             back for <JAIL>
    set <JAIL> bantime <TIME>                sets the number of seconds <TIME>
                                             a host will be banned for <JAIL>
    set <JAIL> datepattern <PATTERN>         sets the <PATTERN> used to match
                                             date/times for <JAIL>
    set <JAIL> usedns <VALUE>                sets the usedns mode for <JAIL>
    set <JAIL> attempt <IP> [<failure1> ... <failureN>]
                                             manually notify about <IP> failure
    set <JAIL> banip <IP> ... <IP>           manually Ban <IP> for <JAIL>
    set <JAIL> unbanip [--report-absent] <IP> ... <IP>
                                             manually Unban <IP> in <JAIL>
    set <JAIL> maxretry <RETRY>              sets the number of failures
                                             <RETRY> before banning the host
                                             for <JAIL>
    set <JAIL> maxmatches <INT>              sets the max number of matches
                                             stored in memory per ticket in
                                             <JAIL>
    set <JAIL> maxlines <LINES>              sets the number of <LINES> to
                                             buffer for regex search for <JAIL>
    set <JAIL> addaction <ACT>[ <PYTHONFILE> <JSONKWARGS>]
                                             adds a new action named <ACT> for
                                             <JAIL>. Optionally for a Python
                                             based action, a <PYTHONFILE> and
                                             <JSONKWARGS> can be specified,
                                             else will be a Command Action
    set <JAIL> delaction <ACT>               removes the action <ACT> from
                                             <JAIL>

                                             COMMAND ACTION CONFIGURATION
    set <JAIL> action <ACT> actionstart <CMD>
                                             sets the start command <CMD> of
                                             the action <ACT> for <JAIL>
    set <JAIL> action <ACT> actionstop <CMD> sets the stop command <CMD> of the
                                             action <ACT> for <JAIL>
    set <JAIL> action <ACT> actioncheck <CMD>
                                             sets the check command <CMD> of
                                             the action <ACT> for <JAIL>
    set <JAIL> action <ACT> actionban <CMD>  sets the ban command <CMD> of the
                                             action <ACT> for <JAIL>
    set <JAIL> action <ACT> actionunban <CMD>
                                             sets the unban command <CMD> of
                                             the action <ACT> for <JAIL>
    set <JAIL> action <ACT> timeout <TIMEOUT>
                                             sets <TIMEOUT> as the command
                                             timeout in seconds for the action
                                             <ACT> for <JAIL>

                                             GENERAL ACTION CONFIGURATION
    set <JAIL> action <ACT> <PROPERTY> <VALUE>
                                             sets the <VALUE> of <PROPERTY> for
                                             the action <ACT> for <JAIL>
    set <JAIL> action <ACT> <METHOD>[ <JSONKWARGS>]
                                             calls the <METHOD> with
                                             <JSONKWARGS> for the action <ACT>
                                             for <JAIL>

                                             JAIL INFORMATION
    get <JAIL> banned                        return banned IPs of <JAIL>
    get <JAIL> banned <IP> ... <IP>]         return 1 if IP is banned in <JAIL>
                                             otherwise 0, or a list of 1/0 for
                                             multiple IPs
    get <JAIL> logpath                       gets the list of the monitored
                                             files for <JAIL>
    get <JAIL> logencoding                   gets the encoding of the log files
                                             for <JAIL>
    get <JAIL> journalmatch                  gets the journal filter match for
                                             <JAIL>
    get <JAIL> ignoreself                    gets the current value of the
                                             ignoring the own IP addresses
    get <JAIL> ignoreip                      gets the list of ignored IP
                                             addresses for <JAIL>
    get <JAIL> ignorecommand                 gets ignorecommand of <JAIL>
    get <JAIL> failregex                     gets the list of regular
                                             expressions which matches the
                                             failures for <JAIL>
    get <JAIL> ignoreregex                   gets the list of regular
                                             expressions which matches patterns
                                             to ignore for <JAIL>
    get <JAIL> findtime                      gets the time for which the filter
                                             will look back for failures for
                                             <JAIL>
    get <JAIL> bantime                       gets the time a host is banned for
                                             <JAIL>
    get <JAIL> datepattern                   gets the pattern used to match
                                             date/times for <JAIL>
    get <JAIL> usedns                        gets the usedns setting for <JAIL>
    get <JAIL> banip [<SEP>|--with-time]     gets the list of of banned IP
                                             addresses for <JAIL>. Optionally
                                             the separator character ('<SEP>',
                                             default is space) or the option '
                                             --with-time' (printing the times
                                             of ban) may be specified. The IPs
                                             are ordered by end of ban.
    get <JAIL> maxretry                      gets the number of failures
                                             allowed for <JAIL>
    get <JAIL> maxmatches                    gets the max number of matches
                                             stored in memory per ticket in
                                             <JAIL>
    get <JAIL> maxlines                      gets the number of lines to buffer
                                             for <JAIL>
    get <JAIL> actions                       gets a list of actions for <JAIL>

                                             COMMAND ACTION INFORMATION
    get <JAIL> action <ACT> actionstart      gets the start command for the
                                             action <ACT> for <JAIL>
    get <JAIL> action <ACT> actionstop       gets the stop command for the
                                             action <ACT> for <JAIL>
    get <JAIL> action <ACT> actioncheck      gets the check command for the
                                             action <ACT> for <JAIL>
    get <JAIL> action <ACT> actionban        gets the ban command for the
                                             action <ACT> for <JAIL>
    get <JAIL> action <ACT> actionunban      gets the unban command for the
                                             action <ACT> for <JAIL>
    get <JAIL> action <ACT> timeout          gets the command timeout in
                                             seconds for the action <ACT> for
                                             <JAIL>

                                             GENERAL ACTION INFORMATION
    get <JAIL> actionproperties <ACT>        gets a list of properties for the
                                             action <ACT> for <JAIL>
    get <JAIL> actionmethods <ACT>           gets a list of methods for the
                                             action <ACT> for <JAIL>
    get <JAIL> action <ACT> <PROPERTY>       gets the value of <PROPERTY> for
                                             the action <ACT> for <JAIL>

Report bugs to https://github.com/fail2ban/fail2ban/issues
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
