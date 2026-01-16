---
title: Write-up Analytics [Easy] - Hack The Box
date: 2023-10-17
categories: [Writeups, Hack The Box]
tags: [writeup, hackthebox, machines, easy, pentesting]
image: /assets/img/hackthebox/machines/analytics/analytics.png
image_link: false
---

Publicación del write-up de sujeto a las [guías generales](https://help.hackthebox.com/en/articles/5188925-streaming-writeups-walkthrough-guidelines) de Hack The Box.

# Write-up Analytics \[Easy\] - Hack The Box 
\\
**Mi perfil de Hack The Box: [julianglez16](https://app.hackthebox.com/users/1084561)**

## Analytics

# Enumeración


### Puertos


![Untitled.png](/assets/img/hackthebox/machines/analytics/Untitled.png)

![Untitled.png](/assets/img/hackthebox/machines/analytics/Untitled%20(1).png)



### Directorios web


![Untitled.png](/assets/img/hackthebox/machines/analytics/Untitled%20(2).png)



### Tecnologías web


![Untitled.png](/assets/img/hackthebox/machines/analytics/Untitled%20(3).png)
![Untitled.png](/assets/img/hackthebox/machines/analytics/Untitled%20(4).png)


### Funcionalidad web


![Untitled.png](/assets/img/hackthebox/machines/analytics/Untitled%20(5).png)
![Untitled.png](/assets/img/hackthebox/machines/analytics/Untitled%20(6).png)
![Untitled.png](/assets/img/hackthebox/machines/analytics/Untitled%20(7).png)


### Análisis con BurpSuite


La petición del login en Metabase se envía en formato JSON, visible en la imagen.


![Untitled.png](/assets/img/hackthebox/machines/analytics/Untitled%20(8).png)
![Untitled.png](/assets/img/hackthebox/machines/analytics/Untitled%20(9).png)


# Explotación


### Usuario


Buscando por internet se averigua que Metabase fue vulnerable al [CVE-2023-38646](https://nvd.nist.gov/vuln/detail/CVE-2023-38646) en el que se puede realizar ejecución de comandos de manera remota sin estar autenticado (unauthenticated RCE). A través del token `setup-token` que se obtiene en las peticiones de la web y de una inyección SQL, se pueden ejecutar comandos de manera remota.


![Untitled.png](/assets/img/hackthebox/machines/analytics/Untitled%20(10).png)
![Untitled.png](/assets/img/hackthebox/machines/analytics/Untitled%20(11).png)



Se obtiene un reverse shell en la máquina.


![Untitled.png](/assets/img/hackthebox/machines/analytics/Untitled%20(12).png)



La máquina está corriendo un contenedor Docker 


![Untitled.png](/assets/img/hackthebox/machines/analytics/Untitled%20(13).png)



Dentro del contenedor, en las variables de entorno, se encuentran credenciales sensibles que nos permiten conectarnos por SSH.


![Untitled.png](/assets/img/hackthebox/machines/analytics/Untitled%20(14).png)


Haciendo ssh se consigue la flag del usuario.


![Untitled.png](/assets/img/hackthebox/machines/analytics/Untitled%20(15).png)



### Root


Mirando los permisos de SUID se puede ver que `/usr/bin/bash` tiene permisos SUID, por lo que se escala de privilegios fácilmente ejecutando `bash -p`.


![Untitled.png](/assets/img/hackthebox/machines/analytics/Untitled%20(16).png)
