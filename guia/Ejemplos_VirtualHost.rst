Ejemplos de Virtual Host para Apache
======================================

Hay un ejemplo SSL de un ambiente de produccion, no es el mejor pero es util. Tenemos otros ejemplos de Virtual Host en donde tiene varias opciones de interés .

Este es un buen ejemplo de un archivo SSL::

	cat /etc/httpd/conf.d/001_ssl.conf
	LoadModule ssl_module modules/mod_ssl.so

	SSLPassPhraseDialog  builtin

	SSLSessionCache         shmcb:/var/cache/mod_ssl/scache(512000)
	SSLSessionCacheTimeout  300

	SSLMutex default

	SSLRandomSeed startup file:/dev/urandom  256
	SSLRandomSeed connect builtin

	SSLCryptoDevice builtin

	SSLCipherSuite ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5

Ejemplo de un Vhost que Redirige las peticiones del puerto 4848 http al 4849 para usar SSL en el mismo server, en este ejemplo es un Apache delante que redirecciona las peticiones a un Glassfish::

	cat /etc/httpd/conf.d/101_vhost_admin.conf
	# Virtual Host - Redirige las peticiones del puerto 4848 http al 4849 para usar SSL en el mismo server
	#        ServerName suafweb.credicard.com.ve
	#        Redirect "/" "https://suafweb.credicard.com.ve"
	#</VirtualHost>


	<VirtualHost *:4848>
			# DocumentRoot /var/www/html/101/
			ServerName srv-vwebsuaf.credicard.com.ve
			ServerAlias suafweb.credicard.com.ve
			# Escritura de los logs de Apache
			ErrorLog logs/suafweb_admin_error.log
			CustomLog logs/suafweb_admin_requests.log common
			# Front Side - Peticiones son recibidas usando TLS v1.2 Con certificados de CA y del WebSite
			SSLEngine On
			SSLProtocol TLSv1.2
			SSLCACertificateFile /etc/httpd/conf.d/certs/srv-vwebsuaf.credicard.com.ve.pem
			SSLCertificateFile /etc/httpd/conf.d/certs/srv-vwebsuaf.credicard.com.ve.pem
			SSLCertificateKeyFile /etc/httpd/conf.d/certs/srv-vwebsuaf.credicard.com.ve.pem
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

Ejemplo de un Vhost que Redirige las peticiones del puerto 15040 http al 15041 para usar SSL en el mismo server, en este ejemplo es un Apache delante que redirecciona las peticiones a un Glassfish::

	cat /etc/httpd/conf.d/102_vhost_controlFraude.conf
	# Virtual Host - Redirige las peticiones del puerto 15040 http al 15041 para usar SSL en el mismo server
	#<VirtualHost *:80>
	#        ServerName suafweb.credicard.com.ve
	#        Redirect "/" "https://suafweb.credicard.com.ve"
	#</VirtualHost>


	<VirtualHost *:15040>
			# DocumentRoot /var/www/html/101/
			ServerName srv-vwebsuaf.credicard.com.ve
			ServerAlias suafweb.credicard.com.ve
			# Escritura de los logs de Apache
			ErrorLog logs/suafweb_error.log
			CustomLog logs/suafweb_requests.log common
			# Front Side - Peticiones son recibidas usando TLS v1.2 Con certificados de CA y del WebSite
			SSLEngine On
			SSLProtocol TLSv1.2
			SSLCACertificateFile /etc/httpd/conf.d/certs/srv-vwebsuaf.credicard.com.ve.pem
			SSLCertificateFile /etc/httpd/conf.d/certs/srv-vwebsuaf.credicard.com.ve.pem
			SSLCertificateKeyFile /etc/httpd/conf.d/certs/srv-vwebsuaf.credicard.com.ve.pem
			# Niveles de seguridad del Apache
			# ServerSignature Off
			# Options -Indexes
			ProxyRequests Off
			ProxyPreserveHost Off
			# Detecta el tipo de error para captura el ID y mostrarlo por el navegador, para tener mas control de los falsos positivos
			<LocationMatch "^/+$">
				   Options -Indexes
				   ErrorDocument 403 /403.php
			</LocationMatch>
			# Backend - Configuracion del backend
			# SSLProxyEngine On
			ProxyPass / http://localhost:15041/
			ProxyPassReverse / http://localhost:15041/
	</VirtualHost>

Ejemplo de un Vhost que Redirige las peticiones del puerto 443 http al 15041 para usar SSL en el mismo server, en este ejemplo es un Apache delante que redirecciona las peticiones a un Glassfish::

	cat /etc/httpd/conf.d/103_vhost_controlFraude.conf
	# Virtual Host - Redirige las peticiones del puerto 443 http al 1504 para usar SSL en el mismo serve
	#<VirtualHost *:80>
	#        ServerName suafweb.credicard.com.ve
	#        Redirect "/" "https://suafweb.credicard.com.ve"
	#</VirtualHost>


	<VirtualHost *:443>
			# DocumentRoot /var/www/html/101/
			ServerName srv-vwebsuaf.credicard.com.ve
			ServerAlias suafweb.credicard.com.ve
			# Escritura de los logs de Apache
			ErrorLog logs/suafweb_error.log
			CustomLog logs/suafweb_requests.log common
			# Front Side - Peticiones son recibidas usando TLS v1.2 Con certificados de CA y del WebSite
			SSLEngine On
			SSLProtocol TLSv1.2
			SSLCACertificateFile /etc/httpd/conf.d/certs/srv-vwebsuaf.credicard.com.ve.pem
			SSLCertificateFile /etc/httpd/conf.d/certs/srv-vwebsuaf.credicard.com.ve.pem
			SSLCertificateKeyFile /etc/httpd/conf.d/certs/srv-vwebsuaf.credicard.com.ve.pem
			# Niveles de seguridad del Apache
			# ServerSignature Off
			# Options -Indexes
			ProxyRequests Off
			ProxyPreserveHost Off
			# Detecta el tipo de error para captura el ID y mostrarlo por el navegador, para tener mas control de los falsos positivos
			<LocationMatch "^/+$">
				   Options -Indexes
				   ErrorDocument 403 /403.php
			</LocationMatch>
			# Backend - Configuracion del backend
			# SSLProxyEngine On
			ProxyPass / http://localhost:15041/
			ProxyPassReverse / http://localhost:15041/
	</VirtualHost>


