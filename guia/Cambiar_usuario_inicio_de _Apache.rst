Cambiar usuario de inicio de  Apache
=====================================

Creamos el usuario y el grupo::

	# adduser httpd

	# groupadd httpd

	# usermod httpd -G httpd

	# id httpd
	uid=1003(httpd) gid=1004(httpd) groups=1004(httpd)

Editamos el archivo de configuración "httpd.conf" y buscamos "User y Group" y lo cambiamos por el usuario que creamos::

	# vi /opt/apache2/2.4.28/conf/httpd.conf

	<IfModule unixd_module>
	#
	# If you wish httpd to run as a different user or group, you must run
	# httpd as root initially and it will switch.
	#
	# User/Group: The name (or #number) of the user/group to run httpd as.
	# It is usually good practice to create a dedicated user and group for
	# running httpd, as with most system services.
	#
	User daemon
	Group daemon

	</IfModule>

