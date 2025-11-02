Para pasar las evaluaciones de PCI
====================================

How to disable HTTP TRACE/TRACK methods
https://techglimpse.com/http-trace-track-methods-disable-web-server-apache/#:~:text=TRACE%20and%20TRACK%20are%20HTTP,attack%20which%20is%20explained%20here.

En /etc/httpd/conf/httpd.conf 
	TraceEnable Off


# Habilitar el HSHS
https://docs.microfocus.com/SM/9.60/Hybrid/Content/security/concepts/support_of_http_strict_transport_security_protocol.htm


# Hacer el redireccionamiento
<VirtualHost *:80>
   ServerName glpiqa.local.com.ve
   Redirect / https://glpiqa.local.com.ve

</VirtualHost>

<VirtualHost *:443>
	# Habilitar el HSHS
        Header always set Strict-Transport-Security "max-age=63072000; includeSubdomains; preload"
        Header always append X-Frame-Options SAMEORIGIN
        ServerAdmin webmaster@localhost
        ServerName glpiqa.local.com.ve
        DocumentRoot /var/www/html/glpi
        SSLEngine on
        SSLCertificateFile /etc/pki/tls/glpi/cert.crt
        SSLCertificateKeyFile /etc/pki/tls/glpi/private.key
        SSLCertificateChainFile /etc/pki/tls/glpi/ca.crt
	# Habilitar unicamente el TLS 2
        SSLProtocol -all +TLSv1.2
	# Solo permitir Cipher stronge
        SSLCipherSuite ALL:+HIGH:!ADH:!EXP:!SSLv2:!SSLv3:!MEDIUM:!LOW:!NULL:!aNULL
        SSLHonorCipherOrder on
        SSLCompression      off
        SSLSessionTickets   off


        <IfModule mod_dir.c>
            DirectoryIndex index.php index.pl index.cgi index.html index.xhtml index.htm
        </IfModule>
</VirtualHost>

