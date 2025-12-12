
Empezamos con la resolución de la máquina, haciendo un **ping** para comprobar si tenemos conexión:

![1](images/1.png)

Podemos ver que hay conexión, enviamos un paquete y recibimos un paquete.

El siguiente paso es **escanear los puertos abiertos** de la máquina víctima para ver por donde podemos entrar, para ello usamos _nmap_ para primero hacer un escaneo sencillo:

![2](images/2.png)

Ya veo que hay solo **2 puertos abiertos** y son los 2 que más me gustan ya que el **puerto 80** corresponde al *http* y el **puerto 22** corresponde al *ssh*, es decir, hay una página web corriendo por detrás de la máquina y posiblemente podamos conectarnos de algún modo con la máquina usando **ssh**. 

Lo siguiente es escanear los puertos en más profundidad para ver si tienen datos que nos puedan servir de algo:

![3](images/3.png)

Ya podemos ver un dominio nuevo ***cozyhosting.htb***, antes de incluirlo al archivo */etc/hosts* quise probar a acceder a él, solo por ver que si resolvía bien: 

![4](images/4.png)

Como pensaba, no resuelve a nada, para resolver esto hay que añadir el dominio al archivo */etc/hosts*: 

![5](images/5.png)

Ahora sí, podemos ver el contenido de la web:

![6](images/6.png)

Parece ser algún tipo de hosting para empresas, si nos fijamos bien, veremos que hay una sección de **login**: 

![7](images/7.png)

![8](images/8.png)

Estuve probando con credenciales típicas y comunes, pero no funcionaron: 

![9](images/9.png)

Me dió por buscar por contenido oculto en la web con **gobuster**, a ver si lograba cazar algo: 

![10](images/10.png)

![11](images/11.png)

No encontré nada que mereciera la pena, pero intenté buscar por la sección de ***registro*** de la web ya que no la pude ver con **gobuster**: 

![12](images/12.png)

![13](images/13.png)

Al buscarla, me saltó este error, y como no tenía ni puta idea de que era, busqué información al respecto: 

![14](images/14.png)

Encontré esta publicación de Stack Overflow que habla sobre el error: 

![15](images/15.png)

Menciona algo llamado **Spring Boot Application**, lo primero que encontré fue esto: 

![16](images/16.png)

Un detalle que puede ser importante es que se habla del lenguaje **Java**, por ahora no es relevante pero puede serlo más adelante. 

Encontré este diccionario en **SecLists** que tiene contenido relacionado con el **Spring Boot**, y me dio por probar otra vez a buscar contenido oculto con **gobuster**: 

![17](images/17.png)

![18](images/18.png)

Todo este contenido es nuevo y podemos acceder a todo (en teoría), estuve mirando uno por uno para ver que encontraba: 

![19](images/19.png)

![20](images/20.png)

En el apartado de **sessions** encontré esto que parecen ser *cookies de sesión de usuarios*, esto me gusta verlo porque podremos hacer un robo de cookies de sesión para acceder a la web como un usuario propio sin tener que dar credenciales.

Para inyectar esta cookie, cree un inicio sesión con credenciales de prueba: 

![21](images/21.png)

Ninguna de estas credenciales es válida, porque lo que busco modificar no es esto, sino esto: 

![22](images/22.png)

Esta cookie que vemos, la cambiamos por la que vimos anteriormente: 

![23](images/23.png)

Y al recargar la página, veremos esto: 

![25](images/25.png)

Podemos ver tanto el contenido como el usuario con el que ingresamos: 

![26](images/26.png)

En esta misma sección de la web, encontramos esto: 

![27](images/27.png)

Es un panel para ingresar datos de un equipo con el que poder conectarse de alguna manera, probé a ingresar la IP de mi máquina atacante y nombre de usuario *kanderson*: 

![28](images/28.png)

Me puse en escucha por el **puerto 22** y ví esto: 

![29](images/29.png)

![30](images/30.png)

Parece ser que la máquina víctima trata de conectarse por **SSH** con la máquina atacante, pero la conexión no se completa. 

Una característica que se destaca por la máquina es la ***inyección de comandos***: 

![31](images/31.png)

Decidí probar otro tipo de solicitudes y probé esto: 

