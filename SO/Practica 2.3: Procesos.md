
# Practica 2.3: Procesos

# Objetivos
En esta práctica se revisan las funciones del sistema básicas para la gestión de procesos: políticas de planificación, creación de procesos, grupos de procesos, sesiones, recursos de un proceso y gestión de señales.

# Políticas de planificación
En esta sección estudiaremos los parámetros de planificador de Linux que permiten variar y consultar la prioridad de un proceso. Veremos tanto la interfaz del sistema como algunos comandos importantes.

### Ejercicio 1

La política de planificación y la prioridad de un proceso puede consultarse y modificarse con el comando chrt. Adicionalmente, los comandos nice y renice permiten ajustar el valor de  _nice_  de un proceso. Consultar la página de manual de ambos comandos y comprobar su funcionamiento cambiando el valor de  _nice_  de la  _shell_  a -10 y después cambiando su política de planificación a SCHED_FIFO con prioridad 12.

### Ejercicio 2 

Escribir un programa que muestre la política de planificación (como cadena) y la prioridad del proceso actual, además de mostrar los valores máximo y mínimo de la prioridad para la política de planificación.

### Ejercicio 3

Ejecutar el programa anterior en una  _shell_  con prioridad 12 y política de planificación SCHED_FIFO como la del ejercicio 1. ¿Cuál es la prioridad en este caso del programa? ¿Se heredan los atributos de planificación?

# Grupos de procesos y sesiones
Los grupos de procesos y sesiones simplifican la gestión que realiza la  _shell_, ya que permite enviar de forma efectiva señales a un grupo de procesos (suspender, reanudar, terminar…). En esta sección veremos esta relación y estudiaremos el interfaz del sistema para controlarla.
### Ejercicio 4
El comando ps es de especial importancia para ver los procesos del sistema y su estado. Estudiar la página de manual y:
- Mostrar todos los procesos del usuario actual en formato extendido.

- Mostrar los procesos del sistema, incluyendo el identificador del proceso, el identificador del grupo de procesos, el identificador de sesión, el estado y la línea de comandos.

- Observar el identificador de proceso, grupo de procesos y sesión de los procesos. ¿Qué identificadores comparten la  _shell_  y los programas que se ejecutan en ella? ¿Cuál es el identificador de grupo de procesos cuando se crea un nuevo proceso?

### Ejercicio 5
Escribir un programa que muestre los identificadores del proceso: identificador de proceso, de proceso padre, de grupo de procesos y de sesión. Mostrar además el número máximo de archivos que puede abrir el proceso y el directorio de trabajo actual.

### Ejercicio 6
Un demonio es un proceso que se ejecuta en segundo plano para proporcionar un servicio. Normalmente, un demonio está en su propia sesión y grupo. Para garantizar que es posible crear la sesión y el grupo, el demonio crea un nuevo proceso para ejecutar la lógica del servicio y crear la nueva sesión. Escribir una plantilla de demonio (creación del nuevo proceso y de la sesión) en el que únicamente se muestren los atributos del proceso (como en el ejercicio anterior). Además, fijar el directorio de trabajo del demonio a /tmp.
¿Qué sucede si el proceso padre termina antes que el hijo (observar el PPID del proceso hijo)? ¿Y si el proceso que termina antes es el hijo (observar el estado del proceso hijo con ps)?

```c
#include <unistd.h>
#include <errno.h>

#define OK_FORK 0
#define KO_FORK -1

#define errnoexit() do{printf("ERROR(%d): %s", errno, strerror(errno)); EXIT(EXIT_FAILURE);};while(0)
void printattr(pid_t pid) {
	print("pid(), parent pid(), group pid(), sesion pid()", pid, getppid(), getpgid(pid), getsid(pid));
}

int main(int argc, int argv*[]) {
	pid_t pid, sid;
	pid = fork();

	if(pid < KO_FORK) {
		errnoexit();
	}
	if(pid > OK_FORK) { 	
		umask(0);
		sid = setsid();
		if(chdir("/tmp")) < 0) {
			errnoexit();
		}
		printattr(pid);
	}
	else {
		sleep(30000);
	}
}
```
1. ¿Qué sucede si el proceso padre termina antes que el hijo (observar el PPID del proceso hijo)? 
	La ejecución es correcta.
2. ¿Y si el proceso que termina antes es el hijo (observar el estado del proceso hijo con ps)?
	El proceso hijo se convierte en un proceso zombie, sonde el nuevo identificador padre pertenece al SO.
	 
**Nota:** Usar sleep(3) o pause(3) para forzar el orden de finalización deseado.

# Ejecución de programas

