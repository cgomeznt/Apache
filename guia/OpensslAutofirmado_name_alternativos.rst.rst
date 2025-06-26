Openssl autofirmado
======================

Esta tecnica nos permite agregar varios nombres alternativos y direcciones IP.

Creamos el archivo de configuración.
+++++++++++++++++++++++++++++++++++++++++++++++++
::

  # cd /tmp
  
  # cat configuracion.cnf
  [req]
  distinguished_name = req_distinguished_name
  x509_extensions = v3_req
  prompt = no
  [req_distinguished_name]
  C = VE
  ST =DC
  L = SomeCity
  O = Pegaso
  OU = MyDivision
  CN = optimuscallcenterqa.mydominio.com.ve
  [v3_req]
  keyUsage = critical, digitalSignature, keyAgreement
  extendedKeyUsage = serverAuth
  subjectAltName = @alt_names
  [alt_names]
  DNS.1 = callcenterqa.mydominio.com.ve
  DNS.2 = appcallcenter.mydominio.com.ve
  IP.1 = 10.194.4.216
  IP.1 = 10.194.4.217

Creamos la llave primaria y un certificado autofirmado.
--------------------------------------------------------

::

  openssl req -x509 -nodes -days 730 -newkey rsa:2048  -keyout server.key -out server.crt -config configuracion.cnf -sha256


Verificamos la existencia de los archivos::
  
  # ls server.{key,crt}
  server.crt  server.key
