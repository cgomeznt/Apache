Instalar la ultima version de Apache
====================================

Pre requisitos
+++++++++++++++++++

Apache httpd usa libtool y autoconf para crear un entorno de compilación que se parece a muchos otros proyectos de código abierto.

Se deben atender primero los requisitos indicados por Apache: * apr * apr-util * pcre * pcre2

Tenemos dos (2) formas de instalar los pre requisitos, una por instalacion desde los repositorios y otra descargando los paquetes y compilando.

Instalando los pre requisitos desde los repositorios::

	dnf install wget gcc expat-devel.x86_64

	dnf install libapreq2-devel.x86_64 libapreq2-libs.x86_64
	
	dnf install pcre-devel.i686 pcre-devel.x86_64
	
	dnf install redhat-rpm-config


Instalando los pre requisitos desde los funentes descargandos y compilando:

Descarga apr1.7.4 y apr-util1.6.3 desde http://apr.apache.org/.

Compilar e Instalar apr::

	./configure --prefix=/opt/apache2/apr
	make
	make install
	
Si presenta este error::

	rm: cannot remove 'libtoolT': No such file or directory

Editar el configure, busque esta linea::

	$RM "$cfgfile"
	
Y le agrega el "-f"::

	$RM -f "$cfgfile"

	
Compilar e Instalar apr-util::

	./configure --prefix=/opt/apache2/apr-util --with-apr=/opt/apache2/apr
	make
	make install

Si apr-util te genera un error de::

	/bin/sh /data/abc/installed/httpd-2.4.56/srclib/apr/libtool --silent --mode=compile gcc -g -O2 -pthread   -DHAVE_CONFIG_H  -DLINUX -D_REENTRANT -D_GNU_SOURCE   -I/data/abc/installed/httpd-2.4.38/srclib/apr-util/include -I/data/abc/installed/httpd-2.4.38/srclib/apr-util/include/private  -I/data/abc/installed/httpd-2.4.38/srclib/apr/include    -o xml/apr_xml.lo -c xml/apr_xml.c && touch xml/apr_xml.lo
	xml/apr_xml.c:35:19: fatal error: expat.h: No such file or directory
	 #include <expat.h>

Esto es para el error de expat. Descargar expat-2.2.9.tar.bz2 from https://libexpat.github.io/.

Extraer expat, compilar e instalar::

	tar xvjf expat-2.2.9.tar.bz2
	cd expat-2.2.9
	./configure --prefix=/path-to-expat-installation-dir
	make
	make install

Y proceder nuevamente con la instalacion, pero indicandole ahora en donde esta el expat::

	./configure --prefix=/opt/apache2/apr-util --with-apr=/opt/apache2/apr --with-expat=/opt/apache2/expat

Instalamos pcre::

	dnf install pcre-devel.i686 pcre.i686 pcre2.i686

Link oficial utilizado::

	https://httpd.apache.org/docs/2.4/install.html
	
Copiar el enlace del ultimo apache estable https://httpd.apache.org/download.cgi

Lo descargamos y lo descomprimimos::

	wget https://dlcdn.apache.org/httpd/httpd-2.4.58.tar.gz
	tar -xvzf httpd-2.4.58.tar.gz
	cd httpd-2.4.58

Compilamos e instalamos el Apache::

	export CC="gcc"
	export CFLAGS="-O2"

	./configure --prefix=/opt/apache2/2.4.28
	
	make
	
	make install
	
Esta es avanzada pero no la he certificado::

	./configure --with-apr=/opt/apache2/apr  \
	--with-apr-util=/opt/apache2/apr-util \
	--prefix=/opt/apache2/2.4.56 \
	--disable-authnz-ldap \
	--disable-authnz-fcgi \
	--disable-isapi \
	--disable-socache-redis \
	--disable-bucketeer \
	--disable-example-hooks \
	--disable-case-filter \
	--disable-case-filter-in \
	--disable-example-ipc \
	--disable-enable-ldap \
	--disable-lua \
	--disable-luajit \
	--disable-ident \
	--disable-usertrack  \
	--disable-proxy-hcheck \
	--disable-ssl-staticlib-deps \
	--disable-optional-hook-export \
	--disable-optional-hook-import \
	--disable-optional-fn-import \
	--disable-optional-fn-export \
	--enable-mods-shared='authn-file authn-core authz-host authz-user authz-core access-compat auth-basic allowmethods socache-shmcb filter deflate mime log-config expires headers unique-id setenvif proxy proxy-connect proxy-http proxy-balancer session ssl lbmethod-byrequests unixd dir rewrite' --enable-mpms-shared=all
	


Iniciarmos el apache::

	/opt/apache2/2.4.56/bin/apachectl -k start

Declaracion de la variable que debe indicar en donde estan las Librerías (Aunque en Rock Linux 9 con Apache 2.4.28 dio error)::

	export LD_LIBRARY_PATH=/opt/apache2/apr/lib:/opt/apache2/apr-util/lib:/usr/lib:/usr/local/lib  # (para levantarlo con root)
	
	
Listo hasta aquí, si abre un navegador vera el mensaje::

	http://IP_SERVER
	
	It works!
	

	 

	
	
	
	
export http_proxy=http://10.132.0.10:8080 \
export https_proxy=http://10.132.0.10:8080 \
export fttp_proxy=http://10.132.0.10:8080