### Ejercicio 7
Escribir dos versiones, una con system(3) y otra con execvp(), de un programa que ejecute otro programa que se pasará como argumento por línea de comandos. En cada caso, se debe imprimir la cadena “El comando terminó de ejecutarse” después de la ejecución. ¿En qué casos se imprime la cadena? ¿Por qué?
**Nota:** Considerar cómo deben pasarse los argumentos en cada caso para que sea sencilla la implementación. Por ejemplo: ¿qué diferencia hay entre ./ejecuta ps -el  y ./ejecuta “ps -el”?
### Ejercicio 8
Usando la versión con execvp() del ejercicio 7 y la plantilla de demonio del ejercicio 6, escribir un programa que ejecute cualquier programa como si fuera un demonio. Además, redirigir los flujos estándar asociados al terminal usando dup2(2):

- La salida estándar al fichero /tmp/daemon.out.

- La salida de error estándar al fichero /tmp/daemon.err.

- La entrada estándar a /dev/null.

Comprobar que el proceso sigue en ejecución tras cerrar la  _shell_.

# Señales

### Ejercicio 9
El comando kill permite enviar señales a un proceso o grupo de procesos por su identificador (pkill permite hacerlo por nombre de proceso). Estudiar la página de manual del comando y las señales que se pueden enviar a un proceso.

### Ejercicio 10

En un terminal, arrancar un proceso de larga duración (ej. sleep 600). En otra terminal, enviar diferentes señales al proceso, comprobar el comportamiento. Observar el código de salida del proceso. ¿Qué relación hay con la señal enviada?

### Ejercicio 11

Escribir un programa que bloquee las señales SIGINT y SIGTSTP. Después de bloquearlas el programa debe suspender su ejecución con sleep(3) un número de segundos que se obtendrán de la variable de entorno SLEEP_SECS.

Después de despertar de sleep(3), el proceso debe informar de si recibió la señal SIGINT y/o SIGTSTP. En este último caso, debe desbloquearla con lo que el proceso se detendrá y podrá ser reanudado en la  _shell_  (imprimir una cadena antes de finalizar el programa para comprobar este comportamiento).
```c
#include <errno.h>
#include <stdio.h>
#include <stlib.h>
#include <sys/types.h>
#include <signal.h>

int main(int argc, int argv*[]) {
	sigset_t blk, pending_blk;
	
	sigemptyset(&blk);
	sigaddset(&blk, SIGINT);
	sigaddset(&blk, SIGTSTP);

	sigprocmask(SIG_BLOCK, &blk, NULL);
	
	char *sleep_sec_chr = getenv("SLEEP_SECS");
	int sleep_sec = atoi(sleep_sec_chr);
	
	sleep(sleep_sec);
	
	sigpending(&pending_blk);
	if(sigismember(&pending_blk, SIGINT) == 1 || sigismember(&pending_blk, SIGTSTP) == 1) {
		printf("Se ha recibido la señal SIGINT o SIGTSTP");
	}
	else {
		printf("No se ha recibido la señal SIGINT o SIGTSTP");
	}
	sigprocmask(SIG_UNBLOCK, &set, NULL);
}
```
### Ejercicio 12
Escribir un programa que instale un manejador sencillo para las señales SIGINT y SIGTSTP. El manejador debe contar las veces que ha recibido cada señal. El programa principal permanecerá en un bucle que se detendrá cuando se hayan recibido 10 señales. El número de señales de cada tipo se mostrará al finalizar el programa.

### Ejercicio 13 
Escribir un programa que realice el borrado programado del propio ejecutable. El programa tendrá como argumento el número de segundos que esperará antes de borrar el fichero. El borrado del fichero se podrá detener si se recibe la señal SIGUSR1.
```c
#include <errno.h>
#include <stdio.h>
#include <stlib.h>
#include <sys/types.h>
#include <signal.h>

void handler(in)
int main(int argc, int argv*[]) {
	sigset_t blk;
	struct sigaction sigact;
	sigemptyset(&blk);
	sigaddset(&blk, SIGUSR1);
	sigemptyset(&sigact);
	sigact = (sigaction) {.sa_flags = 0, sa_handler = handler};
	
	
	if(sigsuspend(&blk) == -1) {
		remove(argv[1]);
	}
}
```
**Nota:** Usar sigsuspend(2) para suspender el proceso y la llamada al sistema apropiada para borrar el fichero.

<!--stackedit_data:
eyJoaXN0b3J5IjpbNTYxMDYyNzA2LC0xNjg5ODk5MTA1LC0xNj
Q4ODk2NzY4LC05NTQyODA5NjAsLTEwNjk0NDc5MzAsLTE2Njgz
ODM4OTEsLTMwMjE3NTIwMSwtMTExNjc4OTYxMiwtNzcxMjgyMT
kwLC0xMjUwMjA5NzJdfQ==
-->