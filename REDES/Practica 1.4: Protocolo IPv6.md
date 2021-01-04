# Practica 1.4: Protocolo IPv6

# Objetivos
En esta práctica se estudian los aspectos básicos del protocolo IPv6, el manejo de los diferentes tipos de direcciones y mecanismos de configuración. Además se analizarán las características más importantes del protocolo ICMP versión 6.

# Preparación del entorno para la práctica

```c
netprefix inet
machine 1 0 0
machine 2 0 0
machine 3 0 0 1 1
machine 4 0 1
```

# Direcciones de enlace local
Una dirección de enlace local es únicamente válida en la subred que está definida. Ningún encaminador dará salida a un datagrama con una dirección de enlace local como destino. El prefijo de formato para estas direcciones es fe80::/10.

### Ejercicio 1[VM1, VM2]
Activar el interfaz eth0 en VM1 y VM2. Comprobar las direcciones de enlace local que tienen asignadas con el comando ip.
```bash 
sudo ip link set eth0 up 
ip a
ip m
```
[dir IPv6 VM1](https://drive.google.com/file/d/1eRXcfnL40LR1SbolcfRMzShjMXU5n4jN/view?usp=sharing)
### Ejercicio 2[VM1, VM2]
Comprobar la conectividad entre VM1 y VM2 con la orden ping6. Cuando se usan direcciones de enlace local, y  **sólo en ese caso**, es necesario especificar el interfaz origen, añadiendo %<nombre_interfaz> a la dirección. Consultar las opciones del comando ping6 en la página de manual. Observar el tráfico generado con la herramienta wireshark, especialmente los protocolos encapsulados en cada datagrama y los parámetros del protocolo IPv6.
```bash 
sudo ping6 fe80::a00:27ff:fe02:e4c5 -I eth0 # vm1
```
Los paquete enviados entre la máquina 1 y la máquina 2 fueron: Echo request/reply y Neighbor Solicitation/Advertisiment. Las opciones de ping6 son: 
```bash
man ping6
```
 - -6: Uso IPv6
 - -c: number of echo request packets enviados
 - -I: interfaz 
[ping6 vm1 to vm2](https://drive.google.com/file/d/1d3uCNSL8D88yNmS9T1Ch014DM74-LYsT/view?usp=sharing)
### Ejercicio 3[Router, VM4]
Activar el interfaz de VM4 y los dos interfaces de Router. Comprobar la conectividad entre Router y VM1, y entre Router y VM4 usando la dirección de enlace local.
```bash 
sudo ip link set eth0 up 
sudo ip link set eth1 up # sólo en el router(vm3)
ip a
ip m
```
[IPv6 VM4-VM3](https://drive.google.com/file/d/1acMNcuY5TSsITx8MFfZuFvODQ96eWFln/view?usp=sharing)
Para comprobar la conectividad VM1-Router:
```bash 
sudo ping6 fe80::a00:27ff:fe3b:2f15 -I eth0 # vm1
```
[ping6 VM1 to VM3](https://drive.google.com/file/d/1XQD13-c8ouHiGFIieeHaqW_YxFxJ9ULw/view?usp=sharing)
Para comprobar la conectividad VM4-Router:
```bash 
sudo ping6 fe80::a00:27ff:fe92:c3f1 -I eth0 # vm4
```
[ping6 VM4 to VM3](https://drive.google.com/file/d/1X0ol-nDlWvDsNXo4zwSefq8nHGnn-tPw/view?usp=sharing)
# Direcciones ULA
Una dirección ULA (_Unique Local Address_) puede usarse dentro de una organización, de forma que los encaminadores internos del sitio deben encaminar los datagramas con una dirección ULA como destino. El prefijo de formato para estas direcciones es fc00::/7.
### Ejercicio 4[VM1, VM2]
Configurar VM1 y VM2 para que tengan una dirección ULA en la red fd00:0:0:a::/64 con el comando ip. La parte de identificador de interfaz puede elegirse libremente, siempre que no coincida para ambas máquinas.  **Nota:** Incluir la longitud del prefijo al fijar las direcciones.
Para asignar a la maquina virtual 1:
```bash 
sudo ip link set eth0 down
sudo ip address add fd00:0:0:a::1/64 dev eth0
sudo ip link set eth0 up
```
Para asignar a la maquina virtual 2:
```bash 
sudo ip link set eth0 down
sudo ip address add fd00:0:0:a::2/64 dev eth0
sudo ip link set eth0 up
```
[dir IPv6 VM1](https://drive.google.com/file/d/1V8O5s95A795uUNt-6oQkBgN81he4Ohgu/view?usp=sharing)
### Ejercicio 5[VM1, VM2]
Comprobar la conectividad entre VM1 y VM2 con la orden ping6 usando la nueva dirección. Observar los mensajes intercambiados con wireshark.
```bash
ping6 fd00:0:0:a::2%eth0 # mv1 o -I eth0
```
[wiresark](https://drive.google.com/file/d/1DV5LR_FRebp2Egl7GnCeqxFZ4DNgAdvO/view?usp=sharing)
### Ejercicio 6[Router, VM4]
Configurar direcciones ULA en los dos interfaces de Router (redes fd00:0:0:a::/64 y fd00:0:0:b::/64) y en el de VM4 (red fd00:0:0:b::/64). Elegir el identificador de interfaz de forma que no coincida dentro de la misma red.
Para asignar a la maquina virtual 3/router:
```bash 
sudo ip link set eth0 down
sudo ip link set eth1 down
sudo ip address add fd00:0:0:a::3/64 dev eth0
sudo ip address add fd00:0:0:b::3/64 dev eth1
sudo ip link set eth0 up
sudo ip link set eth1 up
```
[IPv6 VM1](https://drive.google.com/file/d/1JQ5cNIDREpRTjX8QKKetDpCuu2Xt6ij5/view?usp=sharing)
Para asignar a la maquina virtual 4:
```bash 
sudo ip link set eth0 down
sudo ip address add fd00:0:0:b::1/64 dev eth0
sudo ip link set eth0 up
```
[IPv6 VM4](https://drive.google.com/file/d/1LDxQIf6UmtL1sU6e59LaGr5inE3sXEzg/view?usp=sharing)
[wiresark VM1-VM4](https://drive.google.com/file/d/1X88U3YPBcQDEf-k0hTnPWMJowuxpjdlT/view?usp=sharing)
### Ejercicio 7[Router]
Comprobar la conectividad entre Router y VM1, y entre Router y VM4 usando direcciones ULA. Comprobar además que VM1 no puede alcanzar a VM4.
Entre VM1-Router:
```bash
ping6 fd00:0:0:a::3%eth0 # mv1
```
[VM1 to VM3](https://drive.google.com/file/d/1bWUWnutD-6Vay9d4fVD7amAfr4S_5UAb/view?usp=sharing)
Entre VM4-Router:
```bash
ping6 fd00:0:0:b::3%eth0 # mv4
```
[VM4 to VM3](https://drive.google.com/file/d/123aAF-b5_LTtYhf9J8Br0NzB6wnWeMKt/view?usp=sharing)
Entre VM1-VM4:
```bash
ping6 fd00:0:0:b::1%eth0 # mv1
```
[VM1 to VM4](https://drive.google.com/file/d/1fwiuZZi1HPwYd2EMBfa2lhJw96bGFrQ0/view?usp=sharing)
# Encaminamiento estático
Según la topología que hemos configurado en esta práctica, Router debe encaminar el tráfico entre las redes fd00:0:0:a::/64 y fd00:0:0:b::/64. En esta sección vamos a configurar un encaminamiento estático basado en las rutas que fijaremos manualmente en todas las máquinas.

### Ejercicio 8[Router, VM1]
Consultar las tablas de rutas en VM1 y Router con el comando ip route. Consultar la página de manual del comando para seleccionar las rutas IPv6.
```bash
sudo ip -6 route
```
Leyendo el manual la opción -6 indica que queremos ver direcciones IP versión 6.
[resultado](https://drive.google.com/file/d/1xit8SeAAUSEK9COwieduhzilq_63c42J/view?usp=sharing)

### Ejercicio 9[Router]
Para que Router actúe efectivamente como encaminador, hay que activar el reenvío de paquetes (_packet forwarding_). De forma temporal, se puede activar con el comando ```sysctl -w net.ipv6.conf.all.forwarding=1```.
```bash
sudo sysctl -w net.ipv6.conf.all.forwarding=1 
```

### Ejercicio 10[Router, VM4, VM1, VM2]
Finalmente, hay que configurar la tabla de rutas en las máquinas virtuales. Añadir la dirección correspondiente de Router como ruta por defecto con el comando ip route. Comprobar la conectividad entre VM1 y VM4 usando el comando ping6.
En las máquinas virtuales VM1 y VM4.
```bash
sudo ip r add default via fd00:0:0:b::3
```

### Ejercicio 11[Router, VM4, VM1]
Borrar la  _cache_  de vecinos en VM1 con ip neigh flush dev eth0 y en Router con ip neigh flush dev eth1. Con ayuda de la herramienta wireshark, completar la siguiente tabla al usar la orden ping6 entre VM1 y VM4 con todos los mensajes hasta el primer ICMP Echo Reply:

#### Red fd00:0:0:a::/64 - VM1
|  MAC Origen| MAC Destino | IPv6 Origen | IPv6 Destino | ICMPv6 Tipo
|--|--|--|--|--|
| MAC<sub>VM1</sub> | Broadcast  | fd00:0:0:a::1| fd02:1:ff00:3 | Neighbor Solicitation |
| MAC<sub>VM3:eth0</sub> | MAC<sub>VM1</sub> | fd00:0:0:a::3 | fd00:0:0:a::1 | Neihbor Advertisement |
| MAC<sub>VM1</sub> | MAC<sub>VM3:eth0</sub>  | fd00:0:0:a::1| fd00:0:0:a::1 | Reply |
| MAC<sub>VM3:eth0</sub> | MAC<sub>VM1</sub> | fd00:0:0:a::3 | fd00:0:0:a::1 | Request |

[wiresahark VM1](https://drive.google.com/file/d/19RArwth1JMJ-IyJu_wcS-u7cDYwiungt/view?usp=sharing)
#### Red fd00:0:0:b::/64 - VM4
|  MAC Origen| MAC Destino | IPv6 Origen | IPv6 Destino | ICMPv6 Tipo |
|--|--|--|--|--|
| MAC<sub>VM4</sub> | Broadcast  | fd00:0:0:a::1| fd02:1:ff00:1 | Neighbor Solicitation |
| MAC<sub>VM3:eth1</sub> | MAC<sub>VM4</sub> | fd00:0:0:a::3 | fd00:0:0:a::1 | Neihbor Advertisement |
| MAC<sub>VM4</sub> | MAC<sub>VM3:eth1</sub>  | fd00:0:0:a::1| fd00:0:0:a::1 | Reply |
| MAC<sub>VM3:eth1</sub> | MAC<sub>VM1</sub> | fd00:0:0:a::3 | fd00:0:0:a::1 | Request |

[wiresahark VM4](https://drive.google.com/file/d/1fVxhUGJqU-HZtgAYZU8PG-qCYemMwrzL/view?usp=sharing)

# Configuración persistente
Las configuraciones realizadas en los apartados anteriores son volátiles y desaparecen cuando se reinician las máquinas. Durante el arranque del sistema se pueden configurar automáticamente determinados interfaces según la información almacenada en el disco.

### Ejercicio 12[Router]
Crear los ficheros ifcfg-eth0 e ifcfg-eth1 en el directorio /etc/sysconfig/network-scripts/ con la configuración de cada interfaz. Usar las siguientes opciones:
|  IPv6 | IPv4 |
|--|--|
| Type=Ethernet BOOTPROTO=static IPV6ADDR=_<dirección IP en formato CIDR>_  IPV6_DEFAULTGW=_<dirección IP del encaminador por defecto (si existe)>_  DEVICE=_<nombre del interfaz (eth0...)> | Type=Ethernet BOOTPROTO=static IPADDR=_<dirección IP en formato CIDR>_ GATEWAY=_<dirección IP del encaminador por defecto (si existe)>_ DEVICE=_<nombre del interfaz (eth0...)> |


[fichero ifcfg eth0](https://drive.google.com/file/d/1PRWjM8pX4o4spXI-4qGpqyDxGfhPhIcF/view?usp=sharing)
[fichero ifcfg eth1](https://drive.google.com/file/d/1rOYTw2kMg4xF3qmAend6BuQE9IKnddZg/view?usp=sharing)

### Ejercicio 13[Router]
Comprobar la configuración automática con las órdenes ifup e ifdown.
```bash
sudo ifdown
```


[resultado del ping](https://drive.google.com/file/d/1C3cBo6C9FlKs7ZTDSSsHFSiRkdTW0LYv/view?usp=sharing)
```bash
sudo ifup
```
# Autoconfiguración. Anuncio de prefijos
El protocolo de descubrimiento de vecinos se usa también para la autoconfiguración de los interfaces de red. Cuando se activa un interfaz, se envía un mensaje de descubrimiento de encaminadores. Los encaminadores presentes responden con un anuncio que contiene, entre otros, el prefijo de la red.

### Ejercicio 14[VM1, VM2, VM4]
Eliminar las direcciones ULA de los interfaces desactivándolos con ip.
#### VM1-VM4
```bash
sudo link set dev eth0 down
```
#### VM2
```bash
sudo link set dev eth1 down
```

[resultado ejecución VM1](https://drive.google.com/file/d/1Bo46e9d0tkNsxWOKe4xYrd93D5PlskXB/view?usp=sharing)
### Ejercicio 15[Router]
Configurar el servicio zebra para que el encaminador anuncie prefijos. Para ello, crear el archivo /etc/quagga/zebra.conf e incluir la información de los prefijos para las dos redes. Cada entrada será de la forma:
```bash
sudo nano /etc/quagga/zebra.conf 
```
Insertar el siguiente código en c.
```c
interface eth0
	no ipv6 nd suppress-ra
	ipv6 nd prefix fd00:0:0:a::/64
```

Finalmente, arrancar el servicio con el comando service zebra start.
```bash
sudo service zebra start
```
### Ejercicio 16[VM4]
Comprobar la autoconfiguración del interfaz de red en VM4, volviendo a activar el interfaz y consultando la dirección asignada.
#### VM4
```bash
ip link set eth0 up
ip a
```
[resultado en VM4](https://drive.google.com/file/d/19bE-WkqyqazJlKWkRXZflFYYevsPZ3FI/view?usp=sharing)
### Ejercicio 17[VM1, VM2]
Estudiar los mensajes del protocolo de descubrimiento de vecinos:
- Activar el interfaz en VM2, comprobar que está configurado correctamente e iniciar una captura de tráfico con wireshark.
- Activar el interfaz en VM1 y estudiar los mensajes ICMP de tipo Router Solicitation y Router Advertisement.
- Comprobar las direcciones destino y origen de los datagramas, así como las direcciones destino y origen de la trama Ethernet. Especialmente la relación entre las direcciones IP y MAC. Estudiar la salida del comando ip maddr.
#### VM1
```bash
ip link set eth0 up
ip a
```
#### VM2
```bash
ip link set eth1 up
ip a
ip m
```
[wireshark VM2](https://drive.google.com/file/d/1tAonqUteU-H6AUaChkoRKjgQcJvxegjm/view?usp=sharing)

### Ejercicio 18[VM1]
La generación del identificador de interfaz mediante EUI-64 supone un problema de privacidad para las máquinas clientes, que pueden ser rastreadas por su dirección MAC. En estos casos, es conveniente activar las extensiones de privacidad para generar un identificador de interfaz pseudoaleatorio temporal para las direcciones globales. Activar las extensiones de privacidad en VM1 con sysctl -w net.ipv6.conf.eth0.use_tempaddr=2.
```bash
sudo sysctl -w net.ipv6.conf.eth0.use_tempaddr=2
```
[wireshark VM2](https://drive.google.com/file/d/1orupPgcHWIJN5XuNDhVnXjXcHlVhz3eK/view?usp=sharing)
# ICMPv6
El protocolo ICMPv6 permite el intercambio de mensajes para el control de la red, tanto para la detección de errores como para la consulta de la configuración de ésta. Durante el desarrollo de la práctica hemos visto los más importantes.
### Ejercicio 19
Generar mensajes de los siguientes tipos en la red y estudiarlos con ayuda de la herramienta wireshark:
- [Solicitud y respuesta de eco](https://drive.google.com/file/d/1NjmWExbSz-L2k_kqQx3PvIP6lWEviXu3/view?usp=sharing)

- [Solicitud y anuncio de encaminador](https://drive.google.com/file/d/1NLbvYGjEiCVdv-LemkXU3XlJUYpwqfIn/view?usp=sharing)

- [Solicitud y anuncio de vecino](https://drive.google.com/file/d/1Zc8fLj2FmyEBTHx9RIYEvRolWZIw1sCX/view?usp=sharing)

- [Destino inalcanzable - Sin ruta al destino (Code: 0)](https://drive.google.com/file/d/1AbG3X4Ggn8Hi-ZpGisahy-iZtW9RNsmi/view?usp=sharing)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwODQ5MzE3NzksLTYyODYzMzg3MCwzMD
AwODM1MTIsLTY3NDIxNzYwOCwtNDU1ODAzMzk2LDc3NjM2Mzc3
NSwtMjAxNjg5MDg4NywtMTI3NTk5MTYyNywtMTM1OTkwMDA3Ni
wtMzM2MzgwNzIyLC00OTc1NzMzNTNdfQ==
-->