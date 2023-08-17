Como instalar ModSecurity, OWASP CRS con Apache en Debian 12/11/10
======================================================================

Instalar pre-requisitos ::
	
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

	apt-cache policy libapache2-mod-security2 modsecurity-crs libmodsecurity3

Consultamos las politicar::

	# apt-cache policy libapache2-mod-security2 modsecurity-crs libmodsecurity3
	libapache2-mod-security2:
	  Installed: (none)
	  Candidate: 2.9.8-1~pre1+0~20230105~bpo11+05396552
	  Version table:
		 2.9.8-1~pre1+0~20230105~bpo11+05396552 500
			500 http://modsecurity.digitalwave.hu/debian bullseye-backports/main amd64 Packages
		 2.9.3-3+deb11u1 500
			500 http://deb.debian.org/debian bullseye/main amd64 Packages
	modsecurity-crs:
	  Installed: (none)
	  Candidate: 3.3.5-1~bpo11+1
	  Version table:
		 3.3.5-1~bpo11+1 500
			500 http://modsecurity.digitalwave.hu/debian bullseye-backports/main amd64 Packages
		 3.3.0-1+deb11u1 500
			500 http://deb.debian.org/debian bullseye/main amd64 Packages
	libmodsecurity3:
	  Installed: (none)
	  Candidate: 3.0.11-1~pre1+0~20230726~bpo11+ccc2d9b5
	  Version table:
		 3.0.11-1~pre1+0~20230726~bpo11+ccc2d9b5 500
			500 http://modsecurity.digitalwave.hu/debian bullseye-backports/main amd64 Packages
		 3.0.4-2 500
			500 http://deb.debian.org/debian bullseye/main amd64 Packages

			
Continuamos instalando y configurando::

	apt install libapache2-mod-security2


	a2enmod security2

	# systemctl restart apache2

