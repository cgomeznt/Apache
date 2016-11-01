Debugging SSL con openssl
===========================


Chequear los MD5 de una public key para estar seguro que hace match con el CSR o private key.::

	# openssl x509 -noout -modulus -in CA.crt | openssl md5
	(stdin)= 2b5ea007e469cb17dd768ee24b7e92d9

.::

	# openssl x509 -noout -modulus -in CA.csr | openssl md5
	unable to load certificate
	140665560598344:error:0906D06C:PEM routines:PEM_read_bio:no start line:pem_lib.c:703:Expecting: TRUSTED CERTIFICATE
	(stdin)= d41d8cd98f00b204e9800998ecf8427e

.::

	# openssl x509 -noout -modulus -in CA.key | openssl md5
	unable to load certificate
	140257806206792:error:0906D06C:PEM routines:PEM_read_bio:no start line:pem_lib.c:703:Expecting: TRUSTED CERTIFICATE
	(stdin)= d41d8cd98f00b204e9800998ecf8427e

Chequear la conexion SSL. Todos los certificados (incluyendo los Intermediates) seran mostrados.::

	# openssl s_client -connect prueba.com:443
