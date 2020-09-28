# Practica 1.2: Conceptos Avanzados de TCP

# Objetivos
En esta práctica estudiaremos el funcionamiento del protocolo TCP. Además veremos algunos parámetros que permiten ajustar el comportamiento de las aplicaciones TCP. Finalmente se consideran algunas aplicaciones del filtrado de paquetes mediante iptables.

# Preparación del entorno

 1. Definir la máquina base de la asignatura

```c
asorregenarte
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

```c
machine <nº maquina> <interfaz n> <red conexion n>
```
 - Crear la topología de red que arrancará las 4 máquinas virtuales (VM1, VM2, Router y VM4).
```c
vtopol <pr2>.topol
```
La tabla correspondiente:
|Maquina| IP | Comentarios
|--|--|--|
| VM1 | 192.168.0.1/24 |Añadir router como encaminado por defecto  |
| VM2 | 192.168.0.2/24 | Añadir router como encaminado por defecto |
| VM3 - Router | 192.168.0.3/24 (eth0) 172.16.0.3/24 (eth1)| Añadir forwarding de paquetes |
| VM4 | 172.16.0.1/24 |Añadir router como encaminado por defecto |
# Estados de una conexión TCP
En esta práctica usaremos la herramienta Netcat, que permite leer y escribir en conexiones de red. Netcat es muy útil para investigar y depurar el comportamiento de la red en la capa de transporte, ya que permite especificar un gran número de los parámetros de la conexión. Además para ver el estado de las conexiones de red usaremos la herramienta netstat (también puede usarse la herramienta ss, que es más moderna y completa).

### Ejercicio 1
Consultar las páginas de manual de nc y netstat. En particular, consultar las siguientes opciones de netstat: -a, -l, -n, -t y -o. Probar algunas de las opciones para ambos programas para familiarizarse con su comportamiento.

### Ejercicio 2
(LISTEN) Abrir un servidor TCP en el puerto 7777 en VM1 usando el comando nc -l 7777. Comprobar el estado de la conexión en el servidor con el comando netstat -ltn.

### Ejercicio 3
(ESTABLISHED) En VM2, iniciar una conexión cliente al servidor arrancado en el ejercicio anterior usando el comando nc 192.168.0.1 7777.
 -  Comprobar el estado de la conexión e identificar los parámetros (dirección IP y puerto) con el comando netstat -tn.
 - Reiniciar el servidor en VM1 usando el comando nc -l 192.168.0.1 7777. Comprobar que no es posible la conexión desde VM1 usando como dirección destino localhost. Observar la diferencia con el comando del ejercicio anterior usando netstat.
### Ejercicio 4 [VM1, VM2]
Ejecutar de nuevo la orden ping entre VM1 y VM2 y, a continuación, comprobar el estado de la tabla ARP en VM1 y VM2 usando el comando ip neigh. El significado del estado de cada entrada de la tabla se puede consultar en la página de manual del comando.

### Ejercicio 5 [Router, VM4]
Repetir la configuración de red para el segmento 192.168.0.0/24. Comprobar la conectividad entre Router y VM4; y entre Router, VM1 y VM2.

# Encaminamiento estática

Según la topología de esta práctica la máquina Router puede encaminar el tráfico entre las redes 10.0.0.0/24 y 192.168.0.0/24. En esta sección, vamos a configurar el encaminamiento estático, basado en rutas que fijaremos manualmente en todas las máquinas virtuales.
### Ejercicio 6 [Router]
Activar el reenvío de paquetes (_forwarding_) en Router para que efectivamente pueda funcionar como encaminador entre las redes 10.0.0.0/24 y 192.168.0.0/24. Ejecutar el siguiente comando:
```c
```
### Ejercicio 7 [VM1, VM2]
Añadir la máquina Router como encaminador por defecto para VM1 y VM2. Usar el comando ip route.
```c
```
### Ejercicio 8 [VM4]
Aunque la configuración adecuada para la tabla de rutas de hosts en redes como las consideradas en esta práctica consiste en añadir una ruta por defecto, es posible incluir rutas para redes concretas. Añadir a la tabla de rutas de VM4 una ruta a la red 10.0.0.0/24 via Router.
```c
```
### Ejercicio 9 [VM1, VM2]
Usar la orden ping entre las máquinas VM1 y VM4. Con ayuda de la herramienta wireshark completar la siguiente tabla para todos los paquetes intercambiados hasta la recepción de la primera respuesta Echo Reply.

#### Red 10.0.0.0/24 - VM1
|MAC Origen| MAC Destino | Protocolo | IP Origen | IP Destino | Tipo de Mensaje
|--|--|--|--|--|--|
|  |  |  |  |  |
|  |  |  |  |  |

#### Red 192.168.0.0/24 - VM4
|MAC Origen| MAC Destino | Protocolo | IP Origen | IP Destino | Tipo de Mensaje
|--|--|--|--|--|--|
|  |  |  |  |  |
|  |  |  |  |  |

# Configuración dinámica de hosts
El protocolo DHCP permite configurar dinámicamente los parámetros de red un host. En esta sección configuraremos Router como servidor DHCP para las dos redes. Aunque DHCP puede incluir muchos parámetros de configuración, en esta práctica sólo fijaremos el encaminador por defecto.

### Ejercicio 10 [VM1, VM2] 
Eliminar las direcciones IP de los interfaces (ip addr del)

### Ejercicio 11 [Router]
Configurar el servidor DHCP para las dos redes:
- Editar el fichero /etc/dhcp/dhcpd.conf y añadir dos secciones subnet para cada red que definan los rangos de direcciones_,_ 10.0.0.50 - 10.0.0.100 y 192.168.0.50 - 192.168.0.100, respectivamente. Además, incluir la opción routers con la IP de Router en cada red. 
- Arrancar el servicio con el comando service dhcpd start.

### Ejercicio 12 [Router, VM1]
Iniciar la captura de paquetes en Router. Arrancar el cliente DHCP (dhclient -d eth0) en la máquina virtual VM1 y observar el proceso de configuración. Completar la siguiente tabla:

|IP Origen| IP Destino | Mensaje DHCP | Opciones DHCP
|--|--|--|--|
|  |  |  |  |
|  |  |  |  |

### Ejercicio 13 [VM4]
Durante el arranque del sistema se pueden configurar automáticamente determinados interfaces según la información almacenada en el disco del servidor. Consultar el fichero /etc/sysconfig/network-scripts/ifcfg-eth0 de VM4, que configura automáticamente eth0 usando DHCP. Para más información, consultar el fichero /usr/share/doc/initscripts-*/sysconfig.txt.

### Ejercicio 14 [VM4]
Comprobar la configuración automática con las órdenes ifup e ifdown. Verificar la conectividad entre todas las máquinas de las dos redes.
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTIwNzQ1MDczMV19
-->