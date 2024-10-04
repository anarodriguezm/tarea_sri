# tarea_sri
# Práctica 2: Instalación y configuración de servidor y cliente DHCP en Debian

### Escenario

En esta práctica vamos a instalar y configurar un servidor DHCP en Debian con dos clientes: uno Windows y otro Linux.
Las tres máquinas (el servidor Debian, el cliente Windows y el cliente Ubuntu)
estarán en la misma subred privada interna SRI2XX con la dirección de red **10.0.XX.0/24**.
Donde **XX** son los dos últimos dígitos de tu nombre de usuario del dominio.
Las máquinas estarán conectadas a Internet a través de un router **pfSense**.

### Preparación del entorno: Instalación y configuración de la red

1. **Creación de la red NAT SRI16**
   - Creamos la red NAT SRI16 con la IP **10.0.16.0/24** en VirtualBox.
   - Configuramos el router pfSense asignando la IP LAN **10.0.16.1**.
   
   ![Captura de configuración LAN](/fotossri/foto1.png)

2. **Configuración de los clientes**
   - Configuramos los clientes (Windows y Ubuntu) para obtener la IP automáticamente desde el servidor DHCP.
   - Verificamos la conectividad entre las máquinas usando `ping` para comprobar que pueden comunicarse entre sí y acceder a Internet.

   ![Captura de configuración en Windows](/fotossri/foto1.png)
   ![Captura de configuración en Ubuntu](/fotossri/foto1.png)

3. **Configuración de Debian**
   - Asignamos la IP estática **10.0.16.2** a la máquina Debian.
   
   ![Captura de configuración en Debian](/fotossri/foto1.png)

### Ejercicio 1: Configuración del servidor DHCP

1. **Instalación del servidor DHCP**
   - Nos conectamos como root:
     ```bash
     su -
     ```
   - Actualizamos el sistema e instalamos el servidor DHCP:
     ```bash
     sudo apt update
     sudo apt install isc-dhcp-server
     ```

2. **Configuración del archivo `dhcpd.conf`**
   - Editamos el archivo de configuración del DHCP:
     ```bash
     sudo nano /etc/dhcp/dhcpd.conf
     ```
   - Configuramos las IPs, el rango y el tiempo de arrendamiento (mínimo 15 días y máximo 30 días):
     ```bash
     subnet 10.0.16.0 netmask 255.255.255.0 {
         range 10.0.16.10 10.0.16.100;
         option routers 10.0.16.1;
         option domain-name-servers 8.8.8.8, 8.8.4.4;
         default-lease-time 1296000;  # 15 días
         max-lease-time 2592000;      # 30 días
     }

     host ubuntu-client {
         hardware ethernet [MAC de Ubuntu];
         fixed-address 10.0.16.60;
     }
     ```

3. **Configuración de la interfaz de red**
   - Editamos el archivo para especificar la interfaz de red que debe usar el servidor DHCP:
     ```bash
     sudo nano /etc/default/isc-dhcp-server
     ```
   - Configuramos la interfaz de red (`enp0s3`):
     ```plaintext
     INTERFACESv4="enp0s3"
     ```

4. **Verificación de la asignación de IPs**
   - Verificamos si las IPs se están asignando correctamente ejecutando:
     ```bash
     cat /var/lib/dhcp/dhcpd.leases
     ```

5. **Capturas de la conexión de clientes**
   - Verificamos la conectividad entre los clientes Windows y Ubuntu, confirmando que tienen conexión a Internet.
   
   ![Captura de cliente Windows](/fotossri/foto1.png)
   ![Captura de cliente Ubuntu](/fotossri/foto1.png)

6. **Verificación del servicio DHCP**
   - Finalmente, verificamos que el servicio DHCP está activo y funcionando:
     ```bash
     sudo systemctl status isc-dhcp-server
     ```

### Ejercicio 3: Funcionamiento del servicio

1. **¿Qué es `journalctl`?**
   - `journalctl` es una herramienta en sistemas Linux que permite ver y filtrar los registros de eventos del sistema.

2. **Ver los logs del servidor DHCP**
   - Para ver los logs generados por el servidor **isc-dhcp-server**, usamos el siguiente comando:
     ```bash
     sudo journalctl -u isc-dhcp-server
     ```

3. **Filtrar por un intervalo de tiempo específico**
   - Para ver los logs generados entre un rango de tiempo, por ejemplo, desde ayer a las 14:00 hasta hoy a las 12:00, usamos:
     ```bash
     sudo journalctl -u isc-dhcp-server --since "2024-10-03 14:00:00" --until "2024-10-04 12:00:00"
     ```

4. **Filtrar los registros relacionados con la asignación de IPs**
   - Para ver solo los eventos relacionados con la asignación de direcciones IP:
     ```bash
     sudo journalctl -u isc-dhcp-server | grep DHCPACK
     ```

5. **Explicación de los mensajes clave en los logs**
   - **DHCPDISCOVER**: El cliente DHCP envía una solicitud para descubrir un servidor DHCP en la red.
   - **DHCPOFFER**: El servidor DHCP ofrece una dirección IP al cliente.
   - **DHCPREQUEST**: El cliente solicita la dirección IP ofrecida.
   - **DHCPACK**: El servidor confirma la asignación de la IP al cliente.
   - **DHCPRELEASE**: El cliente libera la IP asignada.

---

### **Conclusión**

Este documento explica los pasos para configurar un servidor DHCP en Debian y cómo verificar el correcto funcionamiento del servidor a través de `journalctl`.

---