![32](images/32.png)

Acompañe el usuario *kanderson* con un comando con **curl** para hacer una petición a la máquina atacante por el puerto 80, en medio de la instrucción está **${IFS}** esto se conoce como **"Internal FIle Separator"**, su función es poner espacios entre texto que no se puede enviar con espacios visibles, gracias a esto podremos poner espacios en donde no se puede para que al enviar peticiones no nos devuelvan errores por sintaxis. 

Para recibir esa petición, nos abrimos un servidor temporal con Python: 

![33](images/33.png)

Y funciona: 

![34](images/34.png)

Si la petición ha funciona, probaremos con una **reverse shell**, empezamos creando el código de shell en un archivo ***index.html***: 

![35](images/35.png)

Este archivo lo moví a una ruta en concreto del sistema: 

![36](images/36.png)

Volví a poner un servidor con Python: 

![37](images/37.png)

Y me hice una petición con a mi mismo equipo, todo dentro de una misma ruta del sistema: 

![38](images/38.png)

Podemos ver que el **curl** busca el archivo ***index.html*** que tiene el código en bash para la reverse shell. 

Modificamos la instrucción de la web por esta misma: 

![39](images/39.png)

Si nos ponemos en escucha por el puerto dentro del código, recibiremos la reverse shell:

![40](images/40.png)

Una vez dentro, hacemos un **tratamiento de la tty** para que la terminal que tengamos no explote: 

```bash
script /dev/null -c bash
ctrl +z
stty raw -echo;fg
reset xterm
export SHELL=bash
export TERM=xterm
```

Una vez dentro, podemos ver el directorio del usuario no privilegiado del sistema, pero no podemos entrar (aún): 

![42](images/42.png)

Estuve buscando archivos con permisos **SUID**, pero no encontré nada útil: 

![44](images/44.png)


Hay un archivo en el directorio **/app**, y le saqué todo el contenido en la ruta ***'/tmp/app'*** :

![43](images/43.png)

![45](images/45.png)

Aquí podemos ver el contenido principal:

![46](images/46.png)

Estuve mirando los archivos y carpetas y terminé en este punto: 

![47](images/47.png)

En este archivo encontré credenciales importantes para resolver la máquina: 

![48](images/48.png)

Parece ser que la máquina tiene una base de datos **PostgreSQL**,, además de que podemos ver credenciales para acceder a la base de datos, tanto el usuario como la contraseña.

Accedemos a la base de datos con las credenciales: 

![49](images/49.png)

Lo primero es listar el contenido principal de la base de datos: 

![50](images/50.png)

Hay una base de datos llamada **cozyhosting**, para acceder, usamos este comando: 

![51](images/51.png)

Al listar las tablas, veremos esto: 

![52](images/52.png)

Listamos todo el contenido de la tabla **users**: 

![53](images/53.png)

Podemos ver hashes de contraseñas, esto es muy bueno porque podremos sacar la contraseña en texto plano. 

Nos copiamos el hash del usuario **admin** en un archivo local de la máquina atacante: 

![54](images/54.png)

Ahora usamos **JohnTheRipper** para sacar la contraseña: 

![55](images/55.png)

Ya con la contraseña en texto plano, podremos ingresar como usuario propio de la máquina, aunque antes revisé si los usuarios tienen una bash como shell por defecto: 

![57](images/57.png)

![58](images/58.png)

Encontré un nuevo usuario llamado **Josh** que tiene la bash por defecto, al igual que root.

Usé la contraseña anterior para ingresar como **Josh** y funcionó: 

![59](images/59.png)

En este punto, ya podemos ver la primera flag: 

![60](images/60.png)

Para escalar privilegios, estuve mirando por comandos que se pueden ejecutar como sudo, sin ser sudo: 

![61](images/61.png)

Parece ser que el binario de **ssh** tiene permisos de ser ejecutado como root del sistema, encontré un comando que nos puede servir para ser root usando **ssh**: 

![62](images/62.png)

![63](images/63.png)

Simplemente ejecutamos este comando tal cual y ya seremos root del sistema: 

```bash
sudo ssh -o ProxyCommand=';sh 0<&2 1>&2' x
```

![64](images/64.png)

![65](images/65.png)

