Corregir un  false positive
================================


Hay varias formas, pero lo vamos hacer por ID.

Primeramente debemos ver los LOG para identificar el bloqueo y ver cual es su ID::

  tail -f /var/log/httpd/modsec_audit.log

Ejemplo, vemos esto en el LOG::

  --3668d252-F--
  HTTP/1.1 406 Not Acceptable
  Content-Length: 249
  Keep-Alive: timeout=5, max=100
  Connection: Keep-Alive
  Content-Type: text/html; charset=iso-8859-1
  
  --3668d252-E--
  
  --3668d252-H--
  Message: Access denied with code 406 (phase 2). Pattern match "etc/passwd" at REQUEST_URI. [file "/etc/httpd/modsecurity.d/local_rules/modsecurity_localrules.conf"] [line "12"] [id "500001"]
  Apache-Error: [file "apache2_util.c"] [line 273] [level 3] [client 192.168.1.108] ModSecurity: Access denied with code 406 (phase 2). Pattern match "etc/passwd" at REQUEST_URI. [file "/etc/httpd/modsecurity.d/local_rules/modsecurity_localrules.conf"] [line "12"] [id "500001"] [hostname "192.168.1.109"] [uri "/etc/passwd"] [unique_id "ZNWguLOx1ktOXHKZWxwGQAAAAII"]
  Action: Intercepted (phase 2)
  Stopwatch: 1691721912663357 3451 (- - -)
  Stopwatch2: 1691721912663357 3451; combined=1845, p1=1243, p2=539, p3=0, p4=0, p5=63, sr=708, sw=0, l=0, gc=0
  Response-Body-Transformed: Dechunked
  Producer: ModSecurity for Apache/2.9.6 (http://www.modsecurity.org/); OWASP_CRS/3.3.4.
  Server: Apache/2.4.37 (AlmaLinux)
  Engine-Mode: "ENABLED"
  
  --3668d252-Z--

El ID es el id "500001", editamos el archivo /etc/httpd/conf.d/mod_security.conf::

  # vi /etc/httpd/conf.d/mod_security.conf

      <LocationMatch .*>
        # Esta fue una regla de prueba que habiamos colocado en /etc/httpd/modsecurity.d/local_rules/modsecurity_localrules.conf
        SecRuleRemoveById 500001
      </LocationMatch>

Reiniciamos el httpd::

  # systemctl restart httpd

