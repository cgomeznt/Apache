SSL
====

Debemos tener instalado mod_ssl y openssl.::

	# yum install mod_ssl openssl


En este laboratorio se firmo el certificado utilizando una entidad certificadora, ver el siguiente link

https://github.com/cgomeznt/Certificados/blob/master/guia/cacentos7.rst


Copiamos los archivos a la localidad correcta o donde el servicio Apache tenga permisos::

	# cp srvutils.crt /etc/pki/tls/certs
	# cp srvutils.key /etc/pki/tls/private/

**Recordemos que la Private Key tendrá una clave**

Ahora debemos actualizar la configuración de SSL de apache y buscamos las secciones de VirtualHostdefault:443 y descomentamos con la modificación que corresponda (ServerName www.srvutils.com:443)
 ::

	# vi +/SSLCertificateFile /etc/httpd/conf.d/ssl.conf
	SSLCertificateFile /etc/pki/tls/certs/srvutils.crt
	SSLCertificateKeyFile /etc/pki/tls/private/srvutils.key
	ServerName ejemplo:443

Podemos buscar las siguientes tres lineas en /etc/httpd/conf.d/ssl.conf y las modificamos, pero así solo tendríamos un certificado por IP, por tal motivo nos vamos al archivo de configuración del VirtualHost y los agregamos ahí.::

	# vi /etc/httpd/conf.d/ejemplo.com.conf

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
		             ServerAdmin webmaster@fabric.com
		             DocumentRoot /var/www/html/ejemplo.com
		             ServerName srvutils
		             ServerAlias srvutils.com
		             ErrorLog /var/www/html/ejemplo.com/error.log
		             CustomLog /var/www/html/ejemplo.com/requests.log common
		             # RedirectPermanent /welcome http://google.com
		             SSLEngine on
		             SSLCertificateFile /etc/pki/tls/certs/srvutils.crt
		             SSLCertificateKeyFile /etc/pki/tls/private/srvutils.key
		    </VirtualHost>



Reiniciamos y veremos que nos pide una clave.::

	# systemctl start httpd
	Enter SSL pass phrase for srvutils:443 (RSA) : 

Configurar el apache para que NO pregunte por la clave en cada reinicio
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Una tecnica es quitarle la clave al private key y ya::

	# openssl rsa -check -in keyservice/srvutils.key -out keyservice/srvutils-without.key

Vamos utilizar la técnica más elegante que es utilizar la Directiva de **SSLPassPhraseDialog** y esto debe ir en el archivo **httpd.conf**::


	SSLPassPhraseDialog |/path/to/passphrase-script

Luego creamos un script, (Recuerda darle permisos de ejcución)::

	# vi /path/to/passphrase-script
	#!/bin/sh
	echo "PASSWORD"

Le damos el permiso de ejecución::

	# chmod -x /path/to/passphrase-script


Tambien la podemos utilizar de esta forma::

	SSLPassPhraseDialog exec:/path/to/passphrase-script

	# service httpd configtest
	# service httpd restart

Existen muchas formas de verificar


Antes de continuar el certificado tiene en sus alter name un DNS llamado : srvscmutils.fabric.local


Verificamos el portal con el comando curl y veremos que nos indica que dicho portal no es seguro por no le indicamos cual es el certificado publico de la CA.::

	# curl https://srvscmutils.fabric.local
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

Si le indicamos el curl el parametro **-k** pues va omitir el certificado y NO va establecer el canal SSL::
	
	# curl https://srvscmutils.fabric.local -k
	<html>
	  <head>
		<title>www.ejemplo.com</title>
	  </head>
	  <body>
		<h1>Felicitaciones, se creo el Virtual Host de ejemplo.com</h1>
	  </body>
	</html>

Pero si al comando curl le indicamos cual es el certificado publico de la CA, validara el certificado y establecerá un canal SSL::

	# curl https://srvscmutils.fabric.local --cacert /opt/certificados/ca-fabric.crt 
	<html>
	  <head>
		<title>www.ejemplo.com</title>
	  </head>
	  <body>
		<h1>Felicitaciones, se creo el Virtual Host de ejemplo.com</h1>
	  </body>
	</html>

Ahora con OpenSSl vamos a consultar el portal para culminar de certificar::

	# openssl s_client -connect srvscmutils.fabric.local:443 -CAfile /opt/certificados/ca-fabric.crt

	CONNECTED(00000003)
	depth=1 C = VE, ST = DC, L = DC, O = Default Fabric ltd, OU = Support Criptography, CN = criptography, emailAddress = root@fabric.com
	verify return:1
	depth=0 C = VE, ST = DC, L = Caracas, O = PERSONAL, OU = TI, CN = srvutils
	verify return:1
	---
	Certificate chain
	 0 s:/C=VE/ST=DC/L=Caracas/O=PERSONAL/OU=TI/CN=srvutils
	   i:/C=VE/ST=DC/L=DC/O=Default Fabric ltd/OU=Support Criptography/CN=criptography/emailAddress=root@fabric.com
	---
	Server certificate
	-----BEGIN CERTIFICATE-----
	MIID2jCCAsKgAwIBAgIJAJfaNaChMvstMA0GCSqGSIb3DQEBCwUAMIGaMQswCQYD
	VQQGEwJWRTELMAkGA1UECAwCREMxCzAJBgNVBAcMAkRDMRswGQYDVQQKDBJEZWZh
	dWx0IEZhYnJpYyBsdGQxHTAbBgNVBAsMFFN1cHBvcnQgQ3JpcHRvZ3JhcGh5MRUw
	EwYDVQQDDAxjcmlwdG9ncmFwaHkxHjAcBgkqhkiG9w0BCQEWD3Jvb3RAZmFicmlj
	LmNvbTAeFw0yMTA5MDEyMTQ3MjFaFw0yMjAzMDUyMTQ3MjFaMF8xCzAJBgNVBAYT
	AlZFMQswCQYDVQQIDAJEQzEQMA4GA1UEBwwHQ2FyYWNhczERMA8GA1UECgwIUEVS
	U09OQUwxCzAJBgNVBAsMAlRJMREwDwYDVQQDDAhzcnZ1dGlsczCCASIwDQYJKoZI
	hvcNAQEBBQADggEPADCCAQoCggEBAJsIvlAeYH3dJLSRml2rccAJaSIcOjMQeGWT
	o0b6YHSnnitLIg8aUdUvvffjvZ0jDCDvuNC/0FIyvrv7YbSVAIABbYaAEbCfAtK9
	KntH3mUIQdZTrNYqvHJxsCuB55TH2ZRAQlwAcqJ0xJ65ZqxYxsAARFej+UgsmErL
	R7dZUMxyT5/VBhuOjFc59UdBZ5WWNcp4ofBL/N7gWMOLw/tltWUyJaqZXoKeSufg
	ga0FLYxuiQmTwrHajZkSsZ7GIYwaf2z964iICZPd4jsSlj4ptRGXM0GeESOkxdWT
	MqGbilaevBqpgpkb3Ut+2kiYcxdTzqKZZ42XKrvAFTKfNtY620UCAwEAAaNdMFsw
	CQYDVR0TBAIwADALBgNVHQ8EBAMCBeAwQQYDVR0RBDowOIIYc3J2c2NtdXRpbHMu
	ZmFicmljLmxvY2FsghZtb25pdG9yZW8uZmFicmljLmxvY2FshwTAqAAUMA0GCSqG
	SIb3DQEBCwUAA4IBAQAXzumElxMqS5xsU6qNpIBTf9xhlU6n5x/aNt9RTGTc6LTT
	+pcBJHhpEZ7eGQe3NbDIjgTv6gsdd/HNZCyiC3sXX5iWR/rcFlNdHfhiIUKOFQlz
	MN3/sCDHwIN3q5BQmH8zSIDHfGVp9lnRLwIkHL0WnB7/1cNRegbkyjXRjbFwlh9t
	K7qZe5NsmFzOwdtU+So0z69NHkArtE4c5DUMY7ThuTq1A3o6GpMeIZAYOjAICEFf
	mm2Xdb6ktkj9GmC0nixSsNtlD2EX5j/MG3Hcee6Y5runJiDnuditakuWf6aVIE7Y
	MZb1RnFOalASBBKEoiAhnolg8V3P04Lgvjw8PKug
	-----END CERTIFICATE-----
	subject=/C=VE/ST=DC/L=Caracas/O=PERSONAL/OU=TI/CN=srvutils
	issuer=/C=VE/ST=DC/L=DC/O=Default Fabric ltd/OU=Support Criptography/CN=criptography/emailAddress=root@fabric.com
	---
	No client certificate CA names sent
	Peer signing digest: SHA512
	Server Temp Key: ECDH, P-256, 256 bits
	---
	SSL handshake has read 1681 bytes and written 415 bytes
	---
	New, TLSv1/SSLv3, Cipher is ECDHE-RSA-AES256-GCM-SHA384
	Server public key is 2048 bit
	Secure Renegotiation IS supported
	Compression: NONE
	Expansion: NONE
	No ALPN negotiated
	SSL-Session:
	    Protocol  : TLSv1.2
	    Cipher    : ECDHE-RSA-AES256-GCM-SHA384
	    Session-ID: D604DE5A33E8065704B32FEA472223850DC1E1515809EAD313109FBC9B54AF97
	    Session-ID-ctx: 
	    Master-Key: 394C87E565B37F86E5D5D519A077EDBD8C5B3C39DB4AD4AAA9AC93EEF4349C388630072D2CAFB771180A4CFF0E5E91D5
	    Key-Arg   : None
	    Krb5 Principal: None
	    PSK identity: None
	    PSK identity hint: None
	    TLS session ticket lifetime hint: 300 (seconds)
	    TLS session ticket:
	    0000 - 1c b3 d4 87 0d 80 1d 49-be 65 3c d9 6e 3f 43 62   .......I.e<.n?Cb
	    0010 - ca 8c 17 8e 6b d2 21 ac-d5 a0 a7 0b db 3d 20 70   ....k.!......= p
	    0020 - ae 3c 76 88 1a b2 ef f5-3f 8d cd c1 0f 66 c5 11   .<v.....?....f..
	    0030 - 40 7d 18 b7 7d 39 9d 2b-ef 92 40 a5 53 e1 78 a6   @}..}9.+..@.S.x.
	    0040 - 8b 26 4d fc 1c fd de 4a-8e 69 63 f4 42 bf cc f7   .&M....J.ic.B...
	    0050 - 94 fd 1d ff f4 81 06 bd-c8 34 67 ca 2f 2c a4 e3   .........4g./,..
	    0060 - 6a e4 8c 9b 7a c8 e2 4a-27 de 88 b2 c0 6f dc cf   j...z..J'....o..
	    0070 - 9a 5b 4b 40 58 05 0d e6-03 c3 46 2f 49 c3 26 e7   .[K@X.....F/I.&.
	    0080 - 8a 4e d7 28 f4 11 72 6a-9f d6 29 88 f5 bc cf de   .N.(..rj..).....
	    0090 - ce f7 0a 97 19 50 59 fc-6a 48 c7 44 75 60 0c ce   .....PY.jH.Du`..
	    00a0 - 20 58 4e 00 31 23 95 52-d2 cf 43 55 9f 74 31 3d    XN.1#.R..CU.t1=
	    00b0 - ea e2 9e 6a ec 2c e4 70-dd af a1 d2 3d 80 43 60   ...j.,.p....=.C`

	    Start Time: 1630619652
	    Timeout   : 300 (sec)
	    Verify return code: 0 (ok)
	---

