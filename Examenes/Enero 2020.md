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
#### VM3
Insertar el siguiente código en c.
```bash
$ip a add FD00:1:1:A::2/64 dev eth0
$ip a add FD00:1:1:B::2/64 dev eth1
sysctl -w net.ipv6.conf.all.forwarding = 1
sudo nano /etc/quagga/zebra.conf 
```
```c
interface eth0
	no ipv6 nd suppress-ra
	ipv6 nd prefix FD00:1:1:A::/64
interface eth1
	no ipv6 nd suppress-ra
	ipv6 nd prefix FD00:1:1:B::/64
```
```bash
sudo service zebra start
```


#### VM1
```bash
$ip a add FD00:1:1:A::1/64 dev eth0
$ip -6 add default via FD00:1:1:A::2/64
$ip link set dev eth0 up
$ip a
```
#### VM2
```bash
$ip a add FD00:1:1:B::1/64 dev eth0
$ip -6 add default via FD00:1:1:B::2/64
$ip link set dev eth1 up
$ip a
```
Para comprobar que se ha podido realizar una correcta configuración se debe realizar:
```c
ping6 ipv6_vm1 y ipv6_vm2
```
## Ejercicio 2 (1 punto)
Escribe un programa en TCP que escuche en una dirección (IPv4-IPv6) y puerto dados como argumentos. El servidor devolverá lo qu4e el cliente le envíe. En cada conexión, el servidor mostrará la dirección y el puerto del cliente.
Realizar una búsqueda previa man 3 getaddrinfo y obtenemos el siguiente código:
**Servidor**:
```bash
sudo gedit servidor_udp.c
```
```c
    #include <sys/types.h>
       #include <stdio.h>
       #include <stdlib.h>
       #include <unistd.h>
       #include <string.h>
       #include <sys/socket.h>
       #include <netdb.h>

       #define BUF_SIZE 500

       int
       main(int argc, char *argv[])
       {
           struct addrinfo hints;
           struct addrinfo *result, *rp;
           int sfd, s;
           struct sockaddr_storage peer_addr;
           socklen_t peer_addr_len;
           ssize_t nread;
           char buf[BUF_SIZE];

           if (argc != 3) {
               fprintf(stderr, "Usage: %s port\n", argv[0]);
               exit(EXIT_FAILURE);
           }

           memset(&hints, 0, sizeof(hints));
           hints.ai_family = AF_UNSPEC;    /* Allow IPv4 or IPv6 */
           hints.ai_socktype = SOCK_DGRAM; /* Datagram socket: UDP */
           hints.ai_flags = AI_PASSIVE;    /* For wildcard IP address */
           hints.ai_protocol = 0;          /* Any protocol */
           hints.ai_canonname = NULL;
           hints.ai_addr = NULL;
           hints.ai_next = NULL;

           s = getaddrinfo(argv[1], argv[2], &hints, &result);
           if (s != 0) {
               fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(s));
               exit(EXIT_FAILURE);
           }

           /* getaddrinfo() returns a list of address structures.
              Try each address until we successfully bind(2).
              If socket(2) (or bind(2)) fails, we (close the socket
              and) try the next address. */

           for (rp = result; rp != NULL; rp = rp->ai_next) {
               sfd = socket(rp->ai_family, rp->ai_socktype,
                       rp->ai_protocol);
               if (sfd == -1)
                   continue;

               if (bind(sfd, rp->ai_addr, rp->ai_addrlen) == 0)
                   break;                  /* Success */

               close(sfd);
           }

           freeaddrinfo(result);           /* No longer needed */

           if (rp == NULL) {               /* No address succeeded */
               fprintf(stderr, "Could not bind\n");
               exit(EXIT_FAILURE);
           }

           /* Read datagrams and echo them back to sender. */

           for (;;) {
               peer_addr_len = sizeof(peer_addr);
               nread = recvfrom(sfd, buf, BUF_SIZE, 0,(struct sockaddr *) &peer_addr, &peer_addr_len);
               if (nread == -1)
                   continue;               /* Ignore failed request */

               char host[NI_MAXHOST], service[NI_MAXSERV];

               s = getnameinfo((struct sockaddr *) &peer_addr,peer_addr_len, host, NI_MAXHOST,service,NI_MAXSERV, NI_NUMERICSERV);
               if (s == 0) {
               /*por aqui recive del cliente datos*/
				printf("Received %zd bytes from %s:%s\n",nread, host, service);
				}
               else {
                   fprintf(stderr, "getnameinfo: %s\n", gai_strerror(s));
				}
				printf("HOST %s, PORT %s\n", host, service);
           }
       }
```
```bash
gcc -o servidor_udp servidor_udp.c
./servidor_udp 192.168.0.3 80
```
**Cliente**:
```bash
nc -u 192.168.0.1 80
```
## Ejercicio 3 (1,5 puntos)
Escribe un programa que ejecute dos comandos de la siguiente forma:
- Los comandos serán el primer y segundo argumento del programa. El resto de argumentos del programa se considerarán argumentos del segundo comando:
- Cada comando se ejecutará en un proceso distinto, que imprimirá su PID por terminal.
- El programa conectará la salida estándar del primer proceso con la entrada estándar del segundo, y esperará la finalización de ambos para terminar su ejecución.

Ejemplificación:
```bash
$ ./conecta comando1 comando2 arg2_1 arg2_2 
```
 ```c
 #include <stdio.h>
 #include <errno.h>
 #include <unistd.h>
 #include <stlib.h>
  #include <sys/types.h>

#define KO_FORK -1
#define OK_FORK 0
#define MAX_SIZE 100


int main(int argc, char * argv[]) {

	if (argc != 2) {
              fprintf(stderr, "Usage: $ ./conecta comando1 comando2 arg2_1 arg2_2 \n");
              exit(EXIT_FAILURE);
        }
     pid_t pid;
     pid = fork();
     int fd[2];

	 if (pid == KO_FORK) {
		exit(EXIT_FAILURE);
	 }
	 else if (pid == OK_FORK) {
	 /*HIJO*/
	 int i = 1;
	 char comando = malloc(sizeof(char)*MAX_SIZE);
		for(i; i < argc; i++) {
			strcat(comando, argv[i]);
			strcat(comando, " ");
		}
		dup2(fd[0], 1);
		close(fd[0]);
		close(fd[1]);
		execvp(argv2[2], comando);
	}
	else {
	/*PADRE*/
		dup2(fd[1], 0);//salida estandar -> extremo tuberia para que el proceso 2 lo lea
		close(fd[0]);
		close(fd[1]);
		exec(argv[1], argv + 1);
		printf("HELLO, WORLD!!");
		sleep(30000);
	}
}
 ```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1MDgyNTIyODcsNTA3MzU4MTU5LC0xOT
U5OTQxMzc3LC05ODIzMDQ1NzEsLTIwOTc5NDIyMzYsLTY4Mzkw
OTE5NiwyODU2Njk2MTJdfQ==
-->