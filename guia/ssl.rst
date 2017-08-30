SSL
====

Debemos tener instalado mod_ssl y openssl.::

	# yum install mod_ssl openssl

Creamos un certificado autofirmado.
--------------------------------------

Generamos la private key. ::

	# openssl genrsa -out ca.key 2048
	Generating RSA private key, 2048 bit long modulus
	.................+++
	........................+++
	e is 65537 (0x10001)

Vemos el contenido.::

	# cat ca.key 
	-----BEGIN RSA PRIVATE KEY-----
	MIIEowIBAAKCAQEA1H+sSZGtzGykLTszrn1vvpEaRHQIo6GVC8qnq1AvNYyGQDyG
	P+RdSLZ9PpRP04AupcKbaA3tATxlynr6EEE4ZqgLFJ30YpI4qQNv0eRsevre5Wu/
	c2VIDVk+xRvMFL8fnWTxtIH2JHMCpeV+4hdW3KAptEI9fb+jr1bvj0et+vjnd1br
	JvZnQbB/07eg/bnRcpqzXoLnrU84YyY7mYmrOYCGeinvMi7jXUnl7KXaibb8xO5A
	4frKfqkorMtSECWLXOwbmZ+YB/XDClhYCpbsUs1ZbOcdfuJbrrXQwvWzdd+sUXsw
	H8FtucNt006xGGOaASlClWdTdBrfxYGWEABzSQIDAQABAoIBAAnLnNiDU5yhwWuo
	V/iKJbWGIMzZAHDyiNlTTSlTd+mjAalCYPnfAAHTD7Dry0Y3mW7gqqNASRWOgC62
	PoKzTvNEecZIhbRpgx0fYG8vdWSx3cZ7kgayu4CKBZ+2aVDngoCR36ZvezYw6wVU
	r+WiJ8nhxCpgB0+dnuD9Q+u55SY1b3ZtBsrdKpLuC3YVkSDAksmqdIU5uERK2+pN
	qiHpg+y8+s/zlW/L6fMAXaKXbEMmxQhNJg2wTXz5BJG+lbv767V8MUyk5eYBfXuH
	jAzujFmMi044fohQBSB5jb0SulIqvL2yMfe1VjW5waam2R+Pl+Mov7tP9cUbJhPa
	22NjrbUCgYEA/3scx63ZSE3h3GqYYlNIDvBSnzInWnAGUdbaD8gSibMjspk/Wl2G
	UcSc8l/9BqRSomID+sf//WZRtI8Me7zPfDqKpcwGQzJ2yEG2FXRuaDcH4lnXvrG1
	R7/FNYf51gEEm/33kQNJUDoM8kv3RAymsAL3GBUe6rNUpRge6uM//lMCgYEA1O40
	Gn/7MA/Nwf2rTWIvs5MKHjMd0bVZ1L0kuuU9WhwbRuB1hbb4PvtfBW7pKGsGRqLL
	OpcyCWgfu73kKsNkUkyBQYhDyFfLHqCMmirTlSINFTgxdO9dQf11HzPTfs+F0q70
	RQpkzSjxFz8ZwRLLnu8knel4tEA6/E7n3AyLfHMCgYATjcCuJ8gxmIRo8l+nZuhk
	/E/Wj2gjq99P9DnMa2u/zk41JTWMHQxixcGda2taTslkVEwprZUSN/qY7zntXo4i
	2/gwqGTyT7J3sU/WZIruvweDc4zns4JEc5EMf9PHZVyM8+s21iGOWmMTSG0scCtx
	3Ug8N6GeJQuddzMmly4WsQKBgB90JIw5lZBu9TUP1Ms0ktlTAi6d3GzK/j8XxaI6
	FMsH1dutco7TDW64UTwLOzP2Q1IR4DWCeii7kdx424iZnmst0/YrO+APX/jhPIPV
	ibXA9u/Igj3E0iDaYP+/9yEHZLxPjdPZCjToNFz7vEEyFpQevWj6QRNXXZ9BxKxT
	yhMTAoGBAOZtx6UU/pF5XtuDOS6hqlL12HX1kCrqtom48+6FiFFEGs2fjjpuU/jW
	YpwbeFJRD7Cn2iF9bipj58qim8CmCwXeoZ6E81BuqMWVwCpVUH65/zh5Dj3DR8dx
	A8Gw3vLpLiyyepKJqEjZb4UT6ePO0OgxOoTKB9CT+2zSz8VRatUk
	-----END RSA PRIVATE KEY-----

Tambien pudieramos crear una clave privada RSA. Esta clave tendra es una clave de 1024 bits RSA que se cifra usando Triple-DES y se almacena en formato PEM de modo que sea legible como texto ASCII.::

	# openssl genrsa -des3 -out CA.key 1024

	Generating RSA private key, 1024 bit long modulus
	.........................................................++++++
	........++++++
	e is 65537 (0x10001)
	Enter PEM pass phrase:
	Verifying password - Enter PEM pass phrase:

Con la Key generada con clave, cada vez que inicie httpd nos pedira dicha clave.

