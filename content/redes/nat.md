+++
date = '2025-10-03'
draft = false
title = 'Configuración de un router (SNAT y DNAT)'
+++

## Qué vas a aprender.

 - Realiza la configuración de un router Linux
 - Activar el reenvío de paquetes.
 - Configurar reglas SNAT y DNAT.

## Ejercicio.

Creamos el siguiente escenario:

{{< image src="/images/redes/escenario.jpg" alt="Escenario" position="center" style="border-radius: 8px;" >}}


Llamaremos a las máquinas de la siguiente manera:

 - La máquina *router* la llamaremos **router.tunombre.org**, y tendrá una distribución **Debian** sin entorno gráfico.
 - La máquina *Servidor Web* la llamaremos **web.tunombre.org**, y tendrá una distribución **Ubuntu Server** (sin entorno gráfico).
 - El *cliente1* la llamaremos **cliente1.tunombre.org**, y tendrá una distribución **Fedora**.
 - El *cliente2* la llamaremos **cliente2.tunombre.org**, y tendrá un sistema operativo **Windows 11**.

Tendremos 3 redes:

 - Una red de tipo **NAT**, cuyas características son:
	- Utiliza un bridge llamado br-nat.
	- No tiene DHCP.
	- Está conectada la máquina **router**, y le da acceso a internet.
 - Una red de tipo **muy aislada**, cuyas características son:
	- Utiliza un bridge llamado br-red2.
	- Tiene que tener un direccionamiento con **masca de red /16**.
	- Están conectadas las máquinas **router, cliente1 y cliente2**.
 - Una red de **tipo aislada**:
	- Utiliza un bridge llamado br-red1.
	- No tiene servidor DHCP.
	- Tiene que tener un direccionamiento con **mascara de red /24**.
	- Están conectadas las máquinas **router, Servidor Web**.

## Configuración de las interfaces de las máquinas.

### Router linux.

Activamos el bit de forwarding. Vamos a trabajar sobre Debian13, por lo que la configuración es diferente:

```bash
#Para hacerlo persistente:
debian@marina:~$ sudo nano /etc/sysctl.d/99-forwarding.conf
  GNU nano 8.4            /etc/sysctl.d/99-forwarding.conf *                    
#activar forwarding
net.ipv4.ip_forward=1
debian@marina:~$ sudo sysctl --system
```
Configuración de las interfaces:

```bash
#static config for enp1s0 (nat)
auto enp1s0
iface enp1s0 inet static
        address 192.168.12.2
        netmask 255.255.255.0

#static config for enp7s0 (aislada)
auto enp7s0
iface enp7s0 inet static
        address 192.168.11.2
        netmask 255.255.255.0

#static config for enp8s0 (muy aislada)
#tiene que ser /16
auto enp8s0
iface enp8s0 inet static
        address 192.168.10.1
        netmask 255.255.0.0

debian@router:~$ sudo systemctl restart networking.service
```
