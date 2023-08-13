Usar Apache como Reverse Proxy con mod_proxy
=============================================

Esto se probo con Apache 2.4.56 y Debian 11::


	# apache2 -V
	[Sat Aug 12 22:21:53.042176 2023] [core:warn] [pid 1555] AH00111: Config variable ${APACHE_RUN_DIR} is not defined
	apache2: Syntax error on line 80 of /etc/apache2/apache2.conf: DefaultRuntimeDir must be a valid directory, absolute or relative to ServerRoot
	Server version: Apache/2.4.56 (Debian)
	Server built:   2023-04-02T03:06:01
	Server's Module Magic Number: 20120211:126
	Server loaded:  APR 1.7.0, APR-UTIL 1.6.1, PCRE 8.39 2016-06-14
	Compiled using: APR 1.7.0, APR-UTIL 1.6.1, PCRE 8.39 2016-06-14
	Architecture:   64-bit
	Server MPM:
	Server compiled with....
	 -D APR_HAS_SENDFILE
	 -D APR_HAS_MMAP
	 -D APR_HAVE_IPV6 (IPv4-mapped addresses enabled)
	 -D APR_USE_PROC_PTHREAD_SERIALIZE
	 -D APR_USE_PTHREAD_SERIALIZE
	 -D SINGLE_LISTEN_UNSERIALIZED_ACCEPT
	 -D APR_HAS_OTHER_CHILD
	 -D AP_HAVE_RELIABLE_PIPED_LOGS
	 -D DYNAMIC_MODULE_LIMIT=256
	 -D HTTPD_ROOT="/etc/apache2"
	 -D SUEXEC_BIN="/usr/lib/apache2/suexec"
	 -D DEFAULT_PIDLOG="/var/run/apache2.pid"
	 -D DEFAULT_SCOREBOARD="logs/apache_runtime_status"
	 -D DEFAULT_ERRORLOG="logs/error_log"
	 -D AP_TYPES_CONFIG_FILE="mime.types"
	 -D SERVER_CONFIG_FILE="apache2.conf"


Un proxy inverso es un tipo de servidor proxy que toma solicitudes HTTP(S) y las distribuye de forma transparente a uno o más servidores backend. Los proxies inversos son útiles porque muchas aplicaciones web modernas procesan las solicitudes HTTP entrantes utilizando servidores de aplicaciones back-end a los que los usuarios no deben acceder directamente y, a menudo, solo admiten funciones HTTP rudimentarias.

Puede utilizar un proxy inverso para evitar que se acceda directamente a estos servidores de aplicaciones subyacentes. También se pueden usar para distribuir la carga de las solicitudes entrantes a varios servidores de aplicaciones diferentes, lo que aumenta el rendimiento a escala y brinda protección contra fallas. Pueden llenar los vacíos con funciones que los servidores de aplicaciones no ofrecen, como el almacenamiento en caché, la compresión o el cifrado SSL también.

En este tutorial, configurará Apache como un proxy inverso básico utilizando la extensión mod_proxy para redirigir las conexiones entrantes a uno o varios servidores back-end que se ejecutan en la misma red. Este tutorial utiliza un backend simple escrito con el marco web with Flask, pero puede usar cualquier servidor backend que prefiera.


Habilitación de los módulos de Apache necesarios
-------------------------------------------------
Apache tiene muchos módulos incluidos que están disponibles pero no habilitados en una instalación nueva. Primero, necesitaremos habilitar los que usaremos en este tutorial.

Los módulos que necesitamos son el propio mod_proxy y varios de sus módulos adicionales, que amplían su funcionalidad para admitir diferentes protocolos de red. En concreto, utilizaremos:

mod_proxy, el módulo proxy principal Módulo de Apache para redirigir conexiones; permite que Apache actúe como puerta de enlace a los servidores de aplicaciones subyacentes.

mod_proxy_http, que agrega soporte para conexiones HTTP de proxy.

mod_proxy_balancer y mod_lbmethod_byrequests, que agregan funciones de balanceo de carga para múltiples servidores backend.

