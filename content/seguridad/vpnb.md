# Configuración OpenVPN Site-to-Site

## Arquitectura del Escenario

**Objetivo:** Conectar Cliente1 (10.50.50.2/24) con Cliente2 (10.60.60.2/24) a través de un túnel VPN site-to-site usando el servidor VPN como punto central.

### Topología de Red

```bash
NAT1 (10.99.80.2/24)
  ↓
Router r1 (10.50.50.1/24)
  ↓
Cliente1 (10.50.50.2/24)

Servidor VPN
  - IP Túnel: 5.168.1.100/24 ↔ 5.168.1.102/24
  - Red VPN interna: 10.99.99.0/24

NAT2 (10.99.70.2/24)
  ↓
Router r2 (10.60.60.1/24)
  ↓
Cliente2 (10.60.60.2/24)
```

### Máquinas en el Escenario

- **Servidor**: Máquina central que ejecuta el servidor OpenVPN
- **NAT1**: Gateway con acceso a Internet para el sitio 1
- **r1**: Router del sitio 1 (cliente OpenVPN)
- **Cliente1**: Máquina final del sitio 1
- **NAT2**: Gateway con acceso a Internet para el sitio 2
- **r2**: Router del sitio 2 (cliente OpenVPN)
- **Cliente2**: Máquina final del sitio 2

---

## FASE 1: Preparación del Servidor VPN

> **🖥️ MÁQUINA: Servidor**

### 1.1 Instalación de OpenVPN y Easy-RSA

```bash
# En el Servidor VPN
sudo apt update
sudo apt install openvpn easy-rsa -y
```

### 1.2 Configuración de la PKI (Infraestructura de Clave Pública)

```bash
# En el Servidor VPN
# Crear directorio para Easy-RSA
make-cadir ~/openvpn-ca
cd ~/openvpn-ca

# Editar variables (opcional pero recomendado)
nano vars
```

Modifica estos campos en el archivo `vars`:
```bash
set_var EASYRSA_REQ_COUNTRY    "ES"
set_var EASYRSA_REQ_PROVINCE   "Aragon"
set_var EASYRSA_REQ_CITY       "Zaragoza"
set_var EASYRSA_REQ_ORG        "MiEmpresa"
set_var EASYRSA_REQ_EMAIL      "admin@miempresa.com"
set_var EASYRSA_REQ_OU         "IT"
```

### 1.3 Generar Certificados

```bash
# En el Servidor VPN
# Inicializar PKI
./easyrsa init-pki

# Crear CA (Autoridad Certificadora)
./easyrsa build-ca nopass
# Nombre sugerido: servidor-vpn-ca

# Generar certificado y clave del servidor
./easyrsa build-server-full servidor nopass

# Generar certificados para los sitios remotos
./easyrsa build-client-full sitio1 nopass
./easyrsa build-client-full sitio2 nopass

# Generar parámetros Diffie-Hellman
./easyrsa gen-dh

# Generar clave TLS (seguridad adicional)
openvpn --genkey secret ta.key
```

### 1.4 Copiar Certificados al Directorio de OpenVPN

```bash
# En el Servidor VPN
sudo mkdir -p /etc/openvpn/server/keys
sudo cp pki/ca.crt /etc/openvpn/server/keys/
sudo cp pki/issued/servidor.crt /etc/openvpn/server/keys/
sudo cp pki/private/servidor.key /etc/openvpn/server/keys/
sudo cp pki/dh.pem /etc/openvpn/server/keys/
sudo cp ta.key /etc/openvpn/server/keys/
```

---

## FASE 2: Configuración del Servidor OpenVPN

> **🖥️ MÁQUINA: Servidor**

### 2.1 Crear Archivo de Configuración del Servidor

```bash
# En el Servidor VPN
sudo nano /etc/openvpn/server/server.conf
```

Contenido del archivo:

