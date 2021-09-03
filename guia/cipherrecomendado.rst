Cipher recomendado para cumplir con PCI
=======================================

Si es necesario deshabilitar SSL versión 2 de compatibilidad a fin de cumplir Requisitos de cumplimiento PCI, Tendrá que añadir la siguiente directiva en el archivo de configuración de Apache::

	SSLProtocol -all +TLSv1.2 +TLSv1.3
	SSLCipherSuite HIGH: MEDIUM:!SSLv2:!EXP:!ADH:!aNULL:!eNULL:!NULL

Si la directiva ya existe, probablemente tendrá que modificar para desactivar 2 versión SSL.

Este es otro ejemplo en el archivo ssl.conf::

	LoadModule ssl_module modules/mod_ssl.so
	SSLPassPhraseDialog  builtin
	SSLSessionCache         shmcb:/var/cache/mod_ssl/scache(512000)
	SSLSessionCacheTimeout  300
	SSLMutex default
	SSLRandomSeed startup file:/dev/urandom  256
	SSLRandomSeed connect builtin
	SSLCryptoDevice builtin
	SSLProtocol -all +TLSv1.2 +TLSv1.3
	SSLCipherSuite ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5



