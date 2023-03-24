Instalar la ultima version de Apache
====================================

Pre requisitos
+++++++++++++++++++

Instalamos los pre requisitos::

	dnf install wget gcc expat-devel.x86_64

	dnf install libapreq2-devel.x86_64 libapreq2-libs.x86_64

Copiar el enlace del ultimo apache estable https://httpd.apache.org/download.cgi

Lo descargamos y lo descomprimimos::

	wget https://dlcdn.apache.org/httpd/httpd-2.4.56.tar.gz

	tar -xvzf httpd-2.4.56.tar.gz

	cd httpd-2.4.56

Esto no aplica, no esta certificado
############################################################
cd srclib/

Copiar el enlace del ultimo apr y de apr-util https://apr.apache.org/download.cgi

wget https://dlcdn.apache.org//apr/apr-1.7.2.tar.gz

wget https://dlcdn.apache.org//apr/apr-util-1.6.3.tar.gz

tar -xvzf apr-1.7.2.tar.gz

tar -xvzf apr-util-1.6.3.tar.gz

mv apr-1.7.2 apr

mv apr-util-1.6.3 apr-util
############################################################


Comenzamos a compilar::

	./configure --prefix=/usr/local/apache

	make

	make install
	
Iniciarmos el apache::

	/usr/local/apache2/bin/apachectl start


	 
