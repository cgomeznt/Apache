Convertid formato rst to HTML
=============================

Instalamos el paquete **python-docutils** que contiene **rst2html** que nos ayudara a leer los formatos .rst y realizar una salida en formato HTML.

Apache on the other hand has mod_ext_filter  which was made for piping content through an external tool. The setup was easy::

	# apt-get install python-docutils

Ahora en apache debemo activar el  **mod_ext_filter** ::

	# a2enmod ext_filter
	# systemctl restart apache2

Ahora en nuestro VHost agregamos::

	ExtFilterDefine rst2html mode=output \
	    outtype=text/html \
	    cmd="/usr/bin/rst2html"
	 
	<FilesMatch "\.rst$">
	    SetOutputFilter rst2html
	</FilesMatch>

Y si queremos que tenga sopoerte de html::

	ExtFilterDefine rst2html mode=output \
	    outtype=text/html \
	    EnableEnv=supports_html \
	    cmd="/usr/bin/rst2html"
	 
	<FilesMatch "\.rst$">
	    SetEnvIf Accept text/html supports_html
	    SetOutputFilter rst2html
	</FilesMatch>






