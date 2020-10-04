# Practica 1.1: Protocolo IPv4. Servicio DHCP

# Objetivos
En esta práctica se presentan las herramientas que se utilizarán en la asignatura y se repasan brevemente los aspectos básicos del protocolo IPv4. Además, se analizan las características del protocolo DHCP.

# Preparación del entorno
|Maquina| IP | Comentarios
|--|--|--|
| VM1 | 192.168.0.1/24 (eth0)|Añadir router como encaminador por defecto |
| VM2 | 192.168.0.2/24 (eth0)| Añadir router como encaminador por defecto|
| VM3 - Router | 192.168.0.3/24 (eth0) 172.16.0.3/16 (eth1)| Añadir forwarding de paquetes |
| VM4 | 172.16.0.1/16 (eth0)| Añadir router como encaminador por defecto |

 1. Definir la máquina base de la asignatura

```bash
$sudo asorregenarte
```
 2. Crear un archivo pr1.topol con la topología de la red, que consta de 4 máquinas y dos redes. El contenido del fichero debe ser:
```c
netprefix inet
machine 1 0 0
machine 2 0 0
machine 3 0 0 1 1
machine 4 0 1
```
La sintaxis es:

```bash
machine <nº maquina> <interfaz n> <red conexion n>
```
 - Crear la topología de red que arrancará las 4 máquinas virtuales (VM1, VM2, Router y VM4).
```bash
$sudo vtopol pr1.topol
```
3. Si la configuración es manual, todas las maquinas clonadas deben ser conectadas a la Red Interna, y cada interfaz configurada como:

|Maquina | Red Interna 0 | Red Interna 1 |
|--|--|--|
| VM1 | eth0 - intnet 0 
| VM2 | eth0 - intnet 0  
| VM3 - Router | eth0 - intnet 0 | eth1 - intnet 1
| VM4 | eth0 - intnet 1
# Configuración estática
En primer lugar, configuraremos cada segmento de red 172.16.0.0/16 y 192.168.0.0/24 de forma estática asignando a cada máquina una dirección IP adecuada.

### Ejercicio 1 [VM1]
Determinar los interfaces de red que tiene la máquina y las direcciones IP y/o MAC que tienen asignadas. Utilizar el comando ip.
```bash
$ip link # interfaces
$ip a # dir IP asociadas a las interfaces
```
Para asociar una dirección a una interfaz (eth0):
```bash
$sudo ip address add 192.168.0.1/24 dev eth0
$ip link set dev eth0 up
$ip a
```
### Ejercicio 2 [VM1, VM2, Router]
Activar los interfaces eth0 en las máquinas VM1, VM2 y Router, y asignar una dirección IP adecuada. La configuración debe realizarse con la utilidad ip, en particular los comandos ip address e ip link.
```bash
$sudo ip address add 192.168.0.2/24 dev eth0
$ip link set dev eth0 up
$ip a
```
### Ejercicio 3 [VM1, VM2]
Arrancar la herramienta wireshark y activar la captura en el interfaz de red. Comprobar la conectividad entre VM1 y VM2 con la orden ping. Observar el tráfico generado, especialmente los protocolos encapsulados en cada datagrama y las direcciones origen y destino.
#### VM2
```bash
$ping 192.168.0.1
$ping 192.168.0.1 -I eth0
```
Completar la siguiente tabla para todos los mensajes intercambiados hasta la recepción de la primera respuesta Echo Reply:
- Anotar las direcciones MAC e IP de los mensajes.

- Para cada protocolo, anotar las características importantes (p. ej. pregunta/respuesta ARP o tipo ICMP) en el campo “Tipo de Mensaje”.

- Comparar los datos observados durante la captura con el formato de los mensajes estudiados en clase.
#### Wireshark VM2

