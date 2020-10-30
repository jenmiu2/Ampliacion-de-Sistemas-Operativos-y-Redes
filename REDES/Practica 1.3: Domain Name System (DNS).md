# Practica 1.3: Domain Name System (DNS)

# Objetivos
En esta práctica, emplearemos herramientas cliente DNS para explorar la estructura del servicio en Internet. Después, configuraremos un servicio de nombres basado en BIND. El objetivo es estudiar tanto los pasos básicos de configuración del servicio como la base de datos y funcionamiento del protocolo.

# Cliente DNS
Usaremos las herramientas clientes DNS, que serán de utilidad tanto para depurar el despliegue del servicio DNS en nuestra red local, como para estudiar la estructura de DNS en Internet. Las herramientas principales para consultar un servicio DNS son dig y host. Para esta primera parte,  **se usará la máquina física**  del puesto del laboratorio. Si las consultas DNS a determinados servidores estuvieran bloqueadas,  **se usará un interfaz web**como  [www.digwebinterface.com](http://www.digwebinterface.com/)  (activando las opciones “Stats” y “Show command”) o  [www.diggui.com](http://www.diggui.com/).

### Ejercicio 1
Ver el contenido del fichero de configuración del cliente DNS, /etc/resolv.conf. Consultar la página de manual de resolv.conf y buscar las opciones nameserver y search.
Para ver el fichero de configuración:
```bash
sudo cat /etc/resolv.conf
```
[imagen ejercicio 1 - resultado](https://drive.google.com/file/d/1InGjKbiyxvaqyDgkJBbBq8sakG0E1Ot-/view?usp=sharing)
```bash
man resolv.conf
```
Las opciones:
- nameserver: indica la dirección IP del servidor.
- search: muestra la lista de nombres de host buscados

### Ejercicio 2 
Partiendo únicamente del servidor raíz a.root-servers.net y de las respuestas obtenidas de cada servidor obtener la dirección IP de  [informatica.ucm.es](http://informatica.ucm.es/). Determinar el TTL de cada registro y completar la siguiente tabla:
| Servidor | Nombre | TTL | Tipo | Datos | 
|--|--|--|--|--|
| a.root-servers.net | es | 172800 | NS |  |
| g.nic.es. | ucm.es | 172800 | NS |  |
| crispin.sim.ucm.es | informatica.ucm.es | 86400 | CNAME | ucm.es |
|  | ucm.es | 86400 | A | 147.96.1.9 |

Lista de imagenes:
[img ejercicio2 - 1ª busqueda](https://drive.google.com/file/d/19KtE2ypzICNpme25vGLGQf0k91DKCHpg/view?usp=sharing)
[img ejercicio2 - 1ª res](https://drive.google.com/file/d/1kvV8IfuZUrwl_ERYWV6mpS2KpdNK3NJk/view?usp=sharing)
[img ejercicio2 - 2ª busqueda](https://drive.google.com/file/d/1d0I_3pHqSzYIsN-vIjYAL6oqaAu_5hm7/view?usp=sharing)
[img ejercicio2 - 2ª res](https://drive.google.com/file/d/1d_k1g4sDMNTembZGiBHMlo3Z2mrKclZL/view?usp=sharing)
[img ejercicio2 - 3ª busqueda](https://drive.google.com/file/d/1EPK1Q7ma4tTEOjsujLfzexAr9dSPRUYC/view?usp=sharing)
[img ejercicio2 - 3ª res](https://drive.google.com/file/d/13k3vJuLdC3T-0i3e4lR_gZiBWxpf05A8/view?usp=sharing)

**NOTA:** Usar el comando dig @<servidor> <nombre> <tipo>. Más información en la página de manual de dig.

### Ejercicio 3 
Obtener el registro SOA de ucm.es. usando un servidor autoritativo de la zona. Identificar los campos relevantes del registro.
Lista de imágenes:
[img ejercicio3 - busqueda](https://drive.google.com/file/d/1NlOh0qAIFG7poZAm29nhTF75S79AAece/view?usp=sharing)
[img ejercicio3 - res a](https://drive.google.com/file/d/1-1nRAZ-as9ITUGHnLexJ75y0T11qTKDe/view?usp=sharing)
[img ejercicio3 - res b](https://drive.google.com/file/d/17ZwPN1RSLH1nuYerm0GDJmL-p5BqU1a1/view?usp=sharing)

### Ejercicio 4
Determinar qué servidor debería usarse para enviar un mail a  [webmaster@fdi.ucm.es](mailto:webmaster@fdi.ucm.es),, usar un servidor autoritativo de la zona.
Lista de imágenes:
[img ejercicio4 - busqueda](https://drive.google.com/file/d/1ykBdvbtp4wJg80lN543LyVbh6KGY2Sc_/view?usp=sharing)
[img ejercicio4 - res](https://drive.google.com/file/d/1eQ40vnw3TQGiXaRi01UturYoL0jLzP8i/view?usp=sharing)

### Ejercicio 5
Determinar el nombre de dominio para 147.96.85.71. artiendo del servidor raíz a.root-servers.net y usando las respuestas obtenidas. Completar la siguiente tabla:
| Servidor | Nombre | TTL | Tipo | Datos | 
|--|--|--|--|--|
| a.root-servers.net | a-in-addr.arpa. | 172800 | NS |  |
| a-in-addr-servers.arpa. | r.arin.net. | 86400  | NS |  |
| r.arin.net. | ns.ripe.net. | 172800 | NS |  |
| ns.ripe.net. | www.fdi.ucm.es | 86400 | PTR |  |
Lista de imagenes:
[img ejercicio5 - 1ªbusqueda/res](https://drive.google.com/file/d/1ldFSc1OqWG5XKCRccElYfNtTlj8iXNiS/view?usp=sharing)
[img ejercicio5 - 2ª busqueda/res](https://drive.google.com/file/d/1G6v31WVWAhsgsfZa33zztpOQFXPGyy2s/view?usp=sharing)
[img ejercicio5 - 3ª busqueda/res](https://drive.google.com/file/d/1fuHIxuJo0QXB_d3IXAIEim--yQCMpev5/view?usp=sharing)
[img ejercicio5 - 4ª busqueda/res](https://drive.google.com/file/d/1EX7-MXmHwT0kRnHTstWeSpVPC5_FI9sv/view?usp=sharing)
**NOTA:** La opción -x de dig (en el interfaz web, se activa seleccionando “Reverse” como tipo de registro) facilita la búsqueda inversa cuando detecta una dirección IP como argumento, creando el dominio de búsqueda a partir de la dirección IP (esto es, invierte el orden de los bytes y añade .in-addr.arpa.) y estableciendo el tipo de registro por defecto a PTR.

### Ejercicio 6
Obtener la IP de [google](www.google.com) usando el servidor por defecto. Usar la opción +trace del comando dig (option “Trace” en el interfaz web) y observar las consultas realizadas.
Lista de imagenes:
[img ejercicio6 - 1ªbusqueda](https://drive.google.com/file/d/1ZNjwXT874K1BhtWTdwk-Ykq2MBy38vP-/view?usp=sharing)
[img ejercicio6 - 2ª res](https://drive.google.com/file/d/1_ToQF1hrsp1JPzdnWtYQj9NA7IcHs2C6/view?usp=sharing)

# Servidor DNS
## Preparación del entorno
Para esta parte, configuraremos la topología de red que se muestra en la siguiente figura:

**VM1**(eth0 - inet0 192.168.0.1 servidor DNS primario y cache)
**VM2**(eth0 - inet0 192.168.0.100 cliente DNS)

Como en prácticas anteriores, construiremos la topología con la herramienta vtopol y un archivo de topología adecuado. Configurar los interfaces de red como se indica en la figura y comprobar la conectividad entre las máquinas.

## Zona directa (_forward_)
La máquina VM1 actuará como servidor de nombres del dominio labfdi.es. La mayoría de los registros se incluyen en la zona directa.

### Ejercicio 7
Configurar el servidor de nombres añadiendo una entrada zone para la zona directa en el fichero /etc/named.conf. El tipo de servidor de la zona debe ser master y el archivo que define la zona, db.labfdi.es. Por ejemplo:
```bash
sudo gedit /etc/named.conf
sudo cat /etc/named.conf
```
Para que la configuración del fichero sea correcta se debe permitir cualquier tipo de query allow-query {any;}; y recursion no;
Lista de imágenes:
[img ejercicio7 - named conf](https://drive.google.com/file/d/1oGd76YECAu6Zpr1nJWSmTqQhcPdmQuHL/view?usp=sharing)
[img ejercicio7 - nueva zona](https://drive.google.com/file/d/1tcjLvc23mMHa2cfKeFdqzGlHpnyfZDkj/view?usp=sharing)

```c
//se añade esto al final del archivo
zone "labfdi.es." {
	type master;
	file "db.labfdi.es";
};
```
```bash
sudo named-checkconf #comprobamos que el fichero modificado es correcto
```
Una vez comprobado que es correcto, se debe activar el servidor:
```bash
sudo service named start
```
[ejercicio 7 - activado el servidor](https://drive.google.com/file/d/1MOE634y1yewyopLNc_yFEukzLlgq-xnZ/view?usp=sharing)
Revisar la configuración por defecto y consultar la página de manual de named.conf para ver las opciones disponibles para el servidor y las zonas. Por ejemplo, la recursión debe estar deshabilitada en servidores autoritativos y las consultas pueden estar restringidas a ciertas máquinas (directiva allow-query). Una vez creado el archivo de configuración, ejecutar el comando named-checkconf para comprobar que la sintaxis es correcta.

### Ejercicio 8
Crear el archivo de la zona directa labfdi.es. en /var/named/db.labfdi.es con los registros especificados en la siguiente tabla. Especificar también la directiva $TTL.

| Registro | Descripcion |
|--|--|
| Start of Authority (SOA) |Descripción de la zona. Se pueden elegir libremente los valores de refresh, update, expiry y nx ttl. El servidor primario de la zona es ns.labfdi.es y el e-mail de contacto es contact@labfdi.es.|
|Servidor de nombres (NS)|El servidor de nombres es ns.labfdi.es, como se especifica en el registro SOA|
|Dirección (A) del servidor de nombres | La dirección de ns.labfdi.es es 192.168.0.1 (VM1) |
|Direcciones (A y AAAA) del servidor web |Las direcciones de www.labfdi.es son 192.168.0.200 y fd00::1|
|Servidor de correo (MX) |El servidor de correo es mail.labfdi.es|
|Dirección (A) del servidor de correo | La dirección de mail.labfdi.es es 192.168.0.250 |
|Nombre canónico (CNAME) de servidor| El nombre canónico de correo.labfdi.es es mail.labfdi.es |

Una vez generado el archivo de zona, se debe comprobar su integridad con el comando named-checkzone <nombre_zona> <archivo>.

Finalmente, arrancar el servicio DNS con el comando service named start.
**NOTA:** No olvidar que los nombres FQDN terminan en el dominio raíz (“.”). El nombre de la zona puede especificarse con @ en el campo nombre del registro.

```bash 
sudo gedit 
```

### Ejercicio 9
Configurar la máquina virtual cliente para que use el nuevo servidor de nombres. Para ello, crear o modificar /etc/resolv.conf con los nuevos valores para nameserver y search.

### Ejercicio 10
Usar el comando dig en el cliente para obtener la información del dominio labfdi.es.

### Ejercicio 11

Realizar más consultas y, con la ayuda de wireshark:

- Comprobar el protocolo y puerto usado por el cliente y servidor DNS

- Estudiar el formato (campos incluidos y longitud) de los mensajes correspondientes a las preguntas y respuestas DNS.


## Zona inversa (reverse)
Además, el servidor incluirá una base de datos para la búsqueda inversa. La zona inversa contiene los registros PTR correspondientes a las direcciones IP.
### Ejercicio 12
Añadir otra entrada zone para la zona inversa 0.168.192.in-addr.arpa.  en /etc/named.conf. El tipo de servidor de la zona debe ser master y el archivo que define la zona, db.0.168.192.

### Ejercicio 13
Crear el archivo de la zona inversa en /var/named/db.0.168.192 con los registros SOA, NS y PTR. Esta zona usará el mismo servidor de nombres y parámetros de configuración en el registro SOA. Después, reiniciar el servicio DNS con el comando service named restart (o bien, recargar la configuración con el comando service named reload).

### Ejercicio 14
Comprobar el funcionamiento de la resolución inversa, obteniendo el nombre asociado a la dirección 192.168.0.250.
<!--stackedit_data:
eyJoaXN0b3J5IjpbODIxNzEyNTE4LC0xOTc4MTExMjIzLC0xNj
M0Mzc3MzNdfQ==
-->