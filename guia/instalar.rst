Instalar Apache en CentOS
==========================

Instalamos.::
	
	# yum install httpd

Iniciamos el servicio.::

	# /etc/init.d/httpd start

Permisologia en Iptables.::

	# vi /etc/sysconfig/iptables
	-A INPUT -p tcp --dport 80  -j ACCEPT
	-A INPUT -p tcp --dport 443  -j ACCEPT

	# service iptables restart

Probamos desde un navegador o con alguna herramienta desde consola.::

	# curl -I -s -S 127.0.0.1 -k
	HTTP/1.1 403 Forbidden
	Date: Mon, 29 Aug 2016 02:42:23 GMT
	Server: Apache/2.2.15 (CentOS)
	Accept-Ranges: bytes
	Content-Length: 4961
	Connection: close
	Content-Type: text/html; charset=UTF-8

