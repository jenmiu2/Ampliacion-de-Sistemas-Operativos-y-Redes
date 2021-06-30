---


---

<h1 id="practica-2.3-procesos">Practica 2.3: Procesos</h1>
<h1 id="objetivos">Objetivos</h1>
<p>En esta práctica se revisan las funciones del sistema básicas para la gestión de procesos: políticas de planificación, creación de procesos, grupos de procesos, sesiones, recursos de un proceso y gestión de señales.</p>
<h1 id="políticas-de-planificación">Políticas de planificación</h1>
<p>En esta sección estudiaremos los parámetros de planificador de Linux que permiten variar y consultar la prioridad de un proceso. Veremos tanto la interfaz del sistema como algunos comandos importantes.</p>
<h3 id="ejercicio-1">Ejercicio 1</h3>
<p>La política de planificación y la prioridad de un proceso puede consultarse y modificarse con el comando chrt. Adicionalmente, los comandos nice y renice permiten ajustar el valor de  <em>nice</em>  de un proceso. Consultar la página de manual de ambos comandos y comprobar su funcionamiento cambiando el valor de  <em>nice</em>  de la  <em>shell</em>  a -10 y después cambiando su política de planificación a SCHED_FIFO con prioridad 12.</p>
<h3 id="ejercicio-2">Ejercicio 2</h3>
<p>Escribir un programa que muestre la política de planificación (como cadena) y la prioridad del proceso actual, además de mostrar los valores máximo y mínimo de la prioridad para la política de planificación.</p>
<h3 id="ejercicio-3">Ejercicio 3</h3>
<p>Ejecutar el programa anterior en una  <em>shell</em>  con prioridad 12 y política de planificación SCHED_FIFO como la del ejercicio 1. ¿Cuál es la prioridad en este caso del programa? ¿Se heredan los atributos de planificación?</p>
<h1 id="grupos-de-procesos-y-sesiones">Grupos de procesos y sesiones</h1>
<p>Los grupos de procesos y sesiones simplifican la gestión que realiza la  <em>shell</em>, ya que permite enviar de forma efectiva señales a un grupo de procesos (suspender, reanudar, terminar…). En esta sección veremos esta relación y estudiaremos el interfaz del sistema para controlarla.</p>
<h3 id="ejercicio-4">Ejercicio 4</h3>
<p>El comando ps es de especial importancia para ver los procesos del sistema y su estado. Estudiar la página de manual y:</p>
<ul>
<li>
<p>Mostrar todos los procesos del usuario actual en formato extendido.</p>
</li>
<li>
<p>Mostrar los procesos del sistema, incluyendo el identificador del proceso, el identificador del grupo de procesos, el identificador de sesión, el estado y la línea de comandos.</p>
</li>
<li>
<p>Observar el identificador de proceso, grupo de procesos y sesión de los procesos. ¿Qué identificadores comparten la  <em>shell</em>  y los programas que se ejecutan en ella? ¿Cuál es el identificador de grupo de procesos cuando se crea un nuevo proceso?</p>
</li>
</ul>
<h3 id="ejercicio-5">Ejercicio 5</h3>
<p>Escribir un programa que muestre los identificadores del proceso: identificador de proceso, de proceso padre, de grupo de procesos y de sesión. Mostrar además el número máximo de archivos que puede abrir el proceso y el directorio de trabajo actual.</p>
<h3 id="ejercicio-6">Ejercicio 6</h3>
<p>Un demonio es un proceso que se ejecuta en segundo plano para proporcionar un servicio. Normalmente, un demonio está en su propia sesión y grupo. Para garantizar que es posible crear la sesión y el grupo, el demonio crea un nuevo proceso para ejecutar la lógica del servicio y crear la nueva sesión. Escribir una plantilla de demonio (creación del nuevo proceso y de la sesión) en el que únicamente se muestren los atributos del proceso (como en el ejercicio anterior). Además, fijar el directorio de trabajo del demonio a /tmp.<br>
¿Qué sucede si el proceso padre termina antes que el hijo (observar el PPID del proceso hijo)? ¿Y si el proceso que termina antes es el hijo (observar el estado del proceso hijo con ps)?</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;unistd.h&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;errno.h&gt;</span></span>

