Virtual Host
=============

Virtual host es para correr mas de un dominio en una IP address. Es especialmente usuado para el publico que necesita correr varios sites en un virtual private server. El site mostrara diferente informacion a los visitadores, dependiendo de que site ingrese. No hay numero de virtual hosts.
Creamos un directorio para almacenar el contenido de la pagina que servira el virtual host.::

Recordamos los comando de administración de Apache2::

	systemctl enable apache2
	
	systemctl disable apache2
	
	systemctl status apache2

	systemctl stop apache2

	systemctl start apache2

	systemctl restart apache2

	systemctl reload apache2

Configurando el Virtual Hosts en Debian
++++++++++++++++++++++++++++++++++++++++

Recordemos que el default es **/var/www/html/**

Vamos a crear un directorio para nuestro Virtual Host::

	# mkdir /var/www/html/public_html

Debemos otorgar los permisos requeridos.::

	# chown -R www-data:www-data /var/www/html/public_html/

	# chmod -R 755 /var/www/html/public_html/
	# ls -l
	total 4
	drwxr-xr-x. 2 www-data www-data 4096 ago 28 22:45 public_html

Recuerda configurar los DNS para este site corresponda con la IP, para probar lo puedes crear en el /etc/hosts.::

	# vi /etc/hosts
	192.168.1.10     www.public.com

	# ping -c 4 www.public.com
	PING www.public.com (192.168.1.4) 56(84) bytes of data.
	64 bytes from www.public.com (192.168.1.10): icmp_seq=1 ttl=64 time=0.387 ms
	64 bytes from www.public.com (192.168.1.10): icmp_seq=2 ttl=64 time=0.349 ms
	64 bytes from www.public.com (192.168.1.10): icmp_seq=3 ttl=64 time=0.310 ms
	64 bytes from www.public.com (192.168.1.10): icmp_seq=4 ttl=64 time=0.239 ms

	--- www.public.com ping statistics ---
	4 packets transmitted, 4 received, 0% packet loss, time 2997ms
	rtt min/avg/max/mdev = 0.239/0.321/0.387/0.056 ms


Creamos una pagina de prueba, que es index.html::

	# vi /var/www/html/public_html/index.html

	<html>
	  <head>
		<title>www.public.com</title>
	  </head>
	  <body>
		<h1>Felicitaciones, se creo el Virtual Host de public en public_html</h1>
	  </body>
	</html>



Crear Nuevos Archivos Virtual Host
--------------------------------


Los archivos Virtual Host son archivos que contienen información y configuración específica para el dominio y que le indican al servidor Apache como responden a las peticiones de varios dominios.

Apache incluye un archivo Virtual Host por defecto denominado 000-default.conf que podemos usar para saltarnos al punto. Realizaremos una copia para trabajar sobre ella y crear nuestro Virtual Host para cada dominio.

Iniciaremos con un dominio, configuralo, copialo para el segundo dominio, y después realiza los ajustes necesarios.

En Debian los archivos de configuracion deben estar en "/etc/apache2"y para el Virtual Host en sites-avaible y un enlace en site-enabled::

	# vi /etc/apache2/sites-available/public_html.conf

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
			 #ErrorLog ${APACHE_LOG_DIR}/public_html_error.log
			 #CustomLog ${APACHE_LOG_DIR}/public_html_requests.log common
			 ErrorLog public_html_error.log
			 CustomLog public_html_requests.log common
		</VirtualHost>

Cuando creamos el Virtual Host en sites-avaible, aun no lo estamos habilitando para habilitarlo podemos usar una de estas dos (2) tecnicas.

**Tecnica 1**, 

lo habilitamos::

	a2ensite public_html.conf
	
Si lo que remos deshabilitar::

	a2dissite public_html.conf
	
Verificamos que no exista errores en la configuracion::

	apachectl configtest
	Syntax OK

**Tecnica 2**, 

lo habilitamos::

	cd /etc/apache2/sites-enabled
	
	ln -s ../public_html.conf .
	
Si lo que remos deshabilitar::

	rm /etc/apache2/sites-enabled/public_html.conf
	
Verificamos que no exista errores en la configuracion::

	apachectl configtest
	Syntax OK
	
Reiniciamos el apache.::

	# systemctl reload apache2
	
Familiarizarse con los archivos y directorios importantes de Apache
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Ahora que sabe cómo administrar el propio servicio de Apache, debe dedicar unos minutos a familiarizarse con algunos directorios y archivos importantes.

Configuración del servidor
------------------------------

**/etc/apache2:** El directorio de configuración de Apache. Todos los archivos de configuración de Apache residen aquí.

