Instalar Apache en CentOS
==========================

Buscamos::

	# apt-cache search apache2
	
Instalamos:
	
	# apt-get install apache2

Iniciamos el servicio.::

	# systemctl start apache2

Verificamos que el puerto 80 este iniciado:

	# netstat -natp

Probamos desde un navegador colocando la IP del servidor.::

	http://192.168.1.10