```bash
# Puerto y protocolo
port 1194
proto udp
dev tun

# Certificados y claves
ca /etc/openvpn/server/keys/ca.crt
cert /etc/openvpn/server/keys/servidor.crt
key /etc/openvpn/server/keys/servidor.key
dh /etc/openvpn/server/keys/dh.pem
tls-auth /etc/openvpn/server/keys/ta.key 0

# Red del túnel VPN
server 10.99.99.0 255.255.255.0
topology subnet

# Permitir comunicación entre clientes
client-to-client

# Rutas para las redes remotas
push "route 10.50.50.0 255.255.255.0"
push "route 10.60.60.0 255.255.255.0"

# Directorio para configuraciones específicas por cliente
client-config-dir /etc/openvpn/server/ccd

# Rutas internas (necesario para site-to-site)
route 10.50.50.0 255.255.255.0
route 10.60.60.0 255.255.255.0

# Mantener conexión activa
keepalive 10 120

# Compresión
compress lz4-v2
push "compress lz4-v2"

# Cifrado
cipher AES-256-GCM
auth SHA256

# Usuario y grupo (seguridad)
user nobody
group nogroup

# Persistencia
persist-key
persist-tun

# Logs
status /var/log/openvpn/openvpn-status.log
log-append /var/log/openvpn/openvpn.log
verb 3

# Notificaciones duplicadas
explicit-exit-notify 1
```

### 2.2 Crear Configuraciones Específicas por Cliente (CCD)

```bash
# En el Servidor VPN
sudo mkdir -p /etc/openvpn/server/ccd
```

**Para sitio1 (Cliente1):**
```bash
# En el Servidor VPN
sudo nano /etc/openvpn/server/ccd/sitio1
```

Contenido:
```bash
iroute 10.50.50.0 255.255.255.0
ifconfig-push 10.99.99.10 255.255.255.0
```

**Para sitio2 (Cliente2):**
```bash
# En el Servidor VPN
sudo nano /etc/openvpn/server/ccd/sitio2
```

Contenido:
```bash
iroute 10.60.60.0 255.255.255.0
ifconfig-push 10.99.99.20 255.255.255.0
```

### 2.3 Habilitar IP Forwarding

```bash
# En el Servidor VPN
# Editar sysctl
sudo nano /etc/sysctl.conf
```

Descomentar o añadir:
```bash
net.ipv4.ip_forward=1
```

Aplicar cambios:
```bash
# En el Servidor VPN
sudo sysctl -p
```

### 2.4 Configurar Firewall

```bash
# En el Servidor VPN

# 1. Permitir tráfico OpenVPN entrante
sudo iptables -A INPUT -p udp --dport 1194 -j ACCEPT

# 2. Permitir tráfico establecido y relacionado
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT

# 3. Permitir forward desde el túnel VPN hacia la red local
sudo iptables -A FORWARD -s 10.99.99.0/24 -j ACCEPT

# 4. Permitir forward desde las redes remotas
sudo iptables -A FORWARD -s 10.50.50.0/24 -j ACCEPT
sudo iptables -A FORWARD -s 10.60.60.0/24 -j ACCEPT

# 5. Configurar NAT/MASQUERADE para el túnel VPN
sudo iptables -t nat -A POSTROUTING -s 10.99.99.0/24 -o ens6 -j MASQUERADE

# 6. Permitir tráfico en la interfaz tun0
sudo iptables -A INPUT -i tun0 -j ACCEPT
sudo iptables -A FORWARD -i tun0 -j ACCEPT
sudo iptables -A FORWARD -o tun0 -j ACCEPT

# 7. Verificar las reglas
sudo iptables -L -v -n
sudo iptables -t nat -L -v -n
```

### 2.5 Crear Directorio de Logs

```bash
# En el Servidor VPN
sudo mkdir -p /var/log/openvpn
```

### 2.6 Iniciar y Habilitar el Servicio

```bash
# En el Servidor VPN
sudo systemctl start openvpn-server@server
sudo systemctl enable openvpn-server@server
sudo systemctl status openvpn-server@server
```

---

## FASE 3: Configuración del Sitio 1 (Router r1)

