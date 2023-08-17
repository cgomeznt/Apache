
apt install apt-transport-https lsb-release ca-certificates curl gpg wget unzip -y

curl -fsSL https://modsecurity.digitalwave.hu/archive.key |  gpg --dearmor |  tee /usr/share/keyrings/digitalwave-modsecurity.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/digitalwave-modsecurity.gpg] http://modsecurity.digitalwave.hu/debian/ $(lsb_release -sc) main" | tee -a /etc/apt/sources.list.d/digitalwave-modsecurity.list

echo "deb [signed-by=/usr/share/keyrings/digitalwave-modsecurity.gpg] http://modsecurity.digitalwave.hu/debian/ $(lsb_release -sc)-backports main" |  tee -a /etc/apt/sources.list.d/digitalwave-modsecurity.list

cat << EOF | sudo tee -a /etc/apt/preferences.d/99modsecurity
Package: *nginx*
Pin: origin modsecurity.digitalwave.hu
Pin-Priority: 900

Package: *libapache2-mod-security2*
Pin: origin modsecurity.digitalwave.hu
Pin-Priority: 900

Package: *modsecurity-crs*
Pin: origin modsecurity.digitalwave.hu
Pin-Priority: 900

Package: *libmodsecurity*
Pin: origin modsecurity.digitalwave.hu
Pin-Priority: 900
EOF

apt update

apt install libapache2-mod-security2 -y


a2enmod security2


mv /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf

mkdir /etc/apache2/modsec/

systemctl restart apache2


mkdir /etc/apache2/modsec/

wget https://github.com/coreruleset/coreruleset/archive/refs/tags/v3.3.5.tar.gz

tar xvf v3.3.5.tar.gz -C /etc/apache2/modsec

cp /etc/apache2/modsec/coreruleset-3.3.5/crs-setup.conf.example /etc/apache2/modsec/coreruleset-3.3.5/crs-setup.conf





vi /etc/modsecurity/modsecurity_localrules.conf
# default action when matching rules
SecDefaultAction "phase:2,deny,log,status:406"

# [etc/passwd] is included in request URI
SecRule REQUEST_URI "etc/passwd" "id:'500001'"

# [../] is included in request URI
SecRule REQUEST_URI "\.\./" "id:'500002'"

# [<SCRIPT] is included in arguments
SecRule ARGS "<[Ss][Cc][Rr][Ii][Pp][Tt]" "id:'500003'"

# [SELECT FROM] is included in arguments
SecRule ARGS "[Ss][Ee][Ll][Ee][Cc][Tt][[:space:]]+[Ff][Rr][Oo][Mm]" "id:'500004'"


systemctl restart apache2

esto es en el VHOST y recuerda debe estar en site-aviable

    <IfModule mod_security2.c>
      SecRuleEngine On
      # Se desactiva ModSecurity para el contexto de errores personalizados del WAF
      <location "/errors">
        SecRuleEngine Off
      </location>
      ##
    </IfModule>

Montamos la lectura del LOG::

	tail -f /var/log/apache2/modsec_audit.log
	
Realizamos con un cliente el test::

	curl http://prueba.com/?etc/passwd
	
	
---------------------------------------------------------
root@norven:/etc/apache2/sites-available# cat testmod.conf
<VirtualHost *:80>
                ServerAdmin webmaster@example.com
                DocumentRoot /var/www/html/#testmod
                ServerName testmod
                ServerAlias public.com
                ErrorLog /var/log/apache2/public_html_error.log
                CustomLog /var/log/apache2/public_html_requests.log common
                <IfModule security2_module>
                        #SecRuleEngine On
                        SecRuleEngine DetectionOnly
                </IfModule>
</VirtualHost>
-------------------------------------------------------