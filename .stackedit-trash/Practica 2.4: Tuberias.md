---


---

<h1 id="practica-2.4-tuberias">Practica 2.4: Tuberias</h1>
<h1 id="objetivos">Objetivos</h1>
<p>Las tuberías ofrecen un mecanismo sencillo y efectivo para la comunicación entre procesos en un mismo sistema. En esta práctica veremos los comandos e interfaz para la gestión de tuberías, y los patrones de comunicación típicos.</p>
<h1 id="tuberias-sin-nombre">Tuberias sin nombre</h1>
<p>Las tuberías sin nombre son entidades gestionadas directamente por el núcleo del sistema y son un mecanismo de comunicación unidireccional eficiente para procesos relacionados (padre-hijo). La forma de compartir los identificadores de la tubería es por herencia (en la llamada fork(2)).</p>
<h3 id="ejercicio-1">Ejercicio 1</h3>
<p>Escribir un programa que emule el comportamiento de la shell en la ejecución de una sentencia en la forma: comando1 argumento1 | comando2 argumento2. El programa creará una tubería sin nombre y creará un hijo:</p>
<ul>
<li>
<p>El proceso padre redireccionará la salida estándar al extremo de escritura de la tubería y ejecutará comando1 argumento1.</p>
</li>
<li>
<p>El proceso hijo redireccionará la entrada estándar al extremo de lectura de la tubería y ejecutará comando2 argumento2.</p>
</li>
</ul>
<p>Probar el funcionamiento con una sentencia similar a: ./ejercicio1 echo 12345 wc -c</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;errno.h&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;stdio.h&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;stlib.h&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;sys/types.h&gt;</span></span>

<span class="token macro property">#<span class="token directive keyword">define</span> NO_ERROR 0</span>
<span class="token macro property">#<span class="token directive keyword">define</span> ERROR -1</span>
<span class="token macro property">#<span class="token directive keyword">define</span> errorexit() do{ printf("ERROR(%d): %s\n", errno, sterror(errno));EXIT(EXIT_FAILURE);}; while(0)</span>
<span class="token macro property">#<span class="token directive keyword">define</span> MAX_SIZE 100</span>

