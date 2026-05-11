# Imapsync

[Imapsync](https://imapsync.lamiral.info/) no se distribuye más como ejecutable
aunque los fuentes siguen manteniéndose [en github
](https://github.com/imapsync/imapsync), y hay [algunas instrucciones para
instalar desde los mismos](https://tecadmin.net/use-imapsync-on-ubuntu/) (que 
yo no conseguí hacer funcionar), el paquete instalable [se vende por €72 (o €144
con soporte)](https://imapsync.lamiral.info/#buy_all). Lo que sí ofrece el autor 
es un [servicio gratuito de sincronización via
web](https://imapsync.lamiral.info/X/).

El autor de imapsync también mantiene [un listado de herramientas similares a
imapsync](https://imapsync.lamiral.info/S/external.shtml).

Acabo de ver [_nuevas_ instrucciones para instalar imapsync en Debian y
Ubuntu](https://help.clouding.io/hc/es/articles/4408860226834-C%C3%B3mo-instalar-y-usar-imapsync-Debian-Ubuntu)
que voy a tratar de hacer funcionar en Debian 12 (bookworm).

## Instalación

* Instalar pre-requisitos (paquetes `.deb` y paquetes directamente bajados de
[CPAN](https://www.cpan.org/))
```bash
sudo apt install libtest-simple-perl libtest-requires-perl \
  libtest-mock-guard-perl libtest-fatal-perl libpar-packer-perl \
  libnet-ssleay-perl libio-compress-perl libdigest-hmac-perl \
  libcrypt-ssleay-perl libssl-dev libauthen-ntlm-perl libclass-load-perl \
  libcgi-pm-perl libcrypt-openssl-rsa-perl libdata-uniqid-perl \
  libencode-imaputf7-perl libfile-copy-recursive-perl \
  libfile-tail-perl libio-socket-inet6-perl libio-socket-ssl-perl \
  libio-tee-perl libhtml-parser-perl libjson-webtoken-perl \
  libmail-imapclient-perl libparse-recdescent-perl libmodule-scandeps-perl \
  libreadonly-perl libregexp-common-perl libsys-meminfo-perl \
  libterm-readkey-perl libtest-mockobject-perl libtest-pod-perl \
  libunicode-string-perl liburi-perl libwww-perl libtest-nowarnings-perl \
  libtest-deep-perl libtest-warn-perl cpanminus make git rcs gcc apt-file

sudo apt-file update

sudo cpanm Crypt::OpenSSL::RSA Proc::ProcessTable Crypt::OpenSSL::Random --force
sudo cpanm Mail::IMAPClient JSON::WebToken Test::MockObject Dist::CheckConflicts
sudo cpanm Unicode::String Data::Uniqid --force
sudo cpanm Mail::IMAPClient Crypt::OpenSSL::PKCS12 IO::Socket::SSL \
  JSON::WebToken JSON Crypt::OpenSSL::RSA LWP HTML::Entities Encode::Byte
```

* Clonar el repositorio oficial de github
```bash
git clone https://github.com/imapsync/imapsync.git
```
* Instalar en `/usr/local`
```bash
export PREFIX="/usr/local"

sudo PREFIX=${PREFIX} make install
```
* Verificar el funcionamiento con los tests incorporados en imapsync
```bash
imapsync --testslive
```
Esta verificación es equivalente a ejecutar el script
[imapsync_example.sh](https://github.com/imapsync/imapsync/blob/master/examples/imapsync_example.sh)
en la carpeta
[examples](https://github.com/imapsync/imapsync/tree/master/examples):
```bash
sh examples/imapsync_example.sh
```

## Uso

Por ahora dejo el link al [tutorial de imapsync para
unix](https://imapsync.lamiral.info/doc/TUTORIAL_Unix.html) y las [mejores
prácticas](https://imapsync.lamiral.info/doc/GOOD_PRACTICES.html).



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
