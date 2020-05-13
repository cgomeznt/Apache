Run Apache non-root
====================

Si compilastes Apache y lo esta ejecutando como non-root, no podra iniciar por el puerto 443.

	
Instalamos el paquete authbind. Una vez que se haya instalado authbind, ejecute lo siguiente según los puertos en los que desee que Apache escuche::

	sudo touch /etc/authbind/byport/80
	sudo chmod 500 /etc/authbind/byport/80
	sudo chown tomcat /etc/authbind/byport/80
	sudo touch /etc/authbind/byport/443
	sudo chmod 500 /etc/authbind/byport/443
	sudo chown tomcat /etc/authbind/byport/443
	

En el template de Systemctl para Tomcat utilizamos::

	[Unit]
	Description=Apache - instance %i
	After=syslog.target network.target

	[Service]
	Type=forking

	User=weblogic
	Group=oinstall

	WorkingDirectory=/orasoft/product/apache/2.4.43/

	Environment="LD_LIBRARY_PATH=/orasoft/product/apr/lib"

	ExecStart=/bin/authbind -deep /orasoft/product/apache/2.4.43/bin/apachectl start
	ExecStop=/bin/authbind -deep /orasoft/product/apache/2.4.43/bin/apachectl stop
	
	RestartSec=10
	Restart=always

	[Install]
	WantedBy=multi-user.target
