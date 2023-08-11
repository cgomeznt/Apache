ab - Apache HTTP server benchmarking tool
===========================================

https://httpd.apache.org/docs/2.4/programs/ab.html#:~:text=ab%20is%20a%20tool%20for,installation%20is%20capable%20of%20serving.


La prueba más simple que puede hacer es realizar 1000 solicitudes, 10 a la vez
(lo que simula aproximadamente 10 usuarios simultáneos que obtienen 100 páginas cada uno, durante la duración de la prueba).

ab -n 1000 -c 10 -k -H "Accept-Encoding: gzip, deflate" http://192.168.1.109/
