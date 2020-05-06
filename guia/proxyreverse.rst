Configurar Proxy inverso
=========================================

Un Proxy es un servicio que actúa como intermediario entre una comunicación del tipo Cliente-Servidor.
Mientras que un proxy normal (Forward Proxy) hace que un servidor no te contacte directamente, sino que es el proxy quien se conecta con el servidor, manteniendo al cliente en el anonimato, un Reverse Proxy, mantiene al servidor en el anonimato para con sus clientes…

Supongamos que instalamos un servicio en nuestro servidor, este servicio responde al puerto 8080, es decir, accedemos a este servicio mediante http://www.example.com:8080.

Pero queremos acceder al servicio mediante http://servicio.example.com porque es más fácil para los usuarios.

Obviamente, no podemos configurar el puerto del servicio para que use el 80 porque tenemos un Apache o que ya está corriendo en ese puerto…

Entonces, “enmascaramos” la verdadera URL con el Reverse Proxy

En el backend se crea un vhost.::

	# vi /etc/httpd/conf.d/corporativo.com.conf
	<VirtualHost *:443>
		ServerAdmin webmaster@corporativo.com
		DocumentRoot /var/www/html/corporativo.com
		SSLEngine on
		SSLCertificateFile /etc/pki/tls/certs/ca.crt
		SSLCertificateKeyFile /etc/pki/tls/private/ca.key
		ServerName corporativo.com
		ErrorLog logs/corporativo.com-error_log
		CustomLog logs/corporativo.com-access_log common
		<IfModule security2_module>
		    SecRuleEngine Off
		</IfModule>
	</VirtualHost>


Crear un nuevo vhost para el frontend.::

	# vi /etc/httpd/conf.d/ejemplo.conf
	<VirtualHost *:80>
		ServerAdmin webmaster@ejemplo.com
		DocumentRoot /var/www/html/ejemplo.com
		ServerName ejemplo.com
		ProxyRequests Off
		ProxyPreserveHost On
		SSLProxyEngine On
		# Cuando en la URL coloquen http://ejemplo.com se cargara el contenido de /var/www/html/ejemplo.com
		# Pero cuando coloquen la URL http://ejemplo.com/ver se realizara el proxy inverso hacia https://corporativo.com
		ProxyPass  /ver https://corporativo.com
		ProxyPassReverse /ver https://corporativo.com
		ErrorLog logs/ejemplo.com-error_log
		CustomLog logs/ejemplo.com-access_log common
		<IfModule security2_module>
		    SecRuleEngine DetectionOnly
		</IfModule>
	</VirtualHost>


La directiva "ServerName" hace referencia a la URL a la que se dirige la petición. Deberá configurarse una entrada en el DNS que apunte al servidor.

La directiva ProxyRequests Off evita que el front-end sea utilizado como proxy, es decir, que usuarios puedan saltar al front-end y de ahí a cualquier otra dirección. Es muy importante dejarlo deshabilitado para evitar problemas de seguridad o incluso legales.

La directiva ProxyPreserveHost On permite que el salto del servidor de front-end al de back-end sea transparente para el usuario. Si no estuviera habilitada, el usuario se dirigiría a http://frontend.organización.com pero inmediatamente vería como la dirección cambia a http://backend.organizacion.com. Además, como en este supuesto el servidor de back-end no es visible desde Internet el usuario vería un error.

Por último, las directivas ProxyPass y ProxyPassReverse gestionan el salto y la vuelta del servidor de front-end al de back-end.

Habilitar el vhost y reiniciar el servicio

Este es un ejemplo cuando en el otro lado tiene certificado, debemos agregar SSLProxyEngine on::

	<VirtualHost *:80>
			 ServerAdmin webmaster@cursoinfraestructura.com.ve
			 DocumentRoot /var/www/html/public_html
			 ServerName www.cursoinfraestructura.com.ve
			 ServerAlias cursoinfraestructura.com.ve
			 ProxyRequests Off
			 ProxyPreserveHost On
			 ErrorLog logs/public_html_error.log
			 CustomLog logs/public_html_requests.log common
			 SSLProxyEngine on
			 SSLProxyVerify none
			 #SSLProxyCheckPeerCN off
             #SSLProxyCheckPeerName off
             #SSLProxyCheckPeerExpire off
			 ProxyPass  /sample https://192.168.1.20/sample
			 ProxyPassReverse /sample https://192.168.1.20/sample
	</VirtualHost>


Hacer un Proxy Pass en Cluster
++++++++++++++++++++++++++++++

::

   ## The rest of the traffic goes to the default webservers
   ## (load balanced cluster - equal traffic goes to "one" and "two")
      <Proxy balancer://web_servers>
        BalancerMember http://webserver_one.domain.lan 
        BalancerMember http://webserver_two.domain.lan 
      </Proxy>
     ProxyPass / balancer://web_servers/
     ProxyPassReverse / balancer://web_servers/
  </VirtualHost>
