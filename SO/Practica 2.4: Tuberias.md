
# Practica 2.4: Tuberias

# Objetivos
Las tuberías ofrecen un mecanismo sencillo y efectivo para la comunicación entre procesos en un mismo sistema. En esta práctica veremos los comandos e interfaz para la gestión de tuberías, y los patrones de comunicación típicos.

# Tuberias sin nombre
Las tuberías sin nombre son entidades gestionadas directamente por el núcleo del sistema y son un mecanismo de comunicación unidireccional eficiente para procesos relacionados (padre-hijo). La forma de compartir los identificadores de la tubería es por herencia (en la llamada fork(2)).

### Ejercicio 1

Escribir un programa que emule el comportamiento de la shell en la ejecución de una sentencia en la forma: comando1 argumento1 | comando2 argumento2. El programa creará una tubería sin nombre y creará un hijo:

- El proceso padre redireccionará la salida estándar al extremo de escritura de la tubería y ejecutará comando1 argumento1.

- El proceso hijo redireccionará la entrada estándar al extremo de lectura de la tubería y ejecutará comando2 argumento2.

Probar el funcionamiento con una sentencia similar a: ./ejercicio1 echo 12345 wc -c
```c
#include <errno.h>
#include <stdio.h>
#include <stlib.h>
#include <sys/types.h>

#define NO_ERROR 0
#define ERROR -1
#define errorexit() do{ printf("ERROR(%d): %s\n", errno, sterror(errno));EXIT(EXIT_FAILURE);}; while(0)
#define MAX_SIZE 100

int main(int argc, int argv* []) {
	pid_t pid;
	pid = fork();
	int fd[2]; //tuberia: lectura[0], escritura[1]
	if(argc != 5) {
		printf("Usage: comando1 argumento1 | comando2 argumento2\n");
		EXIT(EXIT_FAILURE);
	}
	if(pid == OK_FORK) {
		close(fd[1]);
		dup2(fd[0], 0);
		if (execlp(argv[3], argv[3], argv[4]) == ERROR) {
			errorexit();
		}
	}
	else if(pid == KO_FORK) {
		errorexit();
	}
	else {
	/*padre*/
		close(fd[0]);
		dup2(fd[1], 1); //redirecciona la salida
		if (execlp(argv[1], argv[1], argv[2]) == ERROR) {
			errorexit();
		}
	}
}
```
**Nota:** Antes de ejecutar el comando correspondiente, deben cerrarse todos los descriptores no necesarios.

### Ejercicio 2 

Para la comunicación bi-direccional, es necesario crear dos tuberías, una para cada sentido: p_h y h_p. Escribir un programa que implemente el mecanismo de sincronización de parada y espera:

-   El padre leerá de la entrada estándar (terminal) y enviará el mensaje al proceso hijo, escribiéndolo en la tubería p_h. Entonces permanecerá bloqueado esperando la confirmación por parte del hijo en la otra tubería, h_p.
- El hijo leerá de la tubería p_h, escribirá el mensaje por la salida estándar y esperará 1 segundo. Entonces, enviará el carácter ‘l’ al proceso padre, escribiéndolo en la tubería h_p, para indicar que está listo. Después de 10 mensajes enviará el carácter ‘q’ para indicar al padre que finalice.

