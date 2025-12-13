--------

Empezamos con la resolución de la máquina, haciendo un **ping** para comprobar si tenemos conexión:

![1](images/1.png)

Podemos ver que hay conexión, enviamos un paquete y recibimos un paquete.

El siguiente paso es **escanear los puertos abiertos** de la máquina víctima para ver por donde podemos entrar, para ello usamos _nmap_ para primero hacer un escaneo sencillo:

![2](images/2.png)

Podemos ver que hay **3 puertos abiertos** y me llama mucho la atención que los servicios *ssh* y *ftp* se encuentran activos ya que podríamos usarlos para acceder a la máquina o transferir archivos. 

Para tener más información de los puertos abiertos, decidí hacer otro escaneo con más profundidad:

![3](images/3.png)

Como no encontré nada útil, al menos a primera vista, decidí copiar la IP de la máquina víctima en el navegador para ver a donde lleva:

![4](images/4.png)

Al acceder a la web, veremos esto:

![5](images/5.png)

En una esquina podremos ver que somos el usuario **Nathan**:

![6](images/6.png)

Podremos ver este panel de las diferentes secciones que tiene la web, parece ser que la web está relacionada con la seguridad de paquetes a nivel de red (o algo así xd):

![7](images/7.png)

Si elegimos la opción de **Security Snapshot** veremos esto:

![8](images/8.png)

Parece ser una captura de seguridad que indican el número de paquetes capturados, además del tipo de paquete.

Y un detalle curioso es que al cambiar de sección de la web, la *URL* cambia y se incluye un número al final de la misma. 

![9](images/9.png)

También sucede que si el número cambia siendo su valor superior a **1**, el número de paquetes no cambia o apenas aumenta su contenido:

![10](images/10.png)

Lo que hice para proceder con la resolución de la máquina fue aprovechar un ***IDOR*** para cambiar el número del ID a **0**, en teoría no tendríamos que tener permisos para hacerlo, pero decidí intentarlo de todas formas: 

![11](images/11.png)

No solo funcionó sino que además apareció una captura de paquetes diferente y con muy buena pinta: 

![12](images/12.png)

La web nos permite descargar la captura:

![13](images/13.png)

Cuando tengamos el archivo descargado, podemos analizarlo con **Wireshark** ya que contiene datos de paquetes a nivel de red:

![14](images/14.png)

![15](images/15.png)

De primeras, nos encontramos con puro relleno que no tiene nada útil:

![16](images/16.png)

Pero analizando mejor el contenido, veremos una captura que muestra al usuario **Nathan** tratando de iniciar sesión en la máquina, y tenemos la contraseña en texto plano en la propia captura:

![17](images/17.png)

Con las credenciales en mano, ya podemos iniciar sesión con este comando:

```bash
ssh nathan@10.129.13.87
```

![18](images/18.png)

Para evitar que la terminal reviente, ejecutamos estos comandos:

```bash
export SHELL=bash
export TERM=xterm
```

![19](images/19.png)

Ahora siendo usuario del sistema, podemos ver la primera flag:

![20](images/20.png)

Nos queda la parte de escalar privilegios, así que me puse a buscar archivos con permisos SUID:

![21](images/21.png)

No encontré nada, así que me dio por buscar si la máquina tiene **Python** instalado:

![22](images/22.png)

Y confirmo que **Python** estaba instalado en la máquina, traté de usar **Python** para escalar privilegios de esta forma: 

- Lo primero es ejecutar el intérprete usando la ruta absoluta:

![23](images/23.png)

- Una vez dentro importamos la librería *os* para interactuar con el sistema operativo, lo siguiente es establecer nuestro *id de usuario* a *0* que corresponde al usuario root, y por último nos lanzamos una bash: 

![24](images/24.png)

Ahora, ya somos usuario root del sistema:

![25](images/25.png)

Aquí tenemos la última flag: 

![26](images/26.png)

