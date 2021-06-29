# Examen Práctico Enero 2020
## Ejercicio 1 (1,5 puntos)
Despliegue la topología de red que se muestra en la figura usando vtopol u la configuración proporcionada: 
- Configura los interfaces de forma manual, eligiendo adecuadamente sus direcciones.
- Configura el encaminado router para que anuncie prefijos en ambas redes
- Comprueba que todas las maquinas son alcanzables entre si

### Preparación del entorno
|Maquina| IP | Comentarios
|--|--|--|
| VM1 | FD00:1:1:A::/64 (eth0)|Añadir router como encaminador por defecto |
| VM2 | FD00:1:1:B::/64 (eth0)| Añadir router como encaminador por defecto|
| VM3 - Router |  | Añadir forwarding de paquetes |

```bash
netprefix inet
machine 1 0 0
machine 2 0 1
machine 3 0 0 1 1
```
#### VM1
```bash
$sudo ip address add FD00:1:1:A::1/64 dev eth0
$ip link set dev eth0 up
$ip a
```
#### VM2
```bash
$sudo ip address add FD00:1:1:B::0/64 dev eth0
$ip link set dev eth0 up
$ip a
```
#### VM3
```bash
$sudo ip address add FD00:1:1:A::3/64 dev eth0
$sudo ip address add FD00:1:1:B::3/64 dev eth1
$ip link set dev eth0 up
$ip link set dev eth up
$ip a
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTA2MTA1MDA3NCwyODU2Njk2MTJdfQ==
-->