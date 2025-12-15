---------

Empezamos con la resolución de la máquina, haciendo un **ping** para comprobar si tenemos conexión:

![[1.png]]

Podemos ver que hay conexión, enviamos un paquete y recibimos un paquete.

El siguiente paso es **escanear los puertos abiertos** de la máquina víctima para ver por donde podemos entrar, para ello usamos _nmap_ para primero hacer un escaneo sencillo:

![[2.png]]

Solo pude ver un puerto activo, el **puerto 22** que tiene un servicio muy raro activo, decidí volver a hacer otro escaneo con más profundidad para ver si podía rascar algo nuevo:

![[3.png]]

Esto ya tiene más sentido, resulta que el **puerto 22** tiene abierto el servicio **ssh**, y hay un nuevo **puerto 5000** que tiene abierto el servicio **http**, es decir, hay una web corriendo por detrás. 

Con lo que hemos visto, podemos acceder a la web para ver que contiene: 

![[6.png]]

Al acceder a la web, podemos ver que hay un editor de código para el lenguaje ***Python***: 

![[7.png]]

Además de que podemos ver el output del código cuando se ejecuta:

![[8.png]]

Al saber que la máquina tiene Python instalado, decidí probar a ejecutar este código a nivel de sistema, para ver que recibía:

![[9.png]]

Parece ser que la máquina no permite ejecutar ciertas instrucciones de Python, será difícil seguir, pero no imposible:

![[10.png]]

Aunque no podamos ejecutar instrucciones a nivel de sistema de forma directa, podremos *camuflarlas* para que parezcan otra cosa:

![[11.png]]

Usé este código para ver si podía *camuflar* las funciones de Python:

```python
test = getattr(print.__self__, '__im' + 'port__')('o' + 's')
getattr(test, 'sy' + 'stem')('whoami')
```

Podemos ver que los nombres de las funciones están escritos como texto dividido en partes, y Python las une automáticamente.  
Luego, *print.__self__* se usa para acceder al núcleo de Python (donde viven las funciones internas).  
Con *getattr*, se obtiene una referencia a esas funciones usando su nombre en texto.  
Finalmente, esas funciones se ejecutan, y el resultado del primer paso (el módulo *os*) se guarda en la variable *test*.

Si ejecutamos el código en la web, veremos esto como resultado:

![[14.png]]

No hay nada, pero en teoría el código debería ejecutarse aunque no se vea nada, para eso modificamos el código para que lance *pings* hacia nuestra máquina:

![[16.png]]

Y si nos ponemos en escucha para recibir paquetes, veremos esto:

![[17.png]]

Podemos ver que el *camuflar* el código funciona porque las funciones se ejecutan, así que vamos a modificar el código para que nos envíe una *reverse shell*::

![[19.png]]

Al ponernos en escucha por el puerto, veremos esto:

![[18.png]]

![[20.png]]

Una vez dentro, hacemos tratamiento de la tty:

```bash
script /dev/null -c bash
ctrl +z
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```

Otra cosa que haremos será cambiar las dimensiones de la terminal:

![[21.png]]

Al terminar, podemos ir al directorio */home* y veremos que hay un usuario con el nombre Martín, aunque no podemos acceder a él:

![[23.png]]

Aunque en el directorio *app-production* podemos ver que dentro de encuentra la primera flag:

![[24.png]]

Ahora hay que buscar formas de tratar de convertirse en usuario del sistema, en este caso en Martin.

Estuve buscando mas datos en el mismo directorio donde encontré la flag, pero no había nada.

Decidí ir a al directorio */app* y encontré una directorio interesante llamado **instance**: 

![[25.png]]

Una vez dentro, veremos este archivo de bases de datos tipo sqlite:

![[26.png]]

Este archivo debe tener las credenciales de algún usuario del sistema, así que vamos a ver que contiene:

![[28.png]]

Filtramos por *.tables*:

![[30.png]]

Hay una tabla *user* que seguramente tendrá credenciales de usuario:

![[31.png]]

Hay una contraseña en formato de hash que pertenece al usuario Martín, así que nos lo copiamos hacia nuestra máquina atacante:

![[32.png]]

Podemos usar la web **crackstation.net** para crackear el hash y sacar la contraseña:

![[33.png]]

Esta es la contraseña en texto plano:

![[35.png]]

Y con esto ya podemos acceder como usuario martín: 

![[36.png]]

Si miramos los archivos que podemos ejecutar como *root*, veremos esto:

![[37.png]]

Hay un archivo llamado **backy.sh** y el cual este es su contenido:

```bash
#!/bin/bash

if [[ $# -ne 1 ]]; then
    /usr/bin/echo "Usage: $0 <task.json>"
    exit 1
fi

json_file="$1"

if [[ ! -f "$json_file" ]]; then
    /usr/bin/echo "Error: File '$json_file' not found."
    exit 1
fi

allowed_paths=("/var/" "/home/")

updated_json=$(/usr/bin/jq '.directories_to_archive |= map(gsub("\\.\\./"; ""))' "$json_file")

/usr/bin/echo "$updated_json" > "$json_file"

directories_to_archive=$(/usr/bin/echo "$updated_json" | /usr/bin/jq -r '.directories_to_archive[]')

is_allowed_path() {
    local path="$1"
    for allowed_path in "${allowed_paths[@]}"; do
        if [[ "$path" == $allowed_path* ]]; then
            return 0
        fi
    done
    return 1
}

for dir in $directories_to_archive; do
    if ! is_allowed_path "$dir"; then
        /usr/bin/echo "Error: $dir is not allowed. Only directories under /var/ and /home/ are allowed."
        exit 1
    fi
done

/usr/bin/backy "$json_file"

```

Estuve revisando el script y pude ver que en esta parte borra los *.* y */* para tratar eliminar los caracteres extra si se incluyen al acceder a archivos:

```bash
updated_json=$(/usr/bin/jq '.directories_to_archive |= map(gsub("\\.\\./"; ""))' "$json_file")
```

Ya sabemos que tenemos que tratar con este problema aunque no directamente de este archivo. 

Mirando más directorios, pude ver que en */backups* hay un archivo *.json*:

![[38.png]]

Y tenemos permisos para modificarlo:

![[39.png]]

Al acceder al código, veremos que hay unas rutas definidas:

![[41.png]]

Lo que hice fue modificar el código entero sin muchos cambios para que cause problemas:

![[49.png]]

Lo que sea que haga el script lo guardará en */tmp* y el directorio hacia el cuál accederá sera en */home/....//root*. 

Si ejecutamos el archivo **backy.sh** y le pasamos el archivo json, veremos esto:

![[50.png]]

Nos dirigimos al directorio */tmp* para que hay: 

![[51.png]]

Podemos ver que hay un comprimido, así que lo descomprimimos y veremos esto:

![[52.png]]

Parece que el script hizo una copia del directorio root y lo guardó en esta ruta: 

![[53.png]]

Aquí podemos todo su contenido, incluido la flag:


