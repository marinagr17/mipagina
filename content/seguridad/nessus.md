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

## Alertas por correo electrónico.

Nuestro objetivo es configurar alertas automáticas que nos lleguen directamente al correo electrónico. 
Voy a usar Postfix para configurar el servidor de correo de Nessus. Para empezar Vamos a configurar la cerificación en dos pasos de nuestro correo:
![Nessus](/images/seguridad/16.png)
Al meternos en ‘Gestionar tu cuenta de Google’, se nos abre una pestaña, donde nos podemos meter en el apartado de seguridad:
![Nessus](/images/seguridad/17.png)
Una vez activada, vamos al buscador e introducimos ‘contraseña de aplicaciones’:
![Nessus](/images/seguridad/18.png)
![Nessus](/images/seguridad/19.png)
![Nessus](/images/seguridad/20.png)
Nos genera una contraseña. Ahora instalamos postfix y lo configuramos:
```bash
nessus@debian:~$ sudo apt install postfix -y
```
![Nessus](/images/seguridad/21.png)
 En este apartado introducimos el FQDN de nuestra máquina. 
![Nessus](/images/seguridad/22.png)
Una vez instalado podemos proceder a configurarlo, para ello modificaremos el archivo:
```bash
GNU nano 8.4                  /etc/postfix/main.cf   
  GNU nano 8.4                                                                                                                                                                                                                                                                                                                                                                                                                                                                          /etc/postfix/main.cf *                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  
# ==========================
#  CONFIGURACIÓN PRINCIPAL DE POSTFIX
#  Adaptado para usar el SMTP de Gmail
#  FQDN del servidor: nessus.org
# ==========================

# Nivel de compatibilidad (por defecto en versiones recientes de Postfix)
compatibility_level = 3.9

# Nombre completo (FQDN) del servidor
myhostname = nessus.org

# Dominio que aparecerá como origen de los correos salientes
myorigin = /etc/mailname

# Interfaces en las que Postfix escuchará (todas)
inet_interfaces = all

# Protocolos IP (soporta IPv4 e IPv6)
inet_protocols = all

# Máquinas de confianza (solo este host podrá enviar)
mynetworks_style = host
mynetworks = 127.0.0.0/8 [::1]/128

# Destinos locales (dominios que este servidor considera propios)
mydestination = $myhostname, localhost.$mydomain, localhost

# No limitar el tamaño del buzón (0 = ilimitado)
mailbox_size_limit = 0

# Separador entre nombre de usuario y extensión (usuario+info@dominio)
recipient_delimiter = +

# Archivos de alias locales
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases

# Desactivar notificaciones tipo "biff"
biff = no

# ==========================
#  CONFIGURACIÓN PARA ENVIAR CORREOS VIA GMAIL
# ==========================

# Servidor SMTP de Gmail y su puerto (STARTTLS en 587)
relayhost = [smtp.gmail.com]:587

# Activar el uso de TLS
smtp_use_tls = yes
smtp_tls_security_level = encrypt
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

# Activar autenticación SASL para Gmail
smtp_sasl_auth_enable = yes

# Archivo con usuario y contraseña (creado por ti en /etc/postfix/sasl_passwd)
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd

# Opciones de seguridad SASL
smtp_sasl_security_options = noanonymous
smtp_sasl_tls_security_options = noanonymous
smtp_sasl_mechanism_filter = plain, login

# ==========================
#  CONFIGURACIÓN DEL SERVIDOR LOCAL
# ==========================

# Banner de presentación cuando alguien conecta por SMTP
smtpd_banner = $myhostname ESMTP $mail_name (Debian)

# Nivel de seguridad TLS del servidor local (no afecta al envío)
smtpd_tls_security_level = may
smtpd_tls_cert_file = /etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file  = /etc/ssl/private/ssl-cert-snakeoil.key

# Restricciones básicas para el reenvío
smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination

# ==========================
#  FIN DEL ARCHIVO
# ==========================
```
A continuación, editamos, o creamos si no existe, el siguiente archivo:
```bash
nessus@debian:~$ sudo nano /etc/postfix/sasl_passwd
  GNU nano 8.4                    /etc/postfix/sasl_passwd *                           
[smtp.gmail.com]:587    marinagraciaontanilla@gmail.com:contraseña
```
En contraseña pondremos la contraseña de aplicación que se nos generó anteriormente. La contraseña aparece con espacios, en este fichero la colocamos sin espacios.
Guardamos el archivo y le cambiamos los permisos:
```bash
nessus@debian:~$ sudo postmap /etc/postfix/sasl_passwd 
nessus@debian:~$ sudo chmod 400 /etc/postfix/sasl_passwd
```
postmap es para convertir el archivo en un formato de base de datos que Postfix puede leer. 
Reiniciamos postfix:
```bash
nessus@debian:~$ sudo systemctl restart postfix
```
Ahora pasamos a configurar el SMTP de nessus:
![Nessus](/images/seguridad/23.png)
En el host tenemos que poner el nombre de nuestro host, ya que si ponemos localhost nos da un error. Para ello debemos hacer el siguiente cambio en el /etc/hosts:
```bash
 GNU nano 8.4                           /etc/hosts                                    
127.0.0.1       nessus.org
. . .
```
Puerto 25 porque Postfix escucha en ese puerto, que es el puerto estándar de SMTP. Esto es porque usamos Postfix local, si fuese gmail externo sería el 587.
En el correo debemos poner el que usamos en ‘/etc/postfix/sasl_passwd’.
Como Nessus habla con Postfix local, en encryption vamos a seleccionar NONE.
Hostname sirve para los enalces dentro de los emails.
En auth method no ponemos nada, ya que Postfix acepta mensajes desde localhost sin credenciales.
A continuación, he programado un nuevo escáner que se ha ejecutado, y al que he configurado para que cuando termine me notifique por correo.
![Nessus](/images/seguridad/24.png)