<span class="token keyword">int</span> <span class="token function">main</span><span class="token punctuation">(</span><span class="token keyword">int</span> argc<span class="token punctuation">,</span> <span class="token keyword">int</span> argv<span class="token operator">*</span> <span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
	pid_t pid<span class="token punctuation">;</span>
	pid <span class="token operator">=</span> <span class="token function">fork</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token keyword">int</span> fd<span class="token punctuation">[</span><span class="token number">2</span><span class="token punctuation">]</span><span class="token punctuation">;</span> <span class="token comment">//tuberia: lectura[0], escritura[1]</span>
	<span class="token keyword">if</span><span class="token punctuation">(</span>argc <span class="token operator">!=</span> <span class="token number">5</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
		<span class="token function">printf</span><span class="token punctuation">(</span><span class="token string">"Usage: comando1 argumento1 | comando2 argumento2\n"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
		<span class="token function">EXIT</span><span class="token punctuation">(</span>EXIT_FAILURE<span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token punctuation">}</span>
	<span class="token keyword">if</span><span class="token punctuation">(</span>pid <span class="token operator">==</span> OK_FORK<span class="token punctuation">)</span> <span class="token punctuation">{</span>
		<span class="token function">close</span><span class="token punctuation">(</span>fd<span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
		<span class="token function">dup2</span><span class="token punctuation">(</span>fd<span class="token punctuation">[</span><span class="token number">0</span><span class="token punctuation">]</span><span class="token punctuation">,</span> <span class="token number">0</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
		<span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token function">execlp</span><span class="token punctuation">(</span>argv<span class="token punctuation">[</span><span class="token number">3</span><span class="token punctuation">]</span><span class="token punctuation">,</span> argv<span class="token punctuation">[</span><span class="token number">3</span><span class="token punctuation">]</span><span class="token punctuation">,</span> argv<span class="token punctuation">[</span><span class="token number">4</span><span class="token punctuation">]</span><span class="token punctuation">)</span> <span class="token operator">==</span> ERROR<span class="token punctuation">)</span> <span class="token punctuation">{</span>
			<span class="token function">errorexit</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
		<span class="token punctuation">}</span>
	<span class="token punctuation">}</span>
	<span class="token keyword">else</span> <span class="token keyword">if</span><span class="token punctuation">(</span>pid <span class="token operator">==</span> KO_FORK<span class="token punctuation">)</span> <span class="token punctuation">{</span>
		<span class="token function">errorexit</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token punctuation">}</span>
	<span class="token keyword">else</span> <span class="token punctuation">{</span>
	<span class="token comment">/*padre*/</span>
		<span class="token function">close</span><span class="token punctuation">(</span>fd<span class="token punctuation">[</span><span class="token number">0</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
		<span class="token function">dup2</span><span class="token punctuation">(</span>fd<span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">,</span> <span class="token number">1</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//redirecciona la salida</span>
		<span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token function">execlp</span><span class="token punctuation">(</span>argv<span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">,</span> argv<span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">,</span> argv<span class="token punctuation">[</span><span class="token number">2</span><span class="token punctuation">]</span><span class="token punctuation">)</span> <span class="token operator">==</span> ERROR<span class="token punctuation">)</span> <span class="token punctuation">{</span>
			<span class="token function">errorexit</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
		<span class="token punctuation">}</span>
	<span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre>
<p><strong>Nota:</strong> Antes de ejecutar el comando correspondiente, deben cerrarse todos los descriptores no necesarios.</p>
<h3 id="ejercicio-2">Ejercicio 2</h3>
<p>Para la comunicación bi-direccional, es necesario crear dos tuberías, una para cada sentido: p_h y h_p. Escribir un programa que implemente el mecanismo de sincronización de parada y espera:</p>
<ul>
<li>El padre leerá de la entrada estándar (terminal) y enviará el mensaje al proceso hijo, escribiéndolo en la tubería p_h. Entonces permanecerá bloqueado esperando la confirmación por parte del hijo en la otra tubería, h_p.</li>
<li>El hijo leerá de la tubería p_h, escribirá el mensaje por la salida estándar y esperará 1 segundo. Entonces, enviará el carácter ‘l’ al proceso padre, escribiéndolo en la tubería h_p, para indicar que está listo. Después de 10 mensajes enviará el carácter ‘q’ para indicar al padre que finalice.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;errno.h&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;stdio.h&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;stlib.h&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;sys/types.h&gt;</span></span>

<span class="token macro property">#<span class="token directive keyword">define</span> OK_FORK 0</span>
<span class="token macro property">#<span class="token directive keyword">define</span> KO_FORK -1</span>
<span class="token macro property">#<span class="token directive keyword">define</span> errorexit() do{ printf("ERROR(%d): %s\n", errno, sterror(errno));}; while(0)</span>
<span class="token macro property">#<span class="token directive keyword">define</span> MAX_SIZE 100</span>
<span class="token macro property">#<span class="token directive keyword">define</span> MAX_COUNT 10 </span><span class="token comment">/*valor maximo del contador de mensajes*/</span>

<span class="token keyword">int</span> <span class="token function">main</span><span class="token punctuation">(</span><span class="token keyword">int</span> argc<span class="token punctuation">,</span> <span class="token keyword">int</span> argv<span class="token operator">*</span><span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
	<span class="token keyword">int</span> p_h<span class="token punctuation">[</span><span class="token number">2</span><span class="token punctuation">]</span><span class="token punctuation">;</span> <span class="token comment">//padre -&gt; hijo:: p_escribe: 0, h_lee: 1</span>
	<span class="token keyword">int</span> h_p<span class="token punctuation">[</span><span class="token number">2</span><span class="token punctuation">]</span><span class="token punctuation">;</span> <span class="token comment">//hijo -&gt; padre:: h_escribre: 0, p_lee: 1</span>
	<span class="token keyword">int</span> readbytes<span class="token punctuation">,</span> count_msg <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">;</span>
	pid_t pid<span class="token punctuation">;</span>
	<span class="token keyword">char</span> chr<span class="token punctuation">[</span>MAX_SIZE<span class="token punctuation">]</span><span class="token punctuation">;</span>
	
	pid <span class="token operator">=</span> <span class="token function">fork</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token keyword">if</span><span class="token punctuation">(</span>pid <span class="token operator">==</span> KO_FORK<span class="token punctuation">)</span> <span class="token punctuation">{</span>
		<span class="token function">errorexit</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token punctuation">}</span>
	<span class="token keyword">else</span> <span class="token keyword">if</span><span class="token punctuation">(</span>pid <span class="token operator">==</span> OK_FORK<span class="token punctuation">)</span> <span class="token punctuation">{</span>
		<span class="token function">close</span><span class="token punctuation">(</span>h_p<span class="token punctuation">[</span><span class="token number">0</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//el padre no va a leer</span>
		<span class="token function">close</span><span class="token punctuation">(</span>p_h<span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//el padre no va a escribir</span>
		<span class="token keyword">while</span><span class="token punctuation">(</span><span class="token punctuation">(</span>readbytes <span class="token operator">=</span> <span class="token function">read</span><span class="token punctuation">(</span>p_h<span class="token punctuation">[</span><span class="token number">0</span><span class="token punctuation">]</span><span class="token punctuation">,</span> <span class="token operator">&amp;</span>chr<span class="token punctuation">,</span> <span class="token number">1</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token operator">&gt;</span> <span class="token number">0</span> <span class="token operator">&amp;&amp;</span> count_msg <span class="token operator">&lt;</span> MAX_COUNT<span class="token punctuation">)</span> <span class="token punctuation">{</span>
			<span class="token function">printf</span><span class="token punctuation">(</span><span class="token string">"%s"</span><span class="token punctuation">,</span> chr<span class="token punctuation">)</span><span class="token punctuation">;</span>
			<span class="token function">sleep</span><span class="token punctuation">(</span><span class="token number">1</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">/*1 segundo*/</span>
			<span class="token function">write</span><span class="token punctuation">(</span>h_p<span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">,</span> <span class="token string">"l"</span><span class="token punctuation">,</span> <span class="token keyword">sizeof</span><span class="token punctuation">(</span><span class="token keyword">char</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
			count_msg <span class="token operator">=</span> count_msg <span class="token operator">+</span> <span class="token number">1</span><span class="token punctuation">;</span>
		<span class="token punctuation">}</span>
		<span class="token function">printf</span><span class="token punctuation">(</span><span class="token string">"\n"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
		<span class="token function">write</span><span class="token punctuation">(</span>h_p<span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">,</span> <span class="token string">"q"</span><span class="token punctuation">,</span> <span class="token keyword">sizeof</span><span class="token punctuation">(</span><span class="token keyword">char</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
		<span class="token function">close</span><span class="token punctuation">(</span>h_p<span class="token punctuation">[</span><span class="token number">0</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span><span class="token comment">//el hijo no va a escribir</span>
		<span class="token function">close</span><span class="token punctuation">(</span>p_h<span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span><span class="token comment">//el hijo no va a leer</span>
		<span class="token function">EXIT</span><span class="token punctuation">(</span>EXIT_SUCCESS<span class="token punctuation">)</span><span class="token punctuation">;</span>
	<span class="token punctuation">}</span>
	<span class="token keyword">else</span> <span class="token punctuation">{</span>
		<span class="token function">close</span><span class="token punctuation">(</span>h_p<span class="token punctuation">[</span><span class="token number">0</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span><span class="token comment">//el hijo no va a escribir</span>
		<span class="token function">close</span><span class="token punctuation">(</span>p_h<span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span><span class="token comment">//el hijo no va a leer</span>
		<span class="token keyword">while</span><span class="token punctuation">(</span><span class="token punctuation">(</span>readbytes <span class="token operator">=</span> <span class="token function">read</span><span class="token punctuation">(</span>h_p<span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">,</span> <span class="token operator">&amp;</span>chr<span class="token punctuation">,</span> <span class="token number">1</span><span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token operator">&gt;</span> <span class="token number">0</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
			<span class="token function">scanf</span><span class="token punctuation">(</span><span class="token string">"%c"</span><span class="token punctuation">,</span> <span class="token operator">&amp;</span>chr<span class="token punctuation">)</span><span class="token punctuation">;</span>
			<span class="token function">write</span><span class="token punctuation">(</span>p_h<span class="token punctuation">[</span><span class="token number">0</span><span class="token punctuation">]</span><span class="token punctuation">,</span> chr<span class="token punctuation">,</span> <span class="token keyword">sizeof</span><span class="token punctuation">(</span>chr<span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
			<span class="token keyword">if</span><span class="token punctuation">(</span>chr <span class="token operator">==</span> <span class="token string">'q'</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
				<span class="token function">EXIT</span><span class="token punctuation">(</span>EXIT_SUCCESS<span class="token punctuation">)</span><span class="token punctuation">;</span>
			<span class="token punctuation">}</span> 
		<span class="token punctuation">}</span>
		<span class="token function">waitpid</span><span class="token punctuation">(</span>pid<span class="token punctuation">,</span> <span class="token constant">NULL</span><span class="token punctuation">,</span> <span class="token number">0</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
		<span class="token function">close</span><span class="token punctuation">(</span>h_p<span class="token punctuation">[</span><span class="token number">0</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//el padre no va a leer</span>
		<span class="token function">close</span><span class="token punctuation">(</span>p_h<span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">//el padre no va a escribir</span>
	<span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre>
<h1 id="tuberias-con-nombre">Tuberias con nombre</h1>
<p>Las tuberías con nombre son un mecanismo de comunicación unidireccional, con acceso de tipo FIFO, útil para procesos sin relación de parentesco. La gestión de las tuberías con nombre es igual a la de un archivo ordinario (open, write, read…). Revisar la información en fifo(7).</p>
<h3 id="ejercicio-3">Ejercicio 3</h3>
<p>Usar la orden mkfifo para crear una tubería con nombre. Usar las herramientas del sistema de ficheros (stat, ls…) para determinar sus propiedades. Comprobar su funcionamiento usando utilidades para escribir y leer de ficheros (ej. echo, cat, tee…).</p>
<pre class=" language-c"><code class="prism  language-c"></code></pre>
<h3 id="ejercicio-4">Ejercicio 4</h3>
<p>Escribir un programa que abra la tubería con el nombre anterior en modo sólo escritura, y escriba en ella el primer argumento del programa. En otro terminal, leer de la tubería usando un comando adecuado.</p>
<pre class=" language-c"><code class="prism  language-c"></code></pre>
<h1 id="multiplexación-síncrona-de-entradasalida">Multiplexación síncrona de entrada/salida</h1>
<p>Es habitual que un proceso lea o escriba de diferentes flujos. La llamada select(2) permite multiplexar las diferentes operaciones de E/S sobre múltiples flujos.</p>
<h3 id="ejercicio-5">Ejercicio 5</h3>
<p>Crear otra tubería con nombre. Escribir un programa que espere hasta que haya datos listos para leer en alguna de ellas. El programa debe mostrar la tubería desde la que leyó y los datos leídos. Consideraciones:</p>
<ul>
<li>
<p>Para optimizar las operaciones de lectura usar un  <em>buffer</em>  (ej. de 256 bytes).</p>
</li>
<li>
<p>Usar read(2) para leer de la tubería y gestionar adecuadamente la longitud de los datos leídos.</p>
</li>
<li>
<p>Normalmente, la apertura de la tubería para lectura se bloqueará hasta que se abra para escritura (ej. con echo 1 &gt; tuberia). Para evitarlo, usar la opción O_NONBLOCK en open(2).</p>
</li>
<li>
<p>Cuando el escritor termina y cierra la tubería, read(2) devolverá 0, indicando el fin de fichero, por lo que hay que cerrar la tubería y volver a abrirla. Si no, select(2) considerará el descriptor siempre listo para lectura y no se bloqueará.</p>
</li>
</ul>
<pre class=" language-c"><code class="prism  language-c">
</code></pre>

<!--stackedit_data:
eyJoaXN0b3J5IjpbMjAyOTczNjU0Ml19
-->