### 3.1 Transferir Certificados al Router r1

Desde el servidor VPN:
```bash
# Crear paquete de certificados
cd ~/openvpn-ca
tar czf sitio1-certs.tar.gz \
    pki/ca.crt \
    pki/issued/sitio1.crt \
    pki/private/sitio1.key \
    ta.key

# Transferir (ajustar IP y usuario según tu router)
scp sitio1-certs.tar.gz usuario@10.99.80.1:/tmp/
```

En el router r1:
```bash
# Descomprimir y mover certificados
cd /tmp
tar xzf sitio1-certs.tar.gz
sudo mkdir -p /etc/openvpn/client
sudo mv pki/ca.crt /etc/openvpn/client/
sudo mv pki/issued/sitio1.crt /etc/openvpn/client/
sudo mv pki/private/sitio1.key /etc/openvpn/client/
sudo mv ta.key /etc/openvpn/client/
```

### 3.2 Crear Configuración del Cliente

```bash
sudo nano /etc/openvpn/client/sitio1.conf
```

Contenido:
```bash
# Cliente
client
dev tun
proto udp

# Servidor remoto (IP pública del servidor VPN)
remote 5.168.1.100 1194

# Resolver DNS infinitamente
resolv-retry infinite

# No bind a puerto local específico
nobind

# Persistencia
persist-key
persist-tun

# Certificados
ca /etc/openvpn/client/ca.crt
cert /etc/openvpn/client/sitio1.crt
key /etc/openvpn/client/sitio1.key
tls-auth /etc/openvpn/client/ta.key 1

# Cifrado (debe coincidir con el servidor)
cipher AES-256-GCM
auth SHA256

# Compresión
compress lz4-v2

# Verificar certificado del servidor
remote-cert-tls server

# Logs
verb 3

# Rutas para alcanzar la red remota
route 10.60.60.0 255.255.255.0
```

### 3.3 Habilitar IP Forwarding en r1

```bash
sudo nano /etc/sysctl.conf
```

Añadir:
```bash
net.ipv4.ip_forward=1
```

Aplicar:
```bash
sudo sysctl -p
```

### 3.4 Configurar Enrutamiento Estático en r1

```bash
# Ruta para que Cliente1 pueda alcanzar Cliente2
sudo ip route add 10.60.60.0/24 via 10.200.0.1 dev tun0

# Para hacer permanente (añadir a /etc/network/interfaces o usar scripts)
echo "post-up ip route add 10.60.60.0/24 via 10.200.0.1 dev tun0" | \
    sudo tee -a /etc/network/interfaces
```

### 3.5 Iniciar Cliente OpenVPN

```bash
sudo systemctl start openvpn-client@sitio1
sudo systemctl enable openvpn-client@sitio1
sudo systemctl status openvpn-client@sitio1
```

---

## FASE 4: Configuración del Sitio 2 (Router r2)

### 4.1 Transferir Certificados al Router r2

Desde el servidor VPN:
```bash
cd ~/openvpn-ca
tar czf sitio2-certs.tar.gz \
    pki/ca.crt \
    pki/issued/sitio2.crt \
    pki/private/sitio2.key \
    ta.key

scp sitio2-certs.tar.gz usuario@10.99.70.1:/tmp/
```

En el router r2:
```bash
cd /tmp
tar xzf sitio2-certs.tar.gz
sudo mkdir -p /etc/openvpn/client
sudo mv pki/ca.crt /etc/openvpn/client/
sudo mv pki/issued/sitio2.crt /etc/openvpn/client/
sudo mv pki/private/sitio2.key /etc/openvpn/client/
sudo mv ta.key /etc/openvpn/client/
```

### 4.2 Crear Configuración del Cliente

```bash
sudo nano /etc/openvpn/client/sitio2.conf
```