Una vez finalizado el escáner, comprobamos nuestro correo:
![Nessus](/images/seguridad/25.png)

El escaneo evalúa los servicios y puertos abiertos del equipo, identificando posibles fallos de configuración, software desactualizado y debilidades de seguridad que podrían ser explotadas por un atacante.
Nessus analiza estos elementos mediante plugins especializados, los cuales comparan las versiones de los servicios detectados con bases de datos de vulnerabilidades conocidas.
En este caso, el informe muestra un resumen con las vulnerabilidades más graves encontradas, clasificadas por nivel de severidad (Crítica, Alta, Media, Baja e Informativa).

## Prueba de uso en red.

Para hacer una prueba de uso en red, vamos a seleccionar varios hosts que me han aparecido en un nuevo escaneo de descubrimiento de hosts. He creado varias máquinas virtuales para poder ejecutar el escáner con credenciales SSH y poder realizarlo completo.

![Nessus](/images/seguridad/26.png)
Copiamos la clave publica en las tres máquinas y creamos un escáner nuevo como hemos hecho anteriormente. 
Vamos a elegir el siguiente escáner:
![Nessus](/images/seguridad/27.png)
Y configuramos:
![Nessus](/images/seguridad/28.png)
![Nessus](/images/seguridad/29.png)

En el apartado “Dynamic Pluggins” podemos especificar la ID de vulnerabilidad que queremos que el escaneo busque.
![Nessus](/images/seguridad/30.png)
Esta ID es una vulnerabilidad conocida en Log4j que permite la ejecución remota de un código. Como podemos ver Nessus tiene pluggins dedicados para detectarla. Guardamos y lanzamos el escaneo.
En la pestaña ‘schedule’ vamos a programarlo:
![Nessus](/images/seguridad/31.png)
Nessus Essentials solo permite una única programación.
En la opción ‘Targets to prioritize credentials’ ponemos la ip de los hosts. Esto agiliza el proceso ya que no las busca, sino que se las indicamos.
![Nessus](/images/seguridad/32.png)
Lo ideal sería programar también un escáner de puertos para saber que puertos están en la red.
Como en mi red no hay vulnerabilidades, solo nos da datos informativos:
![Nessus](/images/seguridad/33.png)

## Escáner a máquina metasploitable.
Mediante un escáner de DiscoveryHost encontramos la ip de nuestra metaspoitable:
![Nessus](/images/seguridad/34.png)
Una vez tenemos la dirección, lanzamos un nuevo escáner para ver las vulnerabilidades. Configuramos servidor de correo, pluggins que no hacen falta los desactivamos para no cargar tanto el escáner y que sea más rápido:
![Nessus](/images/seguridad/35.png)
Si accedemos a las vulnerabilidades, nos aparece un informe:
![Nessus](/images/seguridad/36.png)

### Informe.

Podemos exportar los informes del escáner.
![Nessus](/images/seguridad/37.png)

Nos genera informes según clasifica las vulnerabilidades. 
Entre las más destacadas aparecen:
    - Debilidad en el generador de números aleatorios de OpenSSH/OpenSSL (Debian), que compromete la fortaleza de las claves criptográficas.
    - Protocolos SSL 2.0 y 3.0 activos, que ya no son seguros y deben deshabilitarse.
    - Vulnerabilidad “Ghostcat” en Apache Tomcat, que permite la inyección de peticiones a través del conector AJP.
    - Puerta trasera detectada (Bind Shell Backdoor) que indica una posible intrusión.
    - Sistema operativo Ubuntu 8.04.x sin soporte (EoL), lo que lo hace vulnerable al no recibir actualizaciones.
El host analizado (192.168.122.247) presenta un total de 172 vulnerabilidades, de las cuales 10 son críticas. El informe también incluye un apartado de remediaciones sugeridas, donde se recomiendan acciones para mitigar los riesgos detectados, como actualizar servicios (BIND, Samba), eliminar software malicioso o reinstalar aplicaciones comprometidas. En conjunto, este informe proporciona una visión general del estado de seguridad del sistema, permitiendo priorizar las correcciones necesarias para reducir la superficie de ataque.
Escaneo por pluggin:

![Nessus](/images/seguridad/38.png)

Escaneo por hots:
![Nessus](/images/seguridad/39.png)





