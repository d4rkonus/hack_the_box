---------

Empezamos con la resolución de la máquina, haciendo un **ping** para comprobar si tenemos conexión: 

![Ping a la máquina](images/1.png)

Podemos ver que hay conexión, enviamos un paquete y recibimos un paquete. 

El siguiente paso es **escanear los puertos abiertos** de la máquina víctima para ver por donde podemos entrar, para ello usamos *nmap* para primero hacer un escaneo sencillo:

![[2.png]]

No hay apenas puertos abiertos, me parece curioso que el **puerto 25565** tiene un servicio llamado **minecraft**, quitando eso, no hay nada interesante.

Decidí hacer un escaneo aún más profundo para ver que podía encontrar: 

![[3.png]]

![[4.png]]

Encontré que hay un nombre de dominio **blocky.htb**, además también traté de ingresar a la máquina mediante *ssh*: 

![[5.png]]

Sin embargo, no conseguí iniciar sesión usando este método.

Al ver que tenía un nombre de dominio, lo añadí al archivo **/etc/hosts** para ver a donde me llevaría: 

![[6.png]]

Si nos vamos al navegador, veremos esto: 

![[7.png]]

Estuve unos minutos buscando la opción para descargar Minecraft, pero no pude, así que me puse a revisar la web para ver si encontraba algo:

![[8.png]]

Ya podemos ver algo interesante, y es que esta web está hecha con **WordPress**. 

Estaba seguro de que dentro de WordPress encontraría archivos interesantes, así que me puse a buscar con Wfuzz, porque Gobuster me daba problemas:

```bash
wfuzz -c --hc=404 -t 200 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt http://blocky.htb/FUZZ
```

Obtuve una buena cantidad de datos:

![[9.png]]

Pero de entre todos, me llamo la atención este de aquí: 

![[10.png]]

Copié el nombre en la URL de la web,  y encontré esto: 

![[11.png]]

![[12.png]]

Dentro de este recurso **plugins**, habían 2 archivos, para comprobar, decidí descargarme el **BlockyCore.jar**: 

![[13.png]]

Parece ser el archivo es como un java pero comprimido, aunque no exactamente así.  

Para ver el contenido de este archivo, es necesario tener una herramienta llamada "*jd-gui* ":

![[14.png]]

Una vez dentro del archivo, podemos ver que los datos son puro texto plano: 

![[15.png]]

Y con esto ya podemos ver que con el usuario root del sistema podemos acceder con la contraseña que vemos en los datos :) 

Pero para poder llegar a root, primero entramos con el usuario normal, además de que podemos usar la misma contraseña del archivo para iniciar sesión como usuario normal: 

![[16.png]]

![[17.png]]


Aquí ya podemos ver la **primera flag del sistema**: 

![[18.png]]

Para escalar privilegios, tan simple como usar la misma contraseña que hemos usado para entrar al sistema como usuario normal:

![[21.png]]

Aquí ya podemos ver la **segunda flag del sistema**: 

![[22.png]]


![[23.png]]