Si por alguna razon queremos remover la clave del key, tenemos que saber cual es la clave y hacer.::

	# openssl rsa -in CA.key.org -out CA.key
	Enter pass phrase for CA.key.org:
	writing RSA key


Generamos el Request CSR. ::

	# openssl req -new -key ca.key -out ca.csr
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
	Organization Name (eg, company) [Default Company Ltd]:Prueba
	Organizational Unit Name (eg, section) []:Personal
	Common Name (eg, your name or your server's hostname) []:srv-01
	Email Address []:webmaster@dominio.local

	Please enter the following 'extra' attributes
	to be sent with your certificate request
	A challenge password []:Venezuela21
	An optional company name []:

vemos el contenido. ::

	# cat ca.csr 
	-----BEGIN CERTIFICATE REQUEST-----
	MIIC6zCCAdMCAQAwgYkxCzAJBgNVBAYTAlZFMQswCQYDVQQIDAJEQzEQMA4GA1UE
	BwwHQ2FyYWNhczEPMA0GA1UECgwGUHJ1ZWJhMREwDwYDVQQLDAhQZXJzb25hbDEP
	MA0GA1UEAwwGc3J2LTAxMSYwJAYJKoZIhvcNAQkBFhd3ZWJtYXN0ZXJAZG9taW5p
	by5sb2NhbDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANR/rEmRrcxs
	pC07M659b76RGkR0CKOhlQvKp6tQLzWMhkA8hj/kXUi2fT6UT9OALqXCm2gN7QE8
	Zcp6+hBBOGaoCxSd9GKSOKkDb9HkbHr63uVrv3NlSA1ZPsUbzBS/H51k8bSB9iRz
	AqXlfuIXVtygKbRCPX2/o69W749Hrfr453dW6yb2Z0Gwf9O3oP250XKas16C561P
	OGMmO5mJqzmAhnop7zIu411J5eyl2om2/MTuQOH6yn6pKKzLUhAli1zsG5mfmAf1
	wwpYWAqW7FLNWWznHX7iW6610ML1s3XfrFF7MB/BbbnDbdNOsRhjmgEpQpVnU3Qa
	38WBlhAAc0kCAwEAAaAcMBoGCSqGSIb3DQEJBzENDAtWZW5lenVlbGEyMTANBgkq
	hkiG9w0BAQUFAAOCAQEAFt1x8k45jUPsc2sf+M8g5IsCdwH2t2joZWmXXf5yz3My
	0Qgosl0woYAkfmC495JVQifq1tQsIVxXfdGjTCY2j76X/63h9ekamGIomQw31MFo
	yFEE2KUR3cmmLI2koJpK5Wi8jDt9rfqlH8myaY2IHX31rf80BKGfEDk8ZCPuTcgB
	PJTshwUoKL0P6B1zrd5RT0QEm8QswpMgwd3nobaAisIoNPHw1evyR9i/oqV9i1ef
	WShqrbo6yM7AnrcpTEIKeM8nyGW1jqSB9AEpeT7zxfUk160cIIgO5yGcBP4cxkF3
	qjNyfLWSScTjHqcr2c2hGI2iCODKXg7ZntSkAJOs0w==
	-----END CERTIFICATE REQUEST-----


Generamos la llave auto firmada (Self Singned Key).::

	# openssl x509 -req -days 180 -in ca.csr -signkey ca.key -out ca.crt
	Signature ok
	subject=/C=VE/ST=DC/L=Caracas/O=Prueba/OU=Personal/CN=srv-01/emailAddress=webmaster@dominio.local
	Getting Private key

Vemos el contenido
::

	# cat ca.crt 
	-----BEGIN CERTIFICATE-----
	MIIDkDCCAngCCQDMe17znZTZ8jANBgkqhkiG9w0BAQUFADCBiTELMAkGA1UEBhMC
	VkUxCzAJBgNVBAgMAkRDMRAwDgYDVQQHDAdDYXJhY2FzMQ8wDQYDVQQKDAZQcnVl
	YmExETAPBgNVBAsMCFBlcnNvbmFsMQ8wDQYDVQQDDAZzcnYtMDExJjAkBgkqhkiG
	9w0BCQEWF3dlYm1hc3RlckBkb21pbmlvLmxvY2FsMB4XDTE2MDgyOTAzNTE1OFoX
	DTE3MDIyNTAzNTE1OFowgYkxCzAJBgNVBAYTAlZFMQswCQYDVQQIDAJEQzEQMA4G
	A1UEBwwHQ2FyYWNhczEPMA0GA1UECgwGUHJ1ZWJhMREwDwYDVQQLDAhQZXJzb25h
	bDEPMA0GA1UEAwwGc3J2LTAxMSYwJAYJKoZIhvcNAQkBFhd3ZWJtYXN0ZXJAZG9t
	aW5pby5sb2NhbDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANR/rEmR
	rcxspC07M659b76RGkR0CKOhlQvKp6tQLzWMhkA8hj/kXUi2fT6UT9OALqXCm2gN
	7QE8Zcp6+hBBOGaoCxSd9GKSOKkDb9HkbHr63uVrv3NlSA1ZPsUbzBS/H51k8bSB
	9iRzAqXlfuIXVtygKbRCPX2/o69W749Hrfr453dW6yb2Z0Gwf9O3oP250XKas16C
	561POGMmO5mJqzmAhnop7zIu411J5eyl2om2/MTuQOH6yn6pKKzLUhAli1zsG5mf
	mAf1wwpYWAqW7FLNWWznHX7iW6610ML1s3XfrFF7MB/BbbnDbdNOsRhjmgEpQpVn
	U3Qa38WBlhAAc0kCAwEAATANBgkqhkiG9w0BAQUFAAOCAQEAFCYKCA9DT633sNIZ
	Zlyn9fl7BXKohjYxIUumByaP6xcuO6iteLsd4nAwPBDQJlSEN8B72PD1i0Jo4xLY
	03huQznR7rs8DXMfWgZtF8V/v3DOpo3z05tYLUk4u0I5TxgLL50ti09Q4d36bGyz
	goVaSwfI1LfoSvz/U3tt+O/IeHXuO1q6fSzz9sfpVm/2ily1ISCgcGHoWoiIkDG1
	8jKypWmWMLbLsgMKqLHywNFvOJ+cc6LC4v78EvrAt3nP+PQ5/XvN+HNF1ajzj+Cu
	rt5nFj7tH+ducqbe3b0mHuuhTveinD+4DzL8inolqoKTbpp7nKu7JaVPh2tZCNga
	yKOR1w==
	-----END CERTIFICATE-----

Copiamos los archivos a la localidad correcta.::

	# cp ca.crt /etc/pki/tls/certs
	# cp ca.key /etc/pki/tls/private/
	# cp ca.csr /etc/pki/tls/private/

	
WARNING: No mueva los archivos esto por el SELinux. Apache se quejara porque los archivos estan perdidos, los archivos no tienen permisos en SELinux.
Si los mueve, se debe indicar a SELinux el contexto de estos archivos, como la definicion correcta para /etc/pki/* qeu vien con la politica de SELinux.::

	# restorecon -RvF /etc/pki

Ahora debemos actualizar la configuracion de SSL de apache y buscamos las secciones de VirtualHostdefault:443 y descomentamos con la modificacion que corresponda (ServerName www.ejemplo.com:443)
 ::

	# vi +/SSLCertificateFile /etc/httpd/conf.d/ssl.conf
	SSLCertificateFile /etc/pki/tls/certs/ca.crt
	SSLCertificateKeyFile /etc/pki/tls/private/ca.key
	ServerName ejemplo.com:443

Podemos buscar las siguientes tres lineas en /etc/httpd/conf.d/ssl.conf y las modificamos, pero asi solo tendriamos un certificado por IP, por tal motivo nos vamos al archivo de configuracion del VirtualHost y los agregamos ahi.::

	# vi /etc/httpd/conf.d/ejemplo.com.conf

	# NameVirtualHost *:443:
	#
	# NOTE: NameVirtualHost cannot be used without a port specifier
	# (e.g. :80) if mod_ssl is being used, due to the nature of the
	# SSL protocol.
	#

	#
	# VirtualHost example:
	# Almost any Apache directive may go into a VirtualHost container.
	# The first VirtualHost section is used for requests without a known
	# server name.
	#
		    <VirtualHost *:443>
		             ServerAdmin webmaster@example.com
		             DocumentRoot /var/www/html/ejemplo.com
		             ServerName www.ejemplo.com
		             ServerAlias ejemplo.com
		             ErrorLog /var/www/html/ejemplo.com/error.log
		             #CustomLog /var/www/html/ejemplo.com/requests.log
		             # RedirectPermanent /welcome http://google.com
		             SSLEngine on
		             SSLCertificateFile /etc/pki/tls/certs/ca.crt
		             SSLCertificateKeyFile /etc/pki/tls/private/ca.key
		    </VirtualHost>



Verificamos la configuracion del apache y lo reiniciamos.::

	# service httpd configtest
	# service httpd restart

Verificamos el funcionamiento.::

	# curl https://ejemplo.com
	curl: (60) SSL certificate problem: self signed certificate
	More details here: http://curl.haxx.se/docs/sslcerts.html

	curl performs SSL certificate verification by default, using a "bundle"
	 of Certificate Authority (CA) public keys (CA certs). If the default
	 bundle file isn't adequate, you can specify an alternate file
	 using the --cacert option.
	If this HTTPS server uses a certificate signed by a CA represented in
	 the bundle, the certificate verification probably failed due to a
	 problem with the certificate (it might be expired, or the name might
	 not match the domain name in the URL).
	If you'd like to turn off curl's verification of the certificate, use
	 the -k (or --insecure) option.

	# curl https://ejemplo.com -k
	<html>
	  <head>
		<title>www.ejemplo.com</title>
	  </head>
	  <body>
		<h1>Felicitaciones, se creo el Virtual Host de ejemplo.com</h1>
	  </body>
	</html>

