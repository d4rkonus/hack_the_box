-----------

Empezamos con la resolución de la máquina, haciendo un **ping** para comprobar si tenemos conexión:

![1](images/1.png)

Podemos ver que hay conexión, enviamos un paquete y recibimos un paquete.

El siguiente paso es **escanear los puertos abiertos** de la máquina víctima para ver por donde podemos entrar, para ello usamos _nmap_ para primero hacer un escaneo sencillo:

![2](images/2.png)

Solo hay un puerto abierto, el **puerto 80** y tiene abierto el servicio **http**, me imagino que habrá una web corriendo por detrás, pero antes de mirar la web, vamos a escanear este puerto en profundidad para ver si encontramos algo:

![3](images/3.png)

No tenemos apenas información útil, así que vamos a ver la web:

![4](images/4.png)

![5](images/5.png)

Parece ser que la web tiene algo llamado **phpbash** que según la web, nos sirve para hacer pentesting.

En la misma web encontramos esta imagen:

![6](images/6.png)

Se supone que en esta ruta encontraremos un archivo **.php** que nos daría una bash. 

![7](images/7.png)

Decidí probar a ingresar a la URL, pero no encontré nada. 

Estaba seguro de que algo encontraría, así que probé a buscar archivos y/o directorios ocultos con **gobuster**: 

![8](images/8.png)

![9](images/9.png)

Encontré todos estos recursos ocultos, estuve mirando los recursos uno por uno y al final me quedé con el **/dev**:

![10](images/10.png)

Dentro de este recurso, encontré esto:

![11](images/11.png)

Al final si que había un archivo **phpbash**, al seleccionarlo, veremos esto:

![12](images/12.png)

El archivo nos ofrece una terminal directa en la web:

![13](images/13.png)

Ya de primeras, podemos ver que hay un usuario llamado **arrexel**: 

![14](images/14.png)

En este punto, me puse a hacer una reverse shell para usar la terminal desde la máquina Kali, para eso, primero me puse en escucha por un puerto:

![15](images/15.png)

Luego, ejecuté este comando desde la terminal de la web: 

![16](images/16.png)

Ya estamos dentro de la web: 

![17](images/17.png)

Una vez dentro hacemos el **tratamiento de la tty**, para que la terminal no explote: 

```bash
script /dev/null -c bash
ctrl+z
stty raw -echo; fg
reset xterm
export term=xterm
export shell=bash
```

Al tratado bien la tty, podemos ver la flag del usuario normal:

![19](images/19.png)

Podemos ver los permisos que tenemos en cuanto a ejecutar comandos con **sudo -l**: 

![20](images/20.png)

Podemos ejecutar cualquier comando dentro del usuario **scriptmanager**, así que con esto en mente, me puse una bash como este mismo usuario: 

![21](images/21.png)

![22](images/22.png)

Con este paso hecho, me puse a buscar archivos o binarios que pueda usar siendo **scriptmanager**, para ver si podía ejecutar algo con mayor permisos dentro del sistema:

![23](images/23.png)

![24](images/24.png)

Encontré una carpeta llamada **/scripts** con un archivo de Python dentro, me dirigí a esa carpeta, y encontré todo esto:

![25](images/25.png)

Siendo el usuario **scriptmanager** puedo modificar el archivo **test.py** y además hay otro archivo **test.txt** que solo root puede modificar.

Tras investigar por Internet, encontré esta lista de comandos que siendo ejecutados uno por uno, conseguimos la flag del root: 

```bash
echo import os > test.py
echo 'os.system("cat /root/root.txt > /tmp/flag4u.txt)' >> test.py
cat test.py
```

![27](images/27.png)

Simplemente usando Python para ingresar unos comandos a nivel de sistema para que se copie el contenido del archivo de la flag de root a un archivo dentro del directorio **/tmp**. 

![28](images/28.png)


![29](images/29.png)