Contenido:
```conf
client
dev tun
proto udp
remote 5.168.1.102 1194
resolv-retry infinite
nobind
persist-key
persist-tun

ca /etc/openvpn/client/ca.crt
cert /etc/openvpn/client/sitio2.crt
key /etc/openvpn/client/sitio2.key
tls-auth /etc/openvpn/client/ta.key 1

cipher AES-256-GCM
auth SHA256
compress lz4-v2
remote-cert-tls server

verb 3

# Ruta para alcanzar la red remota
route 10.50.50.0 255.255.255.0
```

### 4.3 Habilitar IP Forwarding en r2

```bash
sudo nano /etc/sysctl.conf
```

Añadir:
```bash
net.ipv4.ip_forward=1
```

Aplicar:
```bash
sudo sysctl -p
```

### 4.4 Configurar Enrutamiento Estático en r2

```bash
sudo ip route add 10.50.50.0/24 via 10.200.0.1 dev tun0

# Hacer permanente
echo "post-up ip route add 10.50.50.0/24 via 10.200.0.1 dev tun0" | \
    sudo tee -a /etc/network/interfaces
```

### 4.5 Iniciar Cliente OpenVPN

```bash
sudo systemctl start openvpn-client@sitio2
sudo systemctl enable openvpn-client@sitio2
sudo systemctl status openvpn-client@sitio2
```

---

## FASE 5: Verificación y Pruebas

### 5.1 Verificar Estado de las Conexiones

**En el Servidor VPN:**
```bash
# Ver estado
sudo systemctl status openvpn-server@server

# Ver log
sudo tail -f /var/log/openvpn/openvpn.log

# Ver clientes conectados
sudo cat /var/log/openvpn/openvpn-status.log
```

**En Router r1 y r2:**
```bash
# Verificar estado
sudo systemctl status openvpn-client@sitio1  # o sitio2

# Ver interfaz tun
ip addr show tun0

# Ver rutas
ip route | grep tun
```

### 5.2 Pruebas de Conectividad

**Desde Cliente1 (10.50.50.2):**
```bash
# Ping al servidor VPN
ping 10.200.0.1

# Ping al router r2
ping 10.200.0.20

# Ping a Cliente2
ping 10.60.60.2
```

**Desde Cliente2 (10.60.60.2):**
```bash
# Ping al servidor VPN
ping 10.200.0.1

# Ping al router r1
ping 10.200.0.10

# Ping a Cliente1
ping 10.50.50.2
```

### 5.3 Traceroute para Verificar Ruta

```bash
# Desde Cliente1 a Cliente2
traceroute 10.60.60.2

# Debería mostrar:
# 1. 10.50.50.1 (router r1)
# 2. 10.200.0.1 (servidor VPN)
# 3. 10.60.60.2 (Cliente2)
```

---

## FASE 6: Solución de Problemas Comunes

### 6.1 Problema: Los Clientes No Se Conectan al Servidor

**Posibles causas:**
1. Firewall bloqueando puerto 1194
2. Certificados incorrectos
3. Diferencias en configuración de cifrado

**Soluciones:**
```bash
# Verificar firewall
sudo ufw status
sudo ufw allow 1194/udp

# Verificar logs del servidor
sudo journalctl -u openvpn-server@server -f

# Verificar logs del cliente
sudo journalctl -u openvpn-client@sitio1 -f

# Verificar conectividad al servidor
telnet 5.168.1.100 1194
```

### 6.2 Problema: Conexión Establecida pero Sin Comunicación

**Posibles causas:**
1. IP forwarding deshabilitado
2. Rutas estáticas faltantes
3. Firewall bloqueando tráfico

**Soluciones:**
```bash
# Verificar IP forwarding
sysctl net.ipv4.ip_forward
# Debe retornar: net.ipv4.ip_forward = 1

# Verificar rutas
ip route | grep 10.200.0.0
ip route | grep 10.50.50.0
ip route | grep 10.60.60.0

# Verificar NAT/iptables
sudo iptables -t nat -L -v -n
```

### 6.3 Problema: Certificados Rechazados

**Error típico:** `TLS Error: cannot locate HMAC in incoming packet`

**Solución:**
```bash
# Verificar que la directiva key-direction sea correcta
# Servidor: tls-auth ta.key 0
# Cliente: tls-auth ta.key 1

# Regenerar certificados si es necesario
cd ~/openvpn-ca
./easyrsa revoke sitio1
./easyrsa gen-crl
./easyrsa build-client-full sitio1 nopass
```

### 6.4 Problema: Conexión Lenta o Inestable

**Soluciones:**
```bash
# Ajustar MTU en configuración
echo "tun-mtu 1500" | sudo tee -a /etc/openvpn/server/server.conf
echo "fragment 1300" | sudo tee -a /etc/openvpn/server/server.conf
echo "mssfix 1250" | sudo tee -a /etc/openvpn/server/server.conf

# Deshabilitar compresión si causa problemas
# Comentar la línea "compress lz4-v2" en ambos lados

# Probar con TCP en lugar de UDP
# Cambiar "proto udp" por "proto tcp" (requiere cambios en firewall)
```

### 6.5 Verificar que CCD Funciona

```bash
# En el servidor, verificar que los archivos CCD se leen
sudo tail -f /var/log/openvpn/openvpn.log | grep iroute

# Debería mostrar algo como:
# ROUTE_GATEWAY 10.200.0.1/255.255.255.0 IFACE=tun0 HWADDR=...
# route_gateway -> 10.200.0.1
```

---

## FASE 7: Configuraciones Adicionales de Seguridad

### 7.1 Limitar Acceso por Firewall

En el servidor VPN:
```bash
# Permitir solo IPs específicas
sudo ufw delete allow 1194/udp
sudo ufw allow from 10.99.80.0/24 to any port 1194 proto udp
sudo ufw allow from 10.99.70.0/24 to any port 1194 proto udp
```

### 7.2 Configurar Logs Detallados

```bash
# En server.conf y client.conf
verb 4  # Mayor nivel de detalle

# Rotar logs
sudo nano /etc/logrotate.d/openvpn
```

Contenido:
```bash
/var/log/openvpn/*.log {
    weekly
    rotate 4
    missingok
    notifempty
    compress
    delaycompress
    sharedscripts
    postrotate
        systemctl reload openvpn-server@server || true
    endscript
}
```

### 7.3 Monitorización Automática

Script de monitoreo:
```bash
sudo nano /usr/local/bin/vpn-monitor.sh
```

Contenido:
```bash
#!/bin/bash
VPN_STATUS=$(systemctl is-active openvpn-server@server)

if [ "$VPN_STATUS" != "active" ]; then
    echo "OpenVPN server is down! Attempting restart..."
    systemctl restart openvpn-server@server
    echo "Alert sent at $(date)" | mail -s "VPN Server Down" admin@miempresa.com
fi
```

Hacer ejecutable y programar:
```bash
sudo chmod +x /usr/local/bin/vpn-monitor.sh
sudo crontab -e
```

Añadir:
```bash
*/5 * * * * /usr/local/bin/vpn-monitor.sh
```

---

## Resumen de Comandos Importantes

```bash
# Servidor VPN
sudo systemctl restart openvpn-server@server
sudo tail -f /var/log/openvpn/openvpn.log
sudo cat /var/log/openvpn/openvpn-status.log

# Clientes (r1 y r2)
sudo systemctl restart openvpn-client@sitio1
sudo journalctl -u openvpn-client@sitio1 -f
ip addr show tun0
ip route

# Pruebas
ping 10.60.60.2  # Desde Cliente1
ping 10.50.50.2  # Desde Cliente2
traceroute 10.60.60.2
```

---

## Referencias

- [Documentación oficial OpenVPN](https://openvpn.net/community-resources/)
- [OpenVPN Site-to-Site Tutorial](https://openvpn.net/as-docs/tutorials/tutorial--site-to-site-network.html)
- [pfSense OpenVPN Configuration](https://docs.netgate.com/pfsense/en/latest/recipes/openvpn-s2s-tls.html)