Para habilitar estos cuatro módulos, ejecute los siguientes comandos en sucesión::

	a2enmod ssl
	sudo a2enmod proxy
	sudo a2enmod proxy_http
	sudo a2enmod proxy_balancer
	sudo a2enmod lbmethod_byrequests
	
Reiniciamos Apache::

	systemctl restart apache2

Creamos los Vhost que se requieran::

	vi /etc/apache2/sites-available/www.public.com.conf

	<VirtualHost *:80>
			ServerAdmin webmaster@example.com
			DocumentRoot /var/www/html/#www.public.com
			ServerName www.public.com
			ServerAlias public.com
			ErrorLog /var/log/apache2/public_html_error.log
			CustomLog /var/log/apache2/public_html_requests.log common
			ProxyRequests Off
			ProxyPreserveHost On
			SSLProxyEngine On
	  
			# Cuando se coloquen la URL http://www.public.com se realizara el proxy inverso hacia http://www.public.com, que se encuentra en otro servidor
			ProxyPass  / http://www.public.com
			ProxyPassReverse / https://www.public.com
			<IfModule security2_module>
				SecRuleEngine DetectionOnly
            </IfModule>
	</VirtualHost>
	
Creamos otro en el cual evitamos el error de SSL Handshake with remote server ::

vi /etc/apache2/sites-available/sencamer.conf

	<VirtualHost *:80>
			ServerAdmin webmaster@ejemplo.com
			DocumentRoot /var/www/html/
			ServerName sencamer.com
			ProxyRequests Off
			ProxyPreserveHost On
			SSLProxyEngine on
			SSLProxyVerify none
			##### Error during SSL Handshake with remote server####################
			SSLProxyEngine on
			SSLProxyVerify none
			SSLProxyCheckPeerCN off
			SSLProxyCheckPeerName off
			SSLProxyCheckPeerExpire off
			############################################
			# Cuando en la URL coloquen http://ejemplo.com se cargara el contenido de /var/www/html/ejemplo.com
			# Pero cuando coloquen la URL http://ejemplo.com/ver se realizara el proxy inverso hacia https://corporativo.com
			ProxyPass  / https://www.sencamer.gob.ve/
			ProxyPassReverse / https://www.sencamer.gob.ve/
			ErrorLog /var/log/apache2/sencamer-error_log
			CustomLog /var/log/apache2/sencamer-access_log common
			<IfModule security2_module>
				SecRuleEngine DetectionOnly
            </IfModule>
	</VirtualHost>


	
Este otro Virtual Host - Redirige las peticiones del puerto 80 http al 443 para usar SSL::

#<VirtualHost *:80>
#        ServerName suafweb
#        Redirect "/" 
#</VirtualHost>


<VirtualHost *:4848>
        # DocumentRoot /var/www/html/101/
        ServerName srv-vwebsuaf.dominio.local
        ServerAlias suafweb.dominio.local
        # Escritura de los logs de Apache
        ErrorLog logs/suafweb_admin_error.log
        CustomLog logs/suafweb_admin_requests.log common
        # Front Side - Peticiones son recibidas usando TLS v1.2 Con certificados de CA y del WebSite
        SSLEngine On
        SSLProtocol TLSv1.2
        SSLCACertificateFile /etc/httpd/conf.d/certs/srv-vwebsuaf.dominio.local.pem
        SSLCertificateFile /etc/httpd/conf.d/certs/srv-vwebsuaf.dominio.local.pem
        SSLCertificateKeyFile /etc/httpd/conf.d/certs/srv-vwebsuaf.dominio.local.pem
        # Niveles de seguridad del Apache
        # ServerSignature Off
        Options -Indexes
        ProxyRequests Off
        ProxyPreserveHost Off
        # Detecta el tipo de error para captura el ID y mostrarlo por el navegador, para tener mas control de los falsos positivos
        <LocationMatch "^/+$">
               Options -Indexes
               ErrorDocument 403 /403.php
        </LocationMatch>
        # Backend - Configuracion del backend
        # SSLProxyEngine On
        ProxyPass / http://localhost:4849/
        ProxyPassReverse / http://localhost:4849/
</VirtualHost>

