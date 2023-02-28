Openssl autofirmado
======================

Creamos la llave primaria y un certificado autofirmado.
+++++++++++++++++++++++++++++++++++++++++++++++++
::

	# cd /tmp
	# openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout privateKey.key -out certificate.crt
	Generating a 2048 bit RSA private key
	...............................+++
	..................+++
	writing new private key to 'privateKey.key'
	-----
	You are about to be asked to enter information that will be incorporated
	into your certificate request.
	What you are about to enter is what is called a Distinguished Name or a DN.
	There are quite a few fields but you can leave some blank
	For some fields there will be a default value,
	If you enter '.', the field will be left blank.
	-----
	Country Name (2 letter code) [XX]:VE
	State or Province Name (full name) []:DC
	Locality Name (eg, city) [Default City]:Caracas
	Organization Name (eg, company) [Default Company Ltd]:Personal Company Ltd
	Organizational Unit Name (eg, section) []:Tecnologia de la Informacion
	Common Name (eg, your name or your server's hostname) []:
	Email Address []:

**Nota** la CA que lo firma es /etc/pki/tls/certs/ca-bundle.crt

Centralizamos el certificado el Private Key y el certificado publico de la CA
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
::

	# cd /tmp/
	# mkdir certificado
	# cp /tmp/certificate.crt /tmp/privateKey.key /tmp/certificado/
	# cp /etc/pki/tls/certs/ca-bundle.crt /tmp/certificado/

Otorgamos los permisos
++++++++++++++++++
::

	# chown -R usuario: /tmp/certificado
	# chmod 0600 /tmp/certificado/*
