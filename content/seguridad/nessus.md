+++
date = '2025-10-17T20:48:24+02:00'
draft = true
title = 'Nessus Essentials'
+++

Nessus Essentials, una versión gratuita del escáner
de vulnerabilidades Nessus de Tenable que permite escanear hasta 16 direcciones IP
para encontrar fallos de seguridad en sistemas y redes. Es una herramienta para
profesionales de ciberseguridad, educadores y estudiantes que desean realizar
evaluaciones de vulnerabilidades de alta velocidad y en profundidad sin costo, aunque
con algunas limitaciones en comparación con las versiones de pago.

## Instalación y configuración inicial.

Para descargarnos el paquete nos vamos a la página de [Nessus](https://www.tenable.com/downloads/nessus?loginAttempted=true):

(/images/seguridad/nessus1.png)

Una vez que tenemos el .deb lo instalamos con dpkg:
```bash
debian@marina:~$ sudo dpkg -i Nessus-10.9.4.deb
```
Activamos el servicio y comprobamos que escucha por el puerto 8834 que es el de
nessus:
```bash
debian@marina:~$ sudo systemctl start nessusd.service
debian@marina:~$ sudo ss -tulnp | grep nessusd
tcp   LISTEN 0      1024                              0.0.0.0:8834      0.0.0.0:*    users:(("nessusd",pid=1423,fd=18))                                                
tcp   LISTEN 0      1024                                 [::]:8834 [::]:*    users:(("nessusd",pid=1423,fd=19))
```
Nos vamos al navegador para poder configurar Nessus:

(/images/seguridad/nessus2.png)

A continuación, nos ofrece una serie de opciones:
(/images/seguridad/nessus3.png)

En este caso, vamos a elegir Nessus Essentials, ya que es la versión gratuita para uso personal/educativo. La limitación que encontraremos será el número de IPs que se pueden escanear. Nos registramos:

(/images/seguridad/nessus4.png)

Una vez hecho esto, esperamos a que se descarguen todos los pluggins, y a continuación nos aparecerá la pantalla de incio de Nessus.
(/images/seguridad/nessus5.png)

