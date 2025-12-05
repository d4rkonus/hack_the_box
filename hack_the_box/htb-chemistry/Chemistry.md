---------

Empezamos con la resolución de la máquina, haciendo un **ping** para comprobar si tenemos conexión:

![1](images/1.png)

Podemos ver que hay conexión, enviamos un paquete y recibimos un paquete.

El siguiente paso es **escanear los puertos abiertos** de la máquina víctima para ver por donde podemos entrar, para ello usamos _nmap_ para primero hacer un escaneo sencillo:

![2](images/2.png)

Podemos ver que los puertos abiertos son los **puertos 22 y 5000** que tienen los servicios **ssh y upnp**.

Me pareció extraño que no hubieran mas puertos, de todas formas, probé un escaneo más profundo sobre los puertos abiertos que tenemos:

![3](images/3.png)

Parece ser que el **puerto 5000** tiene el servicio **http** abierto, aunque no termino de entender porque en el otro escaneo tenía otro servicio asignado.

En este punto, puse la IP junto con el puerto de la máquina víctima en el navegador para ver a donde lleva:

![4](images/4.png)

![5](images/5.png)

La web permite analizar archivos **.CIF**, y en esta parte podemos iniciar sesión o crear un usuario, así que nos creamos un nuevo usuario:

![6](images/6.png)

Al ingresar con credenciales, podemos ver que tenemos un panel para subir archivos a la máquina, pero solo pueden ser **.CIF**: 

![7](images/7.png)

Podemos buscar en Internet por ejemplos de **archivos .CIF maliciosos** para poder seguir con la resolución de la máquina:

![8](images/8.png)

Podemos usar este repositorio como ejemplo:

![9](images/9.png)

Podremos ver este código que es el que usaremos:

![10](images/10.png)

El archivo contiene información tipo **CIF** que es un formato de datos usado en cristalografía, sin embargo hay una expresión de Python metida como texto, podemos usar este código de Python para seguir con la resolución: 

![12](images/12.png)

Podemos ver que el código usa la librería **os** para poder crear un archivo *pwned* , pero lo cambiaremos para que lance una reverse shell: 

![13](images/13.png)

***Nota: tuve problemas al momento de lanzar la reverse shell con la instrucción de arriba, la solución que encontré fue en poner la ruta absoluta en las 2 veces que se aparece 'bash'***

![16](images/16.png)

Nos ponemos en escucha por el puerto que usaremos para recibir la reverse shell: 

![14](images/14.png)

Y para recibir la shell, subimos el archivo modificado a la web: 

![15](images/15.png)

Ya estamos dentro: 

![17](images/17.png)

Una vez dentro, hacemos un **tratamiento de la tty**: 

```bash
script /dev/null -c bash
ctrl+z
stty raw -echo; fg
reset xterm
```

También cambiamos las dimensiones de la terminal, para que no se corte el texto:

![18](images/18.png)

![19](images/19.png)

En este punto estuve mirando el contenido del directorio actual, y encontré esto:

![21](images/21.png)

Hay un archivo **app.py** que solo el usuario actual puede usar, y entre todo el contenido que tiene, destacó esto: 

![22](images/22.png)

Parece ser que hay un archivos de  base de datos que usa **sqlite**, quise ver la ruta absoluta del archivo:

![23](images/23.png)

Aquí podemos ver que el archivo usa **SQlite 3**:

![24](images/24.png)

Podemos ver el contenido de la base de datos ya que la máquina tiene instalado **SQlite 3**:

![25](images/25.png)

Con  esto en mente, vamos a ver que encontramos dentro:

![26](images/26.png)

Una vez dentro, filtramos por las bases de datos:

![27](images/27.png)

Hay una tabla **user** que seguramente contiene las credenciales de los usuarios:

![28](images/28.png)

Hay unos 15 nombres de usuario junto con sus hashes de contraseñas, en este punto usé la web **Crackstation.net** para sacar la contraseñas en texto plano, pero primero tenemos que quedarnos solo con los hashes:

![30](images/30.png)

Ahora sí, vamos a la web y ahí pegamos los hashes: 

![31](images/31.png)

Aunque son bastantes hashes, solo se pudieron sacar algunas contraseñas:

![32](images/32.png)

Estuve viendo los usuarios en el equipo para relacionarlos con las contraseñas, y pude ver que el usuario **rosa** puede tener la contraseña **unicorniosrosados**. 

![34](images/34.png)

Pues la contraseña funciona, ahora siendo el usuario **rosa**, ya podemos ver la flag: 

![36](images/36.png)

Ahora solo falta obtener la flag del usuario **root** del sistema, estuve probando diferentes métodos entre repositorios de Github, hasta que me dió por probar un **directory path traversal** usando curl en el propio localhost de la máquina: 

![41](images/41.png)

***Nota: el '--path-as-is' se añade para que el path no sea modificado de ninguna forma y se mande tal cual.***

En este ejemplo como prueba, apuntaba al **/etc/passwd** y podemos ver el contenido del archivo, ahora lo probé con la ruta de la flag del usuario **root**: 

![42](images/42.png)

![43](images/43.png)