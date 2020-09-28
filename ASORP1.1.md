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
| VM1 | 192.168.0.1/24 |Añadir router como encamiVM2 | 192.168.0.2/24 | Añadir router como encamidor  conado por defecto |
| VM3 - Router | 192.168.0.3/24 (eth0) 172.16.0.3/24 (eth1)| Añadir forwarding de paquetes |
| VM4 | 172.16.0.1/24 |Añadir router como encaminado por defecto |
# Estados de una conexión TCP
En esta práctica ueremosramienta Netcat, que permite leer y escribir en conexiones de red. Netcat es muy útil para investigar y depurar el comportamiento de la red en la capa de transporte, ya que permite especificar un gran número de los parámetro |  |
# Configuracion estática

### Ejercicio 1
Determinar los interfaces de red que tiene la máquina y las direcciones IP y/o MAC que tienen asignadas. Utilizar el comando ip.
### Ejercicio 2
Activar los interfaces eth0 en las máquinas , y asignar una dirección IP adecuada. La configuración debe realizarse con la utilidad ip, en particular los comandos ip address de la conexión. Además para ver el estado de las conexiones de red usaremos la herramienta netstat (también puede usarse la herramienta ss, que es más moderna y completa).

### Ejercicio 1
Consultar las páginas de manual de nc y netstat. En particular, consultar las siguientes opciones de netstat: -a, -l, -n, -t y -o. Probar algunas de las opciones para ambos programas para familiarizarse con su comportamiento.

### Ejercicio 2
**(LISTEN)** Abrir un servidor TCP en el puerto 7777 en VM1 usando el comando nc -l 7777. Comprobar el estado de la conexión en el servidor con el comando netstat -ltn.

### Ejercicio 3
**(ESTABLISHED)** En VM2, iniciar una conexión cliente al servidor ardo eip link.
### Ejercicio 3
Arrancar la herramienta wireshark y activar la captura en el interfaz de red. Comprobar la conectividad entre VM1 y VM2 con la orden ping. Observa eico geneado,especialmente los protocolos encapsularancado en el ejercicio anterior usandos en cada datagrama y las direcciones origen y destino.
Completar la siguiente tabla para todos los mensajes intera ta corresndiente
Maquina IP  oentarios
 VM1  .12 airutermamiadol comando nc 192.168.0.1 7777.
 -  Comprobar el estado de la conexión e identificar los parámetros (dirección IP y puerto) con el comando netstat -tn.
 - Reinicia por deecto  r el serviador en VM1 usando el comando nc -l 192.168.0.1 7777. Comprobar que no es posible la conexión desde VM1 usando como dirección destino localhost. Observar la diferencia con el comando del ejercicio anterior usando nets e cad.
   stt.t
### Ejercicio 4
Ejecutar de nuevo la orden ping entre VM1 y VM2 y, a continuación, comprobar el estado de la tabla ARP en aa    sando eomano ip neigh. El significado del estado de cada entrada de la tabla se puede consultar en la página de manual del com áina itando.

 jerci
Repetir la configuración de red para el segmento 192.168.0.0/24. Comprobar la conectividad entre Router y VM4; y entre Router, VM1 y VM2.
tat.
 - Iniciar el servidor e intercambiar un único carácter con el cliente. Con ayuda de wireshark, observar los mensajes intercambiados (especialmente los números de secuencia, confirmación y flags TCP) y determinar cuántos bytes (y número de mensajes) han sido necesarios.

### Ejercicio 4
**(TIMEWAIT)** Cerrar la conexión en el cliente (con Ctrl+C) y comprobar el estado de la conexión usando netstat. Usar la opción -o de netstat para observar el valor del temporizador TIMEWAIT.

### Ejercicio 5
**(SYN-SENT y SYN-RCVD)** El comando iptables permite filtrar paquetes según los flags TCP del segmento con la opción --tcp-flags (consultar la página de manual iptables-extensions). Usando esta opción:
- Fijar una regla en el servidor (VM1) que bloquee un mensaje del acuerdo TCP de forma que el cliente (VM2) se quede en el estado SYN-SENT. Comprobar el resultado usando netstat en el cliente.
- Borrar la regla anterior y fijar otra en el cliente que bloquee un mensaje del acuerdo TCP de forma que el servidor se quede en el estado SYN-RCVD. Además, esta regla debe dejar al servidor también en el estado LAST-ACK después de cerrar la conexión (con Ctrl+C) en el cliente. Con ayuda de netstat (usando la opción -o) determinar cuántas retransmisiones se realizan y con qué frecuencia.

### Ejercicio 6
Intentar una conexión a un puerto cerrado del servidor (ej. 7778) y, con ayuda de la herramienta wireshark, observar los mensajes TCP intercambiados, especialmente los flags TCP.
``nc

# Introducción a la seguridad en el protocolo TCP
Diferentes aspectos del protocolo TCP pueden aprovecharse para comprometer la seguridad del sistema. En este apartado vamos a estudiar dos: ataques DoS basados en TCP SYN  _flood_  y técnicas de exploración de puertos.
### Ejercicio 7
El ataque TCP SYN  _flood_  consiste en saturar un servidor mediante el envío masivo de mensajes SYN.
-  **(Cliente VM2)** Para evitar que el atacante responda al mensaje SYN+ACK del servidor con un mensaje RST que liberaría los recursos, bloquear los mensajes SYN+ACK en el atacante con iptables.
- **(Cliente VM2)** Para enviar paquetes TCP con los datos de interés usaremos el comando hping3 (estudiar la página de manual). En este caso, enviar mensajes SYN al puerto 22 del servidor (ssh) lo más rápido posible (_flood_).
- **(Servidor VM1)** Estudiar el comportamiento de la máquina, en términos del número de paquetes recibidos. Comprobar si es posible la conexión al servicio ssh.
- **(Servidor VM1)** Repetir el ejercicio desactivando el mecanismo SYN  _cookies_  en el servidor con el comando sysctl (parámetro net.ipv4.tcp_syncookies).
Como se puede ver existen mas datagramas con longitud menor de 60 bytes, es más facil realizar una ataque de este estilo en una máquina sin las cookies.
### Ejercicio 8
**(Técnica CONNECT)** Netcat permite explorar puertos usando la técnica CONNECT que intenta establecer una conexión a un puerto determinado. En función de la respuesta (SYN+ACK o RST), es posible determinar si hay un proceso escuchando.
- **(Servidor VM1)** Abrir un servidor en el puerto 7777.
- **(Cliente VM2)** Explorar de uno en uno, el rango de puertos 7775-7780 usando nc, en este caso usar las opciones de exploración (-z) y de salida detallada (-v).  **Nota:**  La versión de nc no soporta rangos de puertos. Por tanto, se debe hacer manualmente, o bien, incluir la sentencia de exploración de un puerto en un bucle para automatizar el proceso.
- Con ayuda de wireshark observar los paquetes intercambiados.
**Opcional:** La herramienta nmap permite realizar diferentes ti```c
```
# rcci  la guracin adecuda para la ala e rtas d osts en  oo  cnidera e es rcica consiste en aar un rosr dxploración de puerfectos, que emplean estrategias más eficientes. Estas estrategias (SYN  _stealth_, ACK  _stealth_, FIN-ACK  _stealth_…) son más rápidas que la anterior y se basan en el funcionamiento del protocolo TCP. Estudiar la página de manual de nmap (PORT SCANNING TECHNIQUES) y emplearlas para explorar los puertos del servidor. Comprobar con wireshark los mensajes intercambiados.
# Opciones y parámetros de TCP
El comportamiento de la conexión TCP se puede controlar con varias opciones que se incluyen en la cabecera en los mensajes SYN y que son configurables en el sistema operativo por medio de parámetros del kernel.

### Ejercicio 9
Con ayuda del comando sysctl y la bibliografía recomendada, completar la siguiente tabla con parámetros que permiten modificar algunas opciones dee e sie nli ra ar ee cnresa al a de ruae  ua ru a la e  a otear la  TCP:

|Parametro del kernel| Proposito | Valor por defecto |
|--|--|--|
| net.ipv4.tcp_window_scaling |  |  |
| net.ipv4.tcp_timestamps |  |  |
| net.ipv4.tcp_sack |  |  |
### Ejercicio 10 
Abrir el servidor en el puerto 7777 y realizar una conexión desde la VM cliente. Con ayuda de wireshark estudiar el valor de las opciones que se intercambian durante la conexión. Variar algunos de los parámetros anteriores (ej. no usar ACKs selectivos) y observar el resultado en una nueva conexión.
### Ejercicio 11 
Con ayuda del comando sysctl y la bibliografía recomendada, completar la siguiente tabla con parámetros que permiten configurar el temporizador  _keepalive_:
defecto |
|--|--|--|
| net.ipv4.tcp_keepalive_time |  |  |
| net.ipv4.tcp_keepalive_probes |  |  |
| net.ipv4.tcp_keepalive_intvl |  |  |
# Traducción de direcciones NAT y reenvío de puertos 
En esta sección supondremos que la red que conecta Router (VM3) con VM4 es pública y que no puede encaminar el tráfico 192.168.0.0/24. Además, asumiremos que la IP de Router es dinámica.

### Ejercicio 12 [Router, VM1]
Configurar la traducción de direcciones dinámica en Router:

-

### Ejercicio 13 [VM4]
Durante el arranque del sistema se pueden configurar automáticamente determinados interfaces según la información almacenada en el disco del servidor. Consultar el fichero /etc/sysconfig/network-scripts/ifcfg-eth0 de VM4, que configura automáticamente eth0 usando DHCP. Para más información, consultar el fichero /usr/share/doc/initscripts-*/sysconfig.txt.

### Ejercicio 14 [VM4]
Comprobar la configuración automática con las órdenes ifup e ifdown. Verificar la conectividad entre todas las máquinas de las dos redes.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIzMTgzMjc3OSwxNjYyOTQ2NTU5LDIxMj
U2NTY1NzksNDA5MDkxNjhdfQ==
-->