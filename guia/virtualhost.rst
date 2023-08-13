Virtual Host
=============

Virtual host es para correr mas de un dominio en una IP address. Es especialmente usuado para el publico que necesita correr varios sites en un virtual private server. El site mostrara diferente informacion a los visitadores, dependiendo de que site ingrese. No hay numero de virtual hosts.
Creamos un directorio para almacenar el contenido de la pagina que servira el virtual host.::

	# mkdir /var/www/html/public_html

Debemos otorgar los permisos requeridos.::

	# chown -R apache: /var/www/html/public_html/
	# ls -l
	total 4
	drwxr-xr-x. 2 apache apache 4096 ago 28 22:45 public_html

	# chmod -R 755 /var/www/html/public_html/
	# ls -l
	total 4
	drwxr-xr-x. 2 apache apache 4096 ago 28 22:45 public_html

Recuerda configurar los DNS para este site corresponda con la IP, para probar lo puedes crear en el /etc/hosts.::

	# vi /etc/hosts
	192.168.1.4     www.public.com

	# ping -c 4 www.public.com
	PING www.public.com (192.168.1.4) 56(84) bytes of data.
	64 bytes from www.public.com (192.168.1.4): icmp_seq=1 ttl=64 time=0.387 ms
	64 bytes from www.public.com (192.168.1.4): icmp_seq=2 ttl=64 time=0.349 ms
	64 bytes from www.public.com (192.168.1.4): icmp_seq=3 ttl=64 time=0.310 ms
	64 bytes from www.public.com (192.168.1.4): icmp_seq=4 ttl=64 time=0.239 ms

	--- www.public.com ping statistics ---
	4 packets transmitted, 4 received, 0% packet loss, time 2997ms
	rtt min/avg/max/mdev = 0.239/0.321/0.387/0.056 ms


Creamos una pagina de prueba.::

	# vi /var/www/html/public_html/index.html

	<html>
	  <head>
		<title>www.public.com</title>
	  </head>
	  <body>
		<h1>Felicitaciones, se creo el Virtual Host de public en public_html</h1>
	  </body>
	</html>

Pero antes debemos activar en NameVirtualHost *:80 en el http.conf.::

	# vi /etc/httpd/confd/http.conf
	NameVirtualHost *:80

Crear Nuevos Archivos Virtual Host
Los archivos Virtual Host son archivos que contienen información y configuración específica para el dominio y que le indican al servidor Apache como responden a las peticiones de varios dominios.

Apache incluye un archivo Virtual Host por defecto denominado 000-default.conf que podemos usar para saltarnos al punto. Realizaremos una copia para trabajar sobre ella y crear nuestro Virtual Host para cada dominio.

Iniciaremos con un dominio, configuralo, copialo para el segundo dominio, y después realiza los ajustes necesarios. La configuración por defecto de Ubuntu requiere que cada archivo de configuración de Virtual Host termine en .conf

En Centos los archivos de configuracion deben estar en "/etc/httpd/conf.d", esa directiva se esta en el archivo de configuracion /etc/httpd/conf/httpd.conf en Include conf.d/*.conf

Creamos un archivo por cada virtual host.::

	# vi /etc/httpd/conf.d/public_html.conf
	#
	# NOTE: NameVirtualHost cannot be used without a port specifier
	# (e.g. :80) if mod_ssl is being used, due to the nature of the
	# SSL protocol.
	#    

	#    
	# VirtualHost example:
	# Almost any Apache directive may go into a VirtualHost container.
	# The first VirtualHost section is used for requests without a known
	# server name.
	# 
	<VirtualHost *:80>
		 ServerAdmin webmaster@example.com
		 DocumentRoot /var/www/html/public_html
		 ServerName www.public.com
		 ServerAlias public.com
		 ErrorLog logs/public_html_error.log
		 CustomLog logs/public_html_requests.log common
	</VirtualHost>

Probamos la configuración::

	# apachectl configtest
	Syntax OK

Reiniciamos el apache.::

	# service httpd restart

Probamos el nuevo virtual host.::

	# curl www.public.com
	<html>
	  <head>
		<title>www.public.com</title>
	  </head>
	  <body>
		<h1>Felicitaciones, se creo el Virtual Host de prueba_html</h1>
	  </body>
	</html>




