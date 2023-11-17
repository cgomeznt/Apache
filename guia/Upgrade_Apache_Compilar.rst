Upgrade compilación
=====================

Como hacer un upgrade de un apache que fue compilado.

En este ejemplo vamos actualizar Apache 2.4.28 al 2.4.58

Link oficial utilizado::

	https://httpd.apache.org/docs/2.4/install.html
	
Copiar el enlace del ultimo apache estable https://httpd.apache.org/download.cgi

Lo descargamos y lo descomprimimos::

	wget https://dlcdn.apache.org/httpd/httpd-2.4.58.tar.gz
	tar -xvzf httpd-2.4.58.tar.gz
	cd httpd-2.4.58
	
Copiamos del apache que queremos hacer el updete el archivo "config.nice" en el directorio donde descargamos la nueva versión::

	cp /opt/apache2/2.4.28/build/config.nice /root/httpd-2.4.58/

Compilamos e instalamos el Apache::

	./config.nice
	
	make
	
	make install
	
	./config.nice


Reiniciamos el Apache::

	/opt/apache2/2.4.28/bin/apachectl -k stop
	/opt/apache2/2.4.28/bin/apachectl -k start

Consultamos la nueva versión, pero no olviden este detalle el directorio dice 2.4.28, pero sus binarios son del update::

	/opt/apache2/2.4.28/bin/httpd -v
	
	Server version: Apache/2.4.58 (Unix)
	Server built:   Nov 17 2023 13:24:13

