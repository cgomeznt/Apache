Configurar Redireccionamiento
================================

Redireccionamiento simpre utilizando el mod_alias de Apache, así como los tipos de redireccionamiento que existen.

Cuando se ha configurado un dominio en Apache, es necesario decidir bajo que configuración deseas que el contenido de tu página web se despliegue. La pregunta básica: con www o sin www o tambien http p https.

¿Que es mod_alias?
++++++++++++++++++
Es un módulo de Apache que permite manipular rutas de tu página web. Entre lo que se puede hacer es: crear rutas virtuales, redireccionar rutas, crear alias para rutas existentes incluso que no formen parte de tu sitio (en otro directorio dentro del servidor).

Verificamos que este habilitado mod_alias en Apache.::

	# grep mod_alias /etc/httpd/conf/httpd.conf 
	LoadModule alias_module modules/mod_alias.so


Ahora agregamos la configuración del redirect en el virtual host que lo requiera.::

	<VirtualHost *:80>

	ServerAdmin webmaster@example.com
	DocumentRoot /var/www/html/prueba
	ServerName public.com
	ServerAlias prueba.com
	ErrorLog /var/www/html/prueba/error.log
	# CustomLog /var/www/html/public_html/requests.log
	Redirect 301 / https://github.com/cgomeznt
	</VirtualHost>

	<VirtualHost *:443>

	ServerAdmin webmaster@example.com
	DocumentRoot /var/www/html/prueba
	ServerName public.com
	ServerAlias prueba.com
	ErrorLog /var/www/html/prueba/error.log
	# CustomLog /var/www/html/public_html/requests.log
	SSLEngine on
	SSLCertificateFile /etc/pki/tls/certs/ca.crt
	SSLCertificateKeyFile /etc/pki/tls/private/private.key
	</VirtualHost>


Hay 4 tipos de redireccionamiento que podemos utilizar según nuestro criterio.

301
Indica que el recurso se ha movido de forma permanente.

302
Indica que el recurso se ha movido de forma temporal.

303
Indica que el recurso ha sido reemplazado por otro.

402
Indica que el recurso se ha eliminado de forma permanente. Cuando se utiliza este estado el argumento URL debe omitirse.

Recargar la configuración de Apache.::

# sudo service httpd reload



