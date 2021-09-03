Compilar Instalar Apache 2.4
================================


Apache httpd usa libtool y autoconf para crear un entorno de compilación que se parece a muchos otros proyectos de código abierto

Se deben atender primero los requisitos indicados por Apache:
* apr
* apr-util
* pcre
* pcre2

Descarga apr1.7.0 y apr-util1.6.0 desde  http://apr.apache.org/.

Compilar e Instalar apr::

	./configure --prefix=/orasoft/product/apr
	make
	make install

Compilar e Instalar apr-util::

	./configure --prefix=/orasoft/product/apr-util --with-apr=/orasoft/product/apr
	make
	make install

Si apr-util te genera un error de::

	/bin/sh /data/abc/installed/httpd-2.4.38/srclib/apr/libtool --silent --mode=compile gcc -g -O2 -pthread   -DHAVE_CONFIG_H  -DLINUX -D_REENTRANT -D_GNU_SOURCE   -I/data/abc/installed/httpd-2.4.38/srclib/apr-util/include -I/data/abc/installed/httpd-2.4.38/srclib/apr-util/include/private  -I/data/abc/installed/httpd-2.4.38/srclib/apr/include    -o xml/apr_xml.lo -c xml/apr_xml.c && touch xml/apr_xml.lo
	xml/apr_xml.c:35:19: fatal error: expat.h: No such file or directory
	 #include <expat.h>
	 
Descargar expat-2.2.9.tar.bz2 from https://libexpat.github.io/.

Extraer expat, compilar e instalar::

	tar xvjf expat-2.2.9.tar.bz2
	cd expat-2.2.9 
	./configure --prefix=/path-to-expat-installation-dir
	make
	make install

Y proceder nuevamente con la instalacion pero indicandole ahora en donde esta el expat::
	
	./configure --prefix=/orasoft/product/apr-util --with-apr=/orasoft/product/apr --with-expat=/orasoft/product/expat

Instalamos pcre::

	yum install pcre-devel.i686 pcre.i686 pcre2.i686


	Compilamos e instalamos el Apache::

	export CC="gcc"
	export CFLAGS="-O2"
	./configure --with-apr=/orasoft/product/apr  \
	--with-apr-util=/orasoft/product/apr-util \
	--prefix=/orasoft/product/apache/2.4.43 \
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
	make
	make install


Declaracion de la variable que debe indicar en donde estan las Librerías::

	LD_LIBRARY_PATH=/orasoft/product/apr/lib:/orasoft/product/apr-util/lib:/usr/lib:/usr/local/lib (para levantarlo con root)


Instalamos y configuramos authbind::

  sudo rpm -Uvh https://github.com/cgomeznt/Apache/tree/master/tools/authbind-2.1.1-0.1.x86_64.rpm
  touch /etc/authbind/byport/80
  chmod 500 /etc/authbind/byport/80
  chown tomcat /etc/authbind/byport/80
  authbind -deep /orasoft/product/apache/2.4.43/bin/apachectl start
  

Creamos la plantilla de servicio en /etc/systemd/system/httpd.service::


	[Unit]
	Description=Apache - instance %i
	After=syslog.target network.target

	[Service]
	Type=forking

	User=weblogic
	Group=oinstall

	WorkingDirectory=/orasoft/product/apache/2.4.43/

	Environment="LD_LIBRARY_PATH=/orasoft/product/apr/lib"

	ExecStart=/bin/authbind -deep /orasoft/product/apache/2.4.43/bin/apachectl start
	ExecStop=/bin/authbind -deep /orasoft/product/apache/2.4.43/bin/apachectl stop

	RestartSec=10
	Restart=always

	[Install]
	WantedBy=multi-user.target

Recargamos el demonios SystemCtl, habilitamos e iniciamos el Apache::

	systemctl daemon-reload
	systemctl enable httpd.service
	systemctl start httpd.service