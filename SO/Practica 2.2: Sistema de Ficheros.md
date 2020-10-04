
# Practica 2.2: Sistema de Ficheros

# Objetivos
En esta práctica se revisan las funciones del sistema básicas para manejar un sistema de ficheros, referentes a la creación de ficheros y directorios, duplicación de descriptores, obtención de información de ficheros o el uso de cerrojos.

# Creación y atributos de ficheros
El i-nodo de un fichero guarda diferentes atributos de éste, como por ejemplo el propietario, permisos de acceso, tamaño o los tiempos de acceso, modificación y creación. En esta sección veremos las llamadas al sistema más importantes para consultar y fijar estos atributos así como las herramientas del sistema para su gestión.

### Ejercicio 1

La herramienta principal para consultar el contenido y atributos básicos de un fichero es ls. Consultar la página de manual y estudiar el uso de las opciones -a -l -d -h -i -R -1 -F y --color. Estudiar el significado de la salida en cada caso.

### Ejercicio 2 
El  _modo_  de un fichero es <tipo><rwx_propietario><rwx_grupo><rwx_resto>:

- tipo: - fichero ordinario; d directorio; l enlace; c dispositivo carácter; b dispositivo bloque; p FIFO; s socket

- rwx: r lectura (4); w escritura (2); x ejecución (1)

Comprobar los permisos de algunos directorios (con ls -ld).

### Ejercicio 3

Los permisos se pueden otorgar de forma selectiva usando la notación octal o la simbólica. Ejemplo, probar las siguientes órdenes (equivalentes):

- ```chmod 540 fichero ```

- ```chmod u+rx,g+r-wx,o-wxr fichero ```

¿Cómo se podrían fijar los permisos rw-r--r-x, de las dos formas?


### Ejercicio 4
Crear un directorio y quitar los permisos de ejecución para usuario, grupo y otros. Intentar cambiar al directorio.

### Ejercicio 5
Escribir un programa que, usando la llamada open(2), cree un fichero con los permisos rw-r--r-x. Comprobar el resultado y las características del fichero con la orden ls.

### Ejercicio 6
Cuando se crea un fichero, los permisos por defecto se derivan de la máscara de usuario (_umask_). El comando interno de la  _shell_ umask permite consultar y fijar esta máscara. Usando este comando, fijar la máscara de forma que los nuevos ficheros no tengan permiso de escritura para el grupo y ningún permiso para otros. Comprobar el funcionamiento con los comandos touch, mkdir y ls.

### Ejercicio 7
Modificar el ejercicio 5 para que, antes de crear el fichero, se fije la máscara igual que en el ejercicio 6. Comprobar el resultado con el comando ls. Comprobar que la máscara del proceso padre (la  _shell_) no cambia.
### Ejercicio 8
El comando ls puede mostrar el i-nodo con la opción -i. El resto de información del i-nodo puede obtenerse usando el comando stat. Consultar las opciones del comando y comprobar su funcionamiento.

### Ejercicio 9
Escribir un programa que emule el comportamiento del comando stat y muestre:

- El número  _major_  y  _minor_ asociado al dispositivo.

- El número de i-nodo del fichero.

- El tipo de fichero (directorio, enlace simbólico o fichero ordinario).

- La hora en la que se accedió el fichero por última vez. ¿Qué diferencia hay entre st_mtime y st_ctime?

### Ejercicio 10
Los enlaces se crean con la orden ln:

- La opción -s crea un enlace simbólico. Crear un enlace simbólico a un fichero ordinario y otro a un directorio. Comprobar el resultado con ls -l y ls -i. Determinar el i-nodo de cada fichero.

- Repetir el apartado anterior con enlaces rígidos. Determinar los i-nodos de los ficheros y las propiedades con stat (observar el atributo número de enlaces).

- ¿Qué sucede cuando se borra uno de los enlaces rígidos? ¿Qué sucede si se borra uno de los enlaces simbólicos? ¿Y si se borra el fichero original?

### Ejercicio 11

Las llamadas link(2) y symlink(2) crean enlaces rígidos y simbólicos, respectivamente. Escribir un programa que reciba una ruta a un fichero como argumento. Si la ruta es un fichero regular, creará un enlace simbólico y rígido con el mismo nombre terminado en .sym y .hard, respectivamente. Comprobar el resultado con la orden ls.

# Redirecciones y duplicación de descriptores

La  _shell_  proporciona operadores (>, >&, >>) que permiten redirigir un fichero a otro, ver los ejercicios propuestos en la práctica opcional. Esta funcionalidad se implementa mediante las llamadas dup(2) y dup2(2).

### Ejercicio 12
Escribir un programa que muestre la hora, en segundos desde el Epoch, usando la función *time(2)*.

### Ejercicio 13 
Modificar el programa anterior para que además de escribir en el fichero la salida estándar también se escriba la salida estándar de error. Comprobar el funcionamiento incluyendo varias sentencias que impriman en ambos flujos. ¿Hay alguna diferencia si las redirecciones se hacen en diferente orden? ¿Por qué no es lo mismo “ls > dirlist 2>&1” que “ls 2>&1 > dirlist”?

### Ejercicio 14
**(Opcional).** La llamada fcntl(2) también permite duplicar descriptores de fichero. Estudiar qué opciones hay que usar para duplicar descriptores.

# Cerrojos de ficheros
El sistema de ficheros ofrece cerrojos de ficheros consultivos.
### Ejercicio 15
El estado y cerrojos de fichero en uso en el sistema se pueden consultar en el fichero /proc/locks. Estudiar el contenido de este fichero.

### Ejercicio 16
Escribir un programa que consulte y muestre en pantalla el estado del cerrojo sobre un fichero. El programa mostrará el estado del cerrojo (bloqueado o desbloqueado). Además:

- Si está desbloqueado, fijará un cerrojo y escribirá la hora actual. Después suspenderá su ejecución durante 30 segundos (función sleep(3)) y a continuación liberará el cerrojo.

- Si está bloqueado, terminará el programa.
### Ejercicio 17
**(Opcional).** El comando flock proporciona funcionalidad de cerrojos antiguos BSD en guiones  _shell_. Consultar la página de manual y el funcionamiento del comando.

# Proyecto: comando ls extendido
Escribir un programa que cumpla las siguientes especificaciones:

- El programa tiene un único argumento que es la ruta a un directorio. El programa debe comprobar la corrección del argumento.

- El programa recorrerá las entradas del directorio de forma que:

			- Si es un fichero normal, escribirá el nombre.

			- Si es un directorio, escribirá el nombre seguido del carácter ‘/’.

			- Si es un enlace simbólico, escribirá su nombre seguido de ‘->’ y el nombre del fichero enlazado. Usar la función readlink(2) y dimensionar adecuadamente el buffer de la función.

			- Si el fichero es ejecutable, escribirá el nombre seguido del carácter ‘*’.

- Al final de la lista el programa escribirá el tamaño total que ocupan los ficheros (no directorios) en kilobytes.
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTc4NTc0NTA2MV19
-->