<span class="token macro property">#<span class="token directive keyword">define</span> OK_FORK 0</span>
<span class="token macro property">#<span class="token directive keyword">define</span> KO_FORK -1</span>

<span class="token macro property">#<span class="token directive keyword">define</span> errnoexit() do{printf("ERROR(%d): %s", errno, strerror(errno)); EXIT(EXIT_FAILURE);};while(0)</span>

<span class="token keyword">void</span> <span class="token function">printattr</span><span class="token punctuation">(</span>pid_t pid<span class="token punctuation">)</span> <span class="token punctuation">{</span>
	<span class="token function">print</span><span class="token punctuation">(</span><span class="token string">"pid(), parent pid(), group pid(), sesion pid()"</span><span class="token punctuation">,</span> pid<span class="token punctuation">,</span> <span class="token function">getppid</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">,</span> <span class="token function">getpgid</span><span class="token punctuation">(</span>pid<span class="token punctuation">)</span><span class="token punctuation">,</span> <span class="token function">getsid</span><span class="token punctuation">(</span>pid<span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>

<span class="token keyword">int</span> <span class="token function">main</span><span class="token punctuation">(</span><span class="token keyword">int</span> argc<span class="token punctuation">,</span> <span class="token keyword">int</span> argv<span class="token operator">*</span><span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
	pid_t pid<span class="token punctuation">,</span> sid<span class="token punctuation">;</span>
	pid <span class="token operator">=</span> <span class="token function">fork</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

	<span class="token keyword">if</span><span class="token punctuation">(</span>pid <span class="token operator">&lt;</span> KO_FORK<span class="token punctuation">)</span> <span class="token punctuation">{</span>
		<span class="token function">errnoexit</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token punctuation">}</span>
	<span class="token keyword">if</span><span class="token punctuation">(</span>pid <span class="token operator">&gt;</span> OK_FORK<span class="token punctuation">)</span> <span class="token punctuation">{</span> 	
		<span class="token function">umask</span><span class="token punctuation">(</span><span class="token number">0</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
		sid <span class="token operator">=</span> <span class="token function">setsid</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
		<span class="token keyword">if</span><span class="token punctuation">(</span><span class="token function">chdir</span><span class="token punctuation">(</span><span class="token string">"/tmp"</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token operator">&lt;</span> <span class="token number">0</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
			<span class="token function">errnoexit</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
		<span class="token punctuation">}</span>
		<span class="token function">printattr</span><span class="token punctuation">(</span>pid<span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token punctuation">}</span>
	<span class="token keyword">else</span> <span class="token punctuation">{</span>
		<span class="token function">sleep</span><span class="token punctuation">(</span><span class="token number">30000</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre>
<ol>
<li>¿Qué sucede si el proceso padre termina antes que el hijo (observar el PPID del proceso hijo)?<br>
La ejecución es correcta.</li>
<li>¿Y si el proceso que termina antes es el hijo (observar el estado del proceso hijo con ps)?<br>
El proceso hijo se convierte en un proceso zombie, sonde el nuevo identificador padre pertenece al SO.</li>
</ol>
<p><strong>Nota:</strong> Usar sleep(3) o pause(3) para forzar el orden de finalización deseado.</p>
<h1 id="ejecución-de-programas">Ejecución de programas</h1>
<h3 id="ejercicio-7">Ejercicio 7</h3>
<p>Escribir dos versiones, una con system(3) y otra con execvp(), de un programa que ejecute otro programa que se pasará como argumento por línea de comandos. En cada caso, se debe imprimir la cadena “El comando terminó de ejecutarse” después de la ejecución. ¿En qué casos se imprime la cadena? ¿Por qué?<br>
<strong>Nota:</strong> Considerar cómo deben pasarse los argumentos en cada caso para que sea sencilla la implementación. Por ejemplo: ¿qué diferencia hay entre ./ejecuta ps -el  y ./ejecuta “ps -el”?</p>
<h3 id="ejercicio-8">Ejercicio 8</h3>
<p>Usando la versión con execvp() del ejercicio 7 y la plantilla de demonio del ejercicio 6, escribir un programa que ejecute cualquier programa como si fuera un demonio. Además, redirigir los flujos estándar asociados al terminal usando dup2(2):</p>
<ul>
<li>
<p>La salida estándar al fichero /tmp/daemon.out.</p>
</li>
<li>
<p>La salida de error estándar al fichero /tmp/daemon.err.</p>
</li>
<li>
<p>La entrada estándar a /dev/null.</p>
</li>
</ul>
<p>Comprobar que el proceso sigue en ejecución tras cerrar la  <em>shell</em>.</p>
<h1 id="señales">Señales</h1>
<h3 id="ejercicio-9">Ejercicio 9</h3>
<p>El comando kill permite enviar señales a un proceso o grupo de procesos por su identificador (pkill permite hacerlo por nombre de proceso). Estudiar la página de manual del comando y las señales que se pueden enviar a un proceso.</p>
<h3 id="ejercicio-10">Ejercicio 10</h3>
<p>En un terminal, arrancar un proceso de larga duración (ej. sleep 600). En otra terminal, enviar diferentes señales al proceso, comprobar el comportamiento. Observar el código de salida del proceso. ¿Qué relación hay con la señal enviada?</p>
<h3 id="ejercicio-11">Ejercicio 11</h3>
<p>Escribir un programa que bloquee las señales SIGINT y SIGTSTP. Después de bloquearlas el programa debe suspender su ejecución con sleep(3) un número de segundos que se obtendrán de la variable de entorno SLEEP_SECS.</p>
<p>Después de despertar de sleep(3), el proceso debe informar de si recibió la señal SIGINT y/o SIGTSTP. En este último caso, debe desbloquearla con lo que el proceso se detendrá y podrá ser reanudado en la  <em>shell</em>  (imprimir una cadena antes de finalizar el programa para comprobar este comportamiento).</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;errno.h&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;stdio.h&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;stlib.h&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;sys/types.h&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;signal.h&gt;</span></span>

<span class="token keyword">int</span> <span class="token function">main</span><span class="token punctuation">(</span><span class="token keyword">int</span> argc<span class="token punctuation">,</span> <span class="token keyword">int</span> argv<span class="token operator">*</span><span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
	sigset_t blk<span class="token punctuation">,</span> pending_blk<span class="token punctuation">;</span>
	
	<span class="token function">sigemptyset</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>blk<span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token function">sigaddset</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>blk<span class="token punctuation">,</span> SIGINT<span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token function">sigaddset</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>blk<span class="token punctuation">,</span> SIGTSTP<span class="token punctuation">)</span><span class="token punctuation">;</span>

	<span class="token function">sigprocmask</span><span class="token punctuation">(</span>SIG_BLOCK<span class="token punctuation">,</span> <span class="token operator">&amp;</span>blk<span class="token punctuation">,</span> <span class="token constant">NULL</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	
	<span class="token keyword">char</span> <span class="token operator">*</span>sleep_sec_chr <span class="token operator">=</span> <span class="token function">getenv</span><span class="token punctuation">(</span><span class="token string">"SLEEP_SECS"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token keyword">int</span> sleep_sec <span class="token operator">=</span> <span class="token function">atoi</span><span class="token punctuation">(</span>sleep_sec_chr<span class="token punctuation">)</span><span class="token punctuation">;</span>
	
	<span class="token function">sleep</span><span class="token punctuation">(</span>sleep_sec<span class="token punctuation">)</span><span class="token punctuation">;</span>
	
	<span class="token function">sigpending</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>pending_blk<span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token keyword">if</span><span class="token punctuation">(</span><span class="token function">sigismember</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>pending_blk<span class="token punctuation">,</span> SIGINT<span class="token punctuation">)</span> <span class="token operator">==</span> <span class="token number">1</span> <span class="token operator">||</span> <span class="token function">sigismember</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>pending_blk<span class="token punctuation">,</span> SIGTSTP<span class="token punctuation">)</span> <span class="token operator">==</span> <span class="token number">1</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
		<span class="token function">printf</span><span class="token punctuation">(</span><span class="token string">"Se ha recibido la señal SIGINT o SIGTSTP"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token punctuation">}</span>
	<span class="token keyword">else</span> <span class="token punctuation">{</span>
		<span class="token function">printf</span><span class="token punctuation">(</span><span class="token string">"No se ha recibido la señal SIGINT o SIGTSTP"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token punctuation">}</span>
	<span class="token function">sigprocmask</span><span class="token punctuation">(</span>SIG_UNBLOCK<span class="token punctuation">,</span> <span class="token operator">&amp;</span>set<span class="token punctuation">,</span> <span class="token constant">NULL</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<h3 id="ejercicio-12">Ejercicio 12</h3>
<p>Escribir un programa que instale un manejador sencillo para las señales SIGINT y SIGTSTP. El manejador debe contar las veces que ha recibido cada señal. El programa principal permanecerá en un bucle que se detendrá cuando se hayan recibido 10 señales. El número de señales de cada tipo se mostrará al finalizar el programa.</p>
<h3 id="ejercicio-13">Ejercicio 13</h3>
<p>Escribir un programa que realice el borrado programado del propio ejecutable. El programa tendrá como argumento el número de segundos que esperará antes de borrar el fichero. El borrado del fichero se podrá detener si se recibe la señal SIGUSR1.</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;errno.h&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;stdio.h&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;stlib.h&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;sys/types.h&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;signal.h&gt;</span></span>

<span class="token macro property">#<span class="token directive keyword">define</span> errnoexit() do{printf("ERROR(%d): %s", errno, strerror(errno)); EXIT(EXIT_FAILURE);};while(0)</span>

<span class="token keyword">int</span> <span class="token function">main</span><span class="token punctuation">(</span><span class="token keyword">int</span> argc<span class="token punctuation">,</span> <span class="token keyword">int</span> argv<span class="token operator">*</span><span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
	sigset_t blk<span class="token punctuation">;</span>
	<span class="token keyword">struct</span> sigaction sigact<span class="token punctuation">;</span>

	
	<span class="token function">sigemptyset</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>blk<span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token function">sigaddset</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>blk<span class="token punctuation">,</span> SIGUSR1<span class="token punctuation">)</span><span class="token punctuation">;</span>
	
	<span class="token function">sigemptyset</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>sigact<span class="token punctuation">)</span><span class="token punctuation">;</span>
	sigact <span class="token operator">=</span> <span class="token punctuation">(</span>sigaction<span class="token punctuation">)</span> <span class="token punctuation">{</span><span class="token punctuation">.</span>sa_flags <span class="token operator">=</span> SA_SIGINFO<span class="token punctuation">}</span><span class="token punctuation">;</span>
	
	<span class="token keyword">int</span> sleep_sec <span class="token operator">=</span> <span class="token function">atoi</span><span class="token punctuation">(</span>argv<span class="token punctuation">[</span><span class="token number">2</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	
	<span class="token function">sleep</span><span class="token punctuation">(</span>sleep_sec<span class="token punctuation">)</span><span class="token punctuation">;</span>
	
	<span class="token keyword">if</span><span class="token punctuation">(</span><span class="token function">sigaction</span><span class="token punctuation">(</span>SIGUSR1<span class="token punctuation">,</span> <span class="token operator">&amp;</span>sigact<span class="token punctuation">,</span> <span class="token constant">NULL</span><span class="token punctuation">)</span> <span class="token operator">&gt;</span> <span class="token number">0</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
		<span class="token function">sigsuspend</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>blk<span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token punctuation">}</span>
	<span class="token keyword">else</span> <span class="token punctuation">{</span>
		<span class="token function">rmdir</span><span class="token punctuation">(</span>argv<span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre>
<p><strong>Nota:</strong> Usar sigsuspend(2) para suspender el proceso y la llamada al sistema apropiada para borrar el fichero.</p>

