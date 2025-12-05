-------------------

Empezamos con la resolución de la máquina, haciendo un **ping** para comprobar si tenemos conexión:

![1](images/1.png)

Podemos ver que hay conexión, enviamos un paquete y recibimos un paquete.

El siguiente paso es **escanear los puertos abiertos** de la máquina víctima para ver por donde podemos entrar, para ello usamos _nmap_ para primero hacer un escaneo sencillo:

![2](images/2.png)

Encontramos todos estos puertos abiertos, el más llamativo es el **puerto 445**, en este caso esta máquina es vulnerable al *Eternal Blue*:

![3](images/3.png)

En esta situación, usaremos el **Metasploit** para resolver la máquina:

![4](images/4.png)

Para lanzar el **Metasploit** usamos el comando: 

![5](images/5.png)

Si buscamos la vulnerabilidad *Eternal Blue* en la base de datos, veremos esto:

![6](images/6.png)

Encontraremos varios ejemplos y tipos de la misma vulnerabilidad, pero en este caso, elegimos el primer ejemplo:

![7](images/7.png)

Para seleccionar un exploit, usamos su índice, en este caso el índice es 0:

![8](images/8.png)

Ahora debemos configurarlo a un nivel básico para que no pueda funcionar al momento de lanzar el exploit, para eso usamos este comando:

![9](images/9.png)

![10](images/10.png)

Debemos configurar la **dirección IP** de la máquina víctima y **dirección IP** de nuestra máquina atacante:

![11](images/11.png)

![12](images/12.png)

Al ya tener todo esto configurado, podemos lanzar el exploit, usando este comando:

![13](images/13.png)

Recibiremos una sesión de **Meterpreter**, pero podemos obtener una terminal si ejecutamos el comando:

![14](images/14.png)

Ya podemos ver que recibimos una terminal siendo **Administrador** del sistema Windows:

![15](images/15.png)

Con estos permisos dentro del sistema, ya podemos ver las flags de la máquina: 

![16](images/16.png)

![17](images/17.png)