```c
#include <errno.h>
#include <stdio.h>
#include <stlib.h>
#include <sys/types.h>

#define OK_FORK 0
#define KO_FORK -1
#define errorexit() do{ printf("ERROR(%d): %s\n", errno, sterror(errno));}; while(0)
#define MAX_SIZE 100
#define MAX_COUNT 10 /*valor maximo del contador de mensajes*/

int main(int argc, int argv*[]) {
	int p_h[2]; //padre -> hijo:: p_escribe: 0, h_lee: 1
	int h_p[2]; //hijo -> padre:: h_escribre: 0, p_lee: 1
	int readbytes, count_msg = 0;
	pid_t pid;
	char chr[MAX_SIZE];
	
	pid = fork();
	if(pid == KO_FORK) {
		errorexit();
	}
	else if(pid == OK_FORK) {
		close(h_p[0]); //el padre no va a leer
		close(p_h[1]); //el padre no va a escribir
		while((readbytes = read(p_h[0], &chr, 1)) > 0 && count_msg < MAX_COUNT) {
			printf("%s", chr);
			sleep(1); /*1 segundo*/
			write(h_p[1], "l", sizeof(char));
			count_msg = count_msg + 1;
		}
		printf("\n");
		write(h_p[1], "q", sizeof(char));
		close(h_p[0]);//el hijo no va a escribir
		close(p_h[1]);//el hijo no va a leer
		EXIT(EXIT_SUCCESS);
	}
	else {
		close(h_p[0]);//el hijo no va a escribir
		close(p_h[1]);//el hijo no va a leer
		while((readbytes = read(h_p[1], &chr, 1)) > 0) {
			scanf("%c", &chr);
			write(p_h[0], chr, sizeof(chr));
			if(chr == 'q') {
				EXIT(EXIT_SUCCESS);
			} 
		}
		waitpid(pid, NULL, 0);
		close(h_p[0]); //el padre no va a leer
		close(p_h[1]); //el padre no va a escribir
	}
}
```
# Tuberias con nombre
Las tuberías con nombre son un mecanismo de comunicación unidireccional, con acceso de tipo FIFO, útil para procesos sin relación de parentesco. La gestión de las tuberías con nombre es igual a la de un archivo ordinario (open, write, read…). Revisar la información en fifo(7).

### Ejercicio 3
Usar la orden mkfifo para crear una tubería con nombre. Usar las herramientas del sistema de ficheros (stat, ls…) para determinar sus propiedades. Comprobar su funcionamiento usando utilidades para escribir y leer de ficheros (ej. echo, cat, tee...).

### Ejercicio 4

Escribir un programa que abra la tubería con el nombre anterior en modo sólo escritura, y escriba en ella el primer argumento del programa. En otro terminal, leer de la tubería usando un comando adecuado.
```c
#include <sys/types.h>
#include <sys/stat.h>
#include <stdlib.h>
#include <errno.h>

#define errnoexit() do{printf("ERROR(%d): %s", errno, strerror(errno)); EXIT(EXIT_FAILURE);} while(0) 
#define NO_ERROR 0

int main(int argc, int argv*[]) {
	char pathname = "pìpe.txt"
	int fd, writebytes;
	if(mkfifo(pathname, 0777) < NO_ERROR) {
		errnoexit();
	}
	if((fd = open(pathname,O_WRONLY)) < NO_ERROR) {
		errnoexit();
	}
	writebytes = write(fd, argv[1], sizeof(argv[1]);
	if((writebytes = write(fd, argv[1], sizeof(argv[1])) < NO_ERROR) {
		errnoexit();
	}
	EXIT(EXIT_SECCESS);
}
```
En otro terminal escribir:
```bash
cat tub
```
# Multiplexación síncrona de entrada/salida

Es habitual que un proceso lea o escriba de diferentes flujos. La llamada select(2) permite multiplexar las diferentes operaciones de E/S sobre múltiples flujos.
### Ejercicio 5
Crear otra tubería con nombre. Escribir un programa que espere hasta que haya datos listos para leer en alguna de ellas. El programa debe mostrar la tubería desde la que leyó y los datos leídos. Consideraciones:

- Para optimizar las operaciones de lectura usar un  _buffer_  (ej. de 256 bytes).

- Usar read(2) para leer de la tubería y gestionar adecuadamente la longitud de los datos leídos.

- Normalmente, la apertura de la tubería para lectura se bloqueará hasta que se abra para escritura (ej. con echo 1 > tuberia). Para evitarlo, usar la opción O_NONBLOCK en open(2).

- Cuando el escritor termina y cierra la tubería, read(2) devolverá 0, indicando el fin de fichero, por lo que hay que cerrar la tubería y volver a abrirla. Si no, select(2) considerará el descriptor siempre listo para lectura y no se bloqueará.
```c

```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTQ3OTUxNTUwMiwtMTE5MzMwNTk1NywtNT
I0NjIwNTQ3LC0zODQ2NTg4MTAsLTMwNTI0MzQxMCw0MTI2MjM0
ODldfQ==
-->