**/etc/apache2/apache2.conf:** El archivo de configuración principal de Apache. Esto se puede modificar para realizar cambios en la configuración global de Apache. Este archivo es responsable de cargar muchos de los otros archivos en el directorio de configuración.

**/etc/apache2/ports.conf:** este archivo especifica los puertos en los que escuchará Apache. De forma predeterminada, Apache escucha en el puerto 80 y, además, escucha en el puerto 443 cuando se habilita un módulo que proporciona capacidades SSL.

**/etc/apache2/sites-disponible/:** el directorio donde se pueden almacenar los hosts virtuales por sitio. Apache no usará los archivos de configuración que se encuentran en este directorio a menos que estén vinculados al directorio habilitado para sitios. Por lo general, toda la configuración del bloque del servidor se realiza en este directorio y luego se habilita vinculando al otro directorio con el comando a2ensite.

**/etc/apache2/sites-enabled/:** el directorio donde se almacenan los hosts virtuales habilitados por sitio. Por lo general, estos se crean vinculando a los archivos de configuración que se encuentran en el directorio de sitios disponibles con el a2ensite. Apache lee los archivos de configuración y los enlaces que se encuentran en este directorio cuando se inicia o recarga para compilar una configuración completa.

**/etc/apache2/conf-disponible/, /etc/apache2/conf-enabled/:** estos directorios tienen la misma relación que los directorios de sitios disponibles y sitios habilitados, pero se usan para almacenar fragmentos de configuración que no pertenecen a un anfitrión virtual. Los archivos en el directorio disponible conf se pueden habilitar con el comando a2enconf y deshabilitar con el comando a2disconf.

**/etc/apache2/mods-disponible/, /etc/apache2/mods-enabled/:** Estos directorios contienen los módulos disponibles y habilitados, respectivamente. Los archivos que terminan en .load contienen fragmentos para cargar módulos específicos, mientras que los archivos que terminan en .conf contienen la configuración de esos módulos. Los módulos se pueden habilitar y deshabilitar usando el comando a2enmod y a2dismod.

LOG del servidor
--------------------

**/var/log/apache2/access.log:** De forma predeterminada, cada solicitud a su servidor web se registra en este archivo de registro a menos que Apache esté configurado para hacer lo contrario.

**/var/log/apache2/error.log:** De forma predeterminada, todos los errores se registran en este archivo. La directiva LogLevel en la configuración de Apache especifica cuánto detalle contendrán los registros de errores.

Probamos el nuevo virtual host. En un navegador::
----------------------------------------------------

http://www.public.com



Configuramos Global ServerName Directive
--------------------------------------------

Para resolver un **AH00558: Could not reliably determine the server's fully qualified domain name**, deberá agregar una directiva ServerName a su configuración de Apache. Apache usa la directiva ServerName para asignar solicitudes HTTP entrantes a una dirección IP o nombre de host DNS usando directivas VirtualHost para manejar solicitudes para múltiples sitios usando un solo servidor::

	vi /etc/apache2/apache2.conf

	. . .
	# Include the virtual host configurations:
	IncludeOptional sites-enabled/*.conf

	# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
	ServerName 127.0.0.1


