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

![Nessus](/images/seguridad/nessus1.png)

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

![Nessus](/images/seguridad/nessus2.png)

A continuación, nos ofrece una serie de opciones:
![Nessus](/images/seguridad/nessus3.png)

En este caso, vamos a elegir Nessus Essentials, ya que es la versión gratuita para uso personal/educativo. La limitación que encontraremos será el número de IPs que se pueden escanear. Nos registramos:

![Nessus](/images/seguridad/nessus4.png)

Una vez hecho esto, esperamos a que se descarguen todos los pluggins, y a continuación nos aparecerá la pantalla de incio de Nessus.
![Nessus](/images/seguridad/nessus5.png)


## Primer escáner.

Si le damos a la opción de nuevo escaneo vemos que hay diversos tipos:
![Nessus](/images/seguridad/nessus6.png)

### Descubrimiento de hosts.

El primer escáner que aparece es el de ‘Descubrimiento de host’. Este nos sirve para saber que hosts hay en mi red y es el primer paso para cualquier evaluación de vulnerabilidades.
![Nessus](/images/seguridad/nessus7.png)

Al lanzar el escaneo de descubrimiento de hosts, vemos además de que hosts están en nuestra red, su información asociada, como la dirección IP, FQDN, sistemas operativos y puertos abiertos, si está disponible. Después de tener una lista de host, podemos elegir cual escanear:
![Nessus](/images/seguridad/nessus8.png)

Si accedemos a la pestaña ‘vulnerabilities’:

![Nessus](/images/seguridad/nessus9.png)

*Settings*: Proporciona información sobre la configuración del escaneo. No detecta vulnerabilidades, sino que reporta detalles operativos del proceso de escaneo, como las opciones de configuración usadas.  El 5 indica que esto se aplica en los 5 hosts escaneados.

*Port scanners*: Incluye plugins dedicados al escaneo de puertos para descubrir hosts activos y servicios abiertos. Ejemplos comunes son "Nessus SYN Scanner" (escaneo TCP SYN para puertos abiertos) o "Nessus UDP Scanner". En esencia, Nessus envía paquetes a rangos de puertos para ver si responden, lo que ayuda a confirmar hosts vivos sin una conexión completa. El descubrimiento de hosts en Nessus depende en gran medida de escaneos de puertos (configurados en la sección "Network Port Scanners" de la política). Aparece como INFO porque reporta los resultados del escaneo de puertos (por ejemplo, puertos abiertos detectados), no vulnerabilidades. 
Este plugin básicamente comprueba si mi host está activo en la red y lo hace mediante ping:

    - Ping ARP: Funciona si el host está en la misma subred local y se usa Ethernet.
    - Ping ICMP: Ping clásico.
    - Ping TCP: Envía un paquete SYN a un puerto del host y espera a un SYN/ACK o RST.
    - Ping UDP: Prueba usando servicios UDP conocidos como DNS, RCP o NTP.


### Escaneo de un host descubierto.

Si nos vamos a la sección hosts del anterior escaneo, podemos seleccionar que host queremos escanear:
![Nessus](/images/seguridad/10.png)

Se nos abre otra vez la lista de plantillas para seleccionar una. Tenable Nessus llena automáticamente la lista de Targets con los hosts que hemos seleccionado.
En este caso, vamos a seleccionar ‘Análisis de malware’.
![Nessus](/images/seguridad/11.png)

Este tipo de escaneo necesita credenciales autenticadas para acceder al interior del host y buscar malware en archivos, registros o procesos. Sin ellas, Nessus no puede hacer un chequeo profundo y falla.
Lo primero será generar un par de claves en la maquina del servidor:
```bash
nessus@debian:~/.ssh$ ssh-keygen 
. . . 
nessus@debian:~/.ssh$ ssh-copy-id -i nessus.pub marina@192.168.1.47
```
A continuación, añadimos credenciales.
![Nessus](/images/seguridad/12.png)
![Nessus](/images/seguridad/13.png)
Guardamos y probamos:
![Nessus](/images/seguridad/14.png)
![Nessus](/images/seguridad/15.png)

Ahora Nessus hizo un escaneo profundo (leyó directorios como /tmp, chequeó procesos, etc.), no solo superficial.
Se han detectado un total de 6 vulnerabilidades, todas calificadas como INFO (informativas, no críticas). En un escaneo de malware esto es común si no hay amenazas reales.
Port scanners: Nos da información de que puertos están abiertos. Nessus usa SSH para ejecutar comandos como netstat o ss directamente en mi máquina. No es una vulnerabilidad real, solo información sobre los puertos que están escuchando.
Puertos detectados:
```bash
Port 22/tcp was found to be open	→ SSH
Port 53/udp was found to be open	Port 53/tcp was found to be open → DNS
Port 67/udp was found to be open	→ DHCP
Port 1521/tcp was found to be open		→ ORACLE
Port 1716/udp was found to be open	 Port 1716/tcp was found to be open → KDE
Port 5353/udp was found to be open	→ mDNS
Port 5500/tcp was found to be open		→ HTTP
```
El resto de vulnerabilidades que aparecen en la lista nos proporcionan información como el hostname de la máquina, el FQDN.