Nos aseguramos que este esta linea IncludeOptional /etc/modsecurity/*.conf::

	vi /etc/apache2/mods-enabled/security2.conf
	<IfModule security2_module>
			# Default Debian dir for modsecurity's persistent data
			SecDataDir /var/cache/modsecurity

			# Include all the *.conf files in /etc/modsecurity.
			# Keeping your local configuration in that directory
			# will allow for an easy upgrade of THIS file and
			# make your life easier
			IncludeOptional /etc/modsecurity/*.conf		#ESTA LINEA

			# Include OWASP ModSecurity CRS rules if installed
			IncludeOptional /usr/share/modsecurity-crs/*.load
	</IfModule>

Continuamos::

	mv /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf

Editamos el archivo y agragamos::

	vi /etc/modsecurity/modsecurity.conf

	SecRuleEngine DetectionOnly

	SecRuleEngine On

	# Log everything we know about a transaction.
	#SecAuditLogParts ABDEFHIJZ   #Esta linea la comentamos  y agregamos la siguiente linea


	SecAuditLogParts ABCEFHJKZ


Reiniciasmos y ya esta listo el mod_security, pero aun no tiene reglas::

	systemctl restart apache2

Creamos esta regla local para hacer el diagnostico::

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

esto se debe agregar en  VHOST y recuerda debe estar en site-aviable::

    <IfModule mod_security2.c>
      SecRuleEngine On
      # Se desactiva ModSecurity para el contexto de errores personalizados del WAF
      <location "/errors">
        SecRuleEngine Off
      </location>
      ##
    </IfModule>

Reiniciamos el apache y probamos::

	systemctl restart apache2


Implementando OWASP Core Rule Set para ModSecurity
========================================================================

Comenzamos::

	mkdir /etc/apache2/modsec/

Ver esta pagina https://coreruleset.org/installation/

Descargamos e instalamos y configuramos::

	wget https://github.com/coreruleset/coreruleset/archive/refs/tags/v3.3.5.tar.gz

	tar xvf v3.3.5.tar.gz -C /etc/apache2/modsec

	cp /etc/apache2/modsec/coreruleset-3.3.5/crs-setup.conf.example /etc/apache2/modsec/coreruleset-3.3.5/crs-setup.conf

Agregamos estas linias y comentamos la otra::

	vi /etc/apache2/mods-enabled/security2.conf

			Include /etc/apache2/modsec/coreruleset-3.3.5/crs-setup.conf
			Include /etc/apache2/modsec/coreruleset-3.3.5/rules/*.conf

	#Comentar esta linea
	#IncludeOptional /usr/share/modsecurity-crs/*.load

Reiniciamos apache::

	systemctl restart apache2

Montamos el LOG::

	tail -f /var/log/apache2/modsec_audit.log


En un Navegador ::

	http://www.public.com/?exec=/bin/bash

	Forbidden
	You don't have permission to access this resource.

	Apache/2.4.56 (Debian) Server at www.public.com Port 80


Delving into the OWASP Core Rule Set
-----------------------------------

The OWASP Core Rule Set (CRS) is a comprehensive tool with numerous customizable options. Its default configuration provides immediate security enhancements for most servers without impacting genuine visitors or Search Engine Optimization (SEO) bots. We will discuss a few significant aspects of the CRS in this section, but it’s always beneficial to explore the configuration files for a complete understanding of all available options.

Adjusting the CRS Configuration
----------------------------------
We initiate by opening the crs-setup.conf file where most of the CRS settings can be altered::

	 nano /etc/apache2/modsec/coreruleset-3.3.4/crs-setup.conf
	 

ModSecurity operates in two distinct modes:
----------------------------------------

Anomaly Scoring Mode: Recommended for most users, this mode assigns an ‘anomaly score’ each time a rule matches. After processing both inbound and outbound rules, the anomaly score is checked and a disruptive action is triggered if necessary. This action often takes the form of a 403 error. This mode provides accurate log information and offers greater flexibility in setting blocking policies.
Self-Contained Mode: In this mode, an action is applied instantly whenever a rule is matched. While it can reduce resource usage, it offers less flexibility and yields less informative audit logs. The rule matching process stops as soon as the first rule is met, and the specified action is executed.
Understanding Paranoia Levels
The OWASP CRS defines four paranoia levels:

	Paranoia Level 1: The default level suitable for the majority of users.
	
	Paranoia Level 2: For advanced users.
	
	Paranoia Level 3: For expert users.
	
	Paranoia Level 4: Only advisable under extraordinary circumstances.
	
	Higher paranoia levels enable additional rules and provide increased security but also augment the likelihood of blocking legitimate traffic due to false positives. It’s important to choose a paranoia level that aligns with your expertise and security needs.

Testing OWASP CRS on Your Server
---------------------------------------
To ensure the OWASP CRS is operating correctly on your server, use your internet browser to access the following URL. Remember to replace “yourdomain.com” with your actual domain:::

	https://www.yourdomain.com/index.html?exec=/bin/bash	

If you receive a 403 Forbidden error, it signifies that OWASP CRS is functioning as expected. If not, there might be a misstep in the setup process that requires your attention which the most common issue can be forgetting to change DetectionOnly to On or something similar.

Example:


Section 6: Addressing False Positives and Crafting Custom Exclusion Rules
---------------------------------------------------------------------------

Dealing with false positives while leveraging ModSecurity and OWASP CRS is often a perpetual task. However, the defense these systems offer against diverse threats makes the effort worthwhile. To begin the process, we recommend maintaining a low paranoia level to reduce the number of potential false positives.

Running the rule set with minimal false positives for a few weeks, or even months, before augmenting the paranoia level can prevent an overwhelming surge of false positives. This method enables you to acclimate to the system’s alerts and understand how to respond appropriately to each.

Managing False Positives in Known Applications
---------------------------------------------------
ModSecurity incorporates an innate capability to whitelist common actions that may inadvertently trigger false positives. Consider the following example::

	#SecAction \
	# "id:900130,\
	#  phase:1,\
	#  nolog,\
	#  pass,\
	#  t:none,\
	#  setvar:tx.crs_exclusions_cpanel=1,\
	#  setvar:tx.crs_exclusions_dokuwiki=1,\
	#  setvar:tx.crs_exclusions_drupal=1,\
	#  setvar:tx.crs_exclusions_nextcloud=1,\
	#  setvar:tx.crs_exclusions_phpbb=1,\
	#  setvar:tx.crs_exclusions_phpmyadmin=1,\
	#  setvar:tx.crs_exclusions_wordpress=1,\
	#  setvar:tx.crs_exclusions_xenforo=1"


To enact whitelisting for applications such as WordPress, phpBB, and phpMyAdmin, simply uncomment the respective lines and maintain the (1) value. For services not in use, adjust the value to (0) to prevent their whitelisting.

The updated configuration could look as follows::

	SecAction \
	 "id:900130,\
	  phase:1,\
	  nolog,\
	  pass,\
	  t:none,\
	  setvar:tx.crs_exclusions_phpbb=1,\
	  setvar:tx.crs_exclusions_phpmyadmin=1,\
	  setvar:tx.crs_exclusions_wordpress=1"
	  
In this instance, extraneous options have been removed, leaving the correct syntax for the configurations you require.


Creating Rule Exclusions Prior to CRS Implementation
-----------------------------------------------------
To craft custom exclusions, start by altering the name of the REQUEST-900-EXCLUSION-RULES-BEFORE-CRS-SAMPLE.conf file using the cp command::

	cp /etc/apache2/modsec/coreruleset-3.3.4/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example /etc/apache2/modsec/coreruleset-3.3.4/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
	
	
	
Remember, each exclusion rule needs to have a unique id:<rule number> to avoid errors during Apache2 service testing.

For example, certain REQUEST_URIs may trigger false positives. The following example illustrates two such scenarios involving Google PageSpeed beacon and the WPMU DEV plugin for WordPress::

	SecRule REQUEST_URI "@beginsWith /wp-load.php?wpmudev" "id:1544,phase:1,log,allow,ctl:ruleEngine=off"

	SecRule REQUEST_URI "@beginsWith /ngx_pagespeed_beacon" "id:1554,phase:1,log,allow,ctl:ruleEngine=off"
	
	
These rules will grant automatic permissions for any URL beginning with the specified path.

IP addresses can also be whitelisted in various ways::


	SecRule REMOTE_ADDR "^195\.151\.128\.96" "id:1004,phase:1,nolog,allow,ctl:ruleEngine=off"

	SecRule REMOTE_ADDR "@ipMatch 127.0.0.1/8, 195.151.0.0/24, 196.159.11.13" "phase:1,id:1313413,allow,ctl:ruleEngine=off"
	
In the first rule, a single IP address is whitelisted, while the second rule uses the @ipMatch directive for broader subnet matching. To block a subnet or IP address, simply replace allow with deny. With this level of flexibility, you can create intricate blacklists and whitelists, and even incorporate them with Fail2Ban for a more comprehensive security strategy.


Disabling Specific Rules
----------------------------
Instead of whitelisting an entire path, another approach involves disabling specific rules that consistently cause false positives. While this method requires more time and rigorous testing, it can prove highly beneficial in the long run.

Let’s say rules 941000 and 942999 in your /admin/ area are continually triggering unwarranted bans and blocks for your team. You would locate the rule ID in your ModSecurity logs and disable only that specific ID using the RemoveByID command::

	SecRule REQUEST_FILENAME "@beginsWith /admin" "id:1004,phase:1,pass,nolog,ctl:ruleRemoveById=941000-942999"

	
	
	
Implementing Log Rotation for ModSecurity
----------------------------------------------

Firstly, we need to create a new file specifically for ModSecurity rotation. To do this, use the nano command to create and open a new file named modsec::

	vi /etc/logrotate.d/modsec
	
Configuring the Rotation File
-----------------------------
In the newly created modsec file, input the following configuration::

	/var/log/modsec_audit.log
	{
			rotate 31
			daily
			missingok
			compress
			delaycompress
			notifempty
	}
	
https://www.linuxcapable.com/how-to-install-modsecurity-with-apache-on-debian-linux/