[img ejercicio 3](https://drive.google.com/file/d/103k5OHxirpA2_YGvVhehy3nUPbnpUUm1/view?usp=sharing)

| MAC Origen| MAC Destino | Protocolo | IP Origen | IP Destino | Tipo Mensaje
|--|--|--|--|--|--|
| 08:00:27:02:e4:c5 | 00:00:00:00:00:00 | ARP | 192.168.0.2 | ff:ff:ff:ff:ff:ff | Who has 192.168.0.1? Tell 192.168.0.2 |
| 08:00:27:02:e4:c5 | 08:00:27:4c:af:03 | ARP | 192.168.0.1 | 192.168.0.2 | 192.168.0.1 is at 08:00:27:4c:af:03 |
|  |  | CMP | 192.168.0.2 | 192.168.0.1 | Echo (ping) request |
|  |  | CMP | 192.168.0.1 | 192.168.0.2 | Echo (ping) reply |
----
#### CMP
El protocolo CMP **(Internet control message protocol)** para IPv4 es el protocolo de intercambio de mensajes, los tipos que tenemos son request (type = 0) y reply (type = 8), el código de ambos es 0 como dicen las transparencias de clase. La primera secuencia del mensaje request es (BE) 0x0aad / (LE) 0xad0a; la primera secuencia del mensaje reply es (BE) 0x0001 / (LE) 0x0100.
#### ARP
EL protocol ARP **(Address Resolution Protocol)** para IPv4 permite la resolución de direcciones MAC. Se encarga la máquina que quiere solicitar una dirección MAC de preguntar a través de broadcast a quien le pertenece la dirección IP que desea conocer la dir MAC. Para ello, debe existir una dirección MAC e IP destino, que en nuestro caso son primero broadcast ambas; y la del remitente en nuestro caso la Máquina  2 que tiene como dirección IP 192.168.0.2, como dirección MAC 08:00:27:92:d9:1b.
Si no existe ningún error ,la máquina VM1, con dirección IP 192.168.0.1 contesta con su dirección MAC 08:00:27:4c:af:03.

----
### Ejercicio 4 [VM1, VM2 ]
Ejecutar de nuevo la orden ping entre VM1 y VM2 y, a continuación, comprobar el estado de la tabla ARP en VM1 y VM2 usando el comando ip neigh. El significado del estado de cada entrada de la tabla se puede consultar en la página de manual del comando.

----

En el caso de ejecutar otra vez ping al principio de la ejecución no se utiliza el protocolo ARP para averiguar de quien es la dirección MAC asociada a la dirección IP 192.168.0.1. Sólo se repite la pregunta cada x milisegundos para comprobar si esta siguen asociadas (MAC/IP).

----

```bash
man ip-neighbour
$ip neigh #ver dir ip con las que se ha conectado la maq
$ip neigh flush dev eth0 # eliminar los valores de la tabla
```
#### VM2
[imagen ejercicio 4](https://drive.google.com/file/d/19DsHsdHZ6D_aLFCksD25oWAadrMdGVNS/view?usp=sharing) 
| address | dev name  | vrf name | proxy | unused | nud
|--|--|--|--|--|--|
| 192.168.0.1| eth0 | lladdr | 08:00:27:4c:af:03 |  | STALE |

### Ejercicio 5 [Router, VM4]
Repetir la configuración de red para el segmento 192.168.0.0/24. Comprobar la conectividad entre Router y VM4; y entre Router, VM1 y VM2.

#### VM3
```bash
$sudo ip address add 192.168.0.3/24 dev eth0
$ip link set dev eth0 up
$sudoip address add 172.16.0.3/16 dev eth1
$ip link set dev eth1 up
$ip a
```

#### VM4
```bash
$sudoip address add 172.16.0.1/16 dev eth0
$ip link set dev eth0 up
$ip a
$ping 172.16.0.3
```
[imagen ejercicio 5](https://drive.google.com/file/d/1-DJSuuqz5VPvuVebdewE5rY6A_eE-YNh/view?usp=sharing) 

| MAC Origen| MAC Destino | Protocolo | IP Origen | IP Destino | Tipo Mensaje
|--|--|--|--|--|--|
| 08:00:27:08:62:ca | 00:00:00:00:00:00 | ARP | 172.16.0.1 | ff:ff:ff:ff:ff:ff | Who has 172.16.0.3? Tell 172.16.0.1 |
| 08:00:27:92:c3:f1 | 08:00:27:08:62:ca | ARP | 172.16.0.3 | 172.16.0.1 | 172.16.0.3 is at ?? 08:00:27:92:c3:f1 |
|  |  | CMP | 172.16.0.3 | 172.16.0.1 | Echo (ping) request |
|  |  | CMP | 172.16.0.1 | 172.16.0.3 | Echo (ping) reply |

```bash
$ip neigh
```
#### VM4
| address | dev name  | vrf name | proxy | unused | nud
|--|--|--|--|--|--|
| 172.16.0.3 | eth0 | lladdr | 08:00:27:92:c3:f1 |  | STALE |

# Encaminamiento estático
Según la topología de esta práctica, Router puede encaminar el tráfico entre ambas redes. En esta sección, vamos a configurar el encaminamiento estático, basado en rutas que fijaremos manualmente en todas las máquinas virtuales.

### Ejercicio 6 [Router]
Activar el reenvío de paquetes (​forwarding​) en Router para que efectivamente pueda funcionar como encaminador entre las redes. Ejecutar el siguiente comando:
#### VM3
```bash
$sudo sysctl net.ipv4.ip_forward=1
```
### Ejercicio 7 [VM1, VM2]
Añadir Router como encaminador por defecto para VM1 y VM2. Usar el comando​ ip route​.
```bash
$sudo ip route add default via 192.168.0.3
```

### Ejercicio 8 [VM4]
Aunque la configuración adecuada para la tabla de rutas en redes como las consideradas en esta práctica consiste en añadir una ruta por defecto, es posible incluir rutas para redes concretas. Añadir en VM4 una ruta a la red 192.168.0.0/24 vía Router. Usar el comando ip route​.
```bash
$sudo ip route add default via 172.16.0.3
```

### Ejercicio 9 [VM1, VM4]
Abrir la herramienta Wireshark en Router e iniciar una captura en sus dos interfaces de red. Eliminar la tabla ARP en VM1 y Router. Usar la orden ​ping entre VM1 y VM4. Completar la siguiente tabla para todos los paquetes intercambiados hasta la recepción del primer Echo Reply​.

#### VM1
```bash
$ping 172.16.0.3
```

#### Wireshark VM1
[imagen ejercicio 9-1](https://drive.google.com/file/d/19_Px_1LqHSlhPFJF8MyeHkOs7yueStlN/view?usp=sharing) 
#### Red 192.168.0.0/24 - Router (eth0)
| MAC Origen | MAC Desitno | Protocolo | IP Origen | IP Destino | Tipo Mensaje |
|--|--|--|--|--|--|--|--|
| 08:00:27:3b:2f:15 | 00:00:00:00:00:00 | ARP | 192.168.0.1 | ff:ff:ff:ff:ff:ff | Who has 192.168.0.1? Tell 192.168.0.3 |
| 08:00:27:3b:2f:15 | 08:00:27:4c:af:03 | ARP | 172.16.0.3 | 172.16.0.1 | 172.16.0.3 is at ?? 08:00:27:3b:2f:15 |
|  |  | CMP | 192.168.0.1 | 172.16.0.1 | Echo (ping) request |
|  |  | CMP | 192.168.0.1 | 192.168.0.1 | Echo (ping) reply |
#### Wireshark VM4
[imagen ejercicio 9-2](https://drive.google.com/file/d/1GtlbBukYXfxluY-p1bp-5y_BIbTdlYMS/view?usp=sharing) 
#### Red 172.16.0.0/16- Router (eth1)
| MAC Origen | MAC Desitno | Protocolo | IP Origen | IP Destino | Tipo Mensaje |
|--|--|--|--|--|--|--|--|
| 08:00:27:92:c3:f1 | 00:00:00:00:00:00 | ARP | 172.16.0.3 | ff:ff:ff:ff:ff:ff | Who has 172.16.0.1? Tell 172.16.0.3 |
| 08:00:27:08:62:ca | 08:00:27:92:c3:f1 | ARP | 172.16.0.1 | 172.16.0.3 | 172.16.0.1 is at ?? 08:00:27:3b:2f:15 |
|  |  | CMP | 192.168.0.1 | 172.16.0.1 | Echo (ping) request |
|  |  | CMP | 172.16.0.1 | 192.168.0.1 | Echo (ping) reply | 

# Configuración dinámica de hosts
El protocolo DHCP permite configurar dinámicamente los parámetros de red de una máquina. En esta sección configuraremos Router como servidor DHCP para las dos redes. Aunque DHCP puede incluir muchos parámetros de configuración, en esta práctica sólo fijaremos el encaminador por defecto.

### Ejercicio 10 [VM1, VM2, VM4]
Eliminar las direcciones IP de los interfaces de todas las máquinas salvo Router.
#### VM1
```bash
$sudo ip address del 192.168.0.1/24 dev eth0
```
#### VM2
```bash
$sudo ip address del 192.168.0.2/24 dev eth0
```
#### VM4
```bash
$sudo ip address del 172.16.0.1/16 dev eth0
```

### Ejercicio 11 [Router]
​Configurar el servidor DHCP para las dos redes​:
-  Editar el fichero ​/etc/dhcp/dhcpd.conf y añadir dos secciones ​subnet​, una para cada red, que definan los rangos de direcciones​, 1​92.168.0.50-192.168.0.100 y 172.16.0.50-172.16.0.100​, respectivamente. Además, incluir la opción ​routers con la dirección IP de Router en cada red. Ejemplo:
```bash
$sudo nano /etc/dhcp/dhcpd.conf
```
----
subnet 192.168.0.0 netmask 255.255.255.0 { 
		range 1​92.168.0.50 192.168.0.100; 
		option routers 192.168.0.3;  
	option broadcast-address 192.168.0.255;
}

subnet 172.16.0.0 netmask 255.255.0.0 { 
		range 172.16.0.50 172.16.0.100; 
		option routers 172.16.0.3;  
	option broadcast-address 172.16.0.255;
}

----
- Arrancar el servicio con el comando ``` service dhcpd start ```
```bash
$sudo service dhcpd start
```

### Ejercicio 12 [Router, VM4]
Iniciar la captura de paquetes en Router. Arrancar el cliente DHCP ``` dhclient -d eth0 ``` en la máquina virtual VM1 y observar el proceso de configuración. Completar la siguiente tabla:
#### VM1
```bash
$sudo dhclient -d eth0
```
[imagen configuracion mv1](https://drive.google.com/file/d/1ALGzaoGJp1D4-W3zPbrmsn57fq4Ewb-F/view?usp=sharing)
[imagen wireshark vm1](https://drive.google.com/file/d/1CxT92sswxSG-w4RugChH84YQKemi8QdR/view?usp=sharing)

#### VM4
```bash
$sudo dhclient -d eth0
```
[imagen vm4 dhcp client](https://drive.google.com/file/d/1mhQcZfeew9DgSLbzPcPWk-E6Oa4MdHy8/view?usp=sharing)

#### VM1
| IP Origen | IP Destino | Mensaje DHCP | Opciones DHCP |
|--|--|--|--|
| 0.0.0.0 | 255.255.255.255 | DHCP Request | (53) DHCP Message Type (50) Requested IP address (55) Parameter Request List (255) End |
| 192.168.0.3 | 192.168.0.50 | DHCP ACK | (53) DHCP Message Type (54) DHCP Server Identifier (51) IP address Lease Time (1) Subnet Mask (28) Broadcast Address (3) Router (255) End|

#### VM3
[imagen wireshark vm3](https://drive.google.com/file/d/1zmHyUqbRz9iLFEUCZwmMtF_SB_LB_fJI/view?usp=sharing)
| IP Origen | IP Destino | Mensaje DHCP | Opciones DHCP |
|--|--|--|--|
| 0.0.0.0 | 255.255.255.255 | DHCP Discover | ---- |
| 192.168.0.3 | 192.168.0.50 | DHCP Offer | ---- |
| 0.0.0.0 | 255.255.255.255 | DHCP Request | ---- |
| 192.168.0.3 | 192.168.0.50 | DHCP ACK | ---- |

### Ejercicio 13 [VM4]
Durante el arranque del sistema se pueden configurar automáticamente determinados interfaces según la información almacenada en el disco del servidor. Consultar el fichero /etc/sysconfig/network-scripts/ifcfg-eth0 de VM4, que configura automáticamente eth0 usando DHCP. Para más información, consultar el fichero /usr/share/doc/initscripts-*/sysconfig.txt.

```bash
​$sudo nano /etc/sysconfig/network-scripts/ifcfg-eth0
```
----
TYPE=Ethernet  
BOOTPROTO=none  
IPADDR=​172.16.0.1/16
GATEWAY=​172.16.0.3/16
DEVICE=eth0

----

**Nota**:​ Estas opciones se describen en detalle en ​/usr/share/doc/initscripts-*/sysconfig.txt​.

### Ejercicio 14 [VM4]
Comprobar la configuración automática con las órdenes ifup e ifdown. Verificar la conectividad entre todas las máquinas de las dos redes.

```bash
$sudo ifdown eth0
$sudo ifup eth0
$ip a
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwNzE3MTIyODIsLTcyNTUxNzc1NCwtMT
k4MDk0Mzk5NywtMTQ3NDc5NTA5OSw5MzU4MTI3NjQsNTQyODM3
OTQ3LDgyMDQ4NzEzMCwtNjMwNjI3NTc3LDEyNTY2MDc5NDIsLT
E3OTExOTAwNSwxMTY0NTk3MDMzLC0yMDMzMTI2Njk1LC01MTQ2
NjY4NDgsNzY0NjEyNjQ4LC02ODE1Nzg5NjgsLTIwMTU5NDY2Mz
IsLTcwNzc3MTE0OCw3NDAwMzM5NzQsMTEzODkyMzg3OCwtMzAy
NjcwOTM4XX0=
-->