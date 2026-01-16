---
title: Write-up CozyHosting [Easy] - Hack The Box (Incompleto)
date: 2023-09-20
categories: [Writeups, Hack The Box]
tags: [writeup, hackthebox, machines, easy, pentesting]
image: /assets/img/hackthebox/machines/cozyhosting/cozyhosting.png
image_link: false
---

Publicación del write-up de sujeto a las [guías generales](https://help.hackthebox.com/en/articles/5188925-streaming-writeups-walkthrough-guidelines) de Hack The Box.

# Write-up CozyHosting \[Easy\] - Hack The Box (Incompleto)
\\
**Mi perfil de Hack The Box: [julianglez16](https://app.hackthebox.com/users/1084561)**



# Enumeración


### Puertos

![Untitled.png](/assets/img/hackthebox/machines/cozyhosting/Untitled%20(12).png)
![Untitled.png](/assets/img/hackthebox/machines/cozyhosting/Untitled.png)


### Directorios web


![Untitled.png](/assets/img/hackthebox/machines/cozyhosting/Untitled%20(1).png)


### Tecnologías web


![Untitled.png](/assets/img/hackthebox/machines/cozyhosting/Untitled%20(2).png)



### Funcionalidad web


![Untitled.png](/assets/img/hackthebox/machines/cozyhosting/Untitled%20(3).png)
![Untitled.png](/assets/img/hackthebox/machines/cozyhosting/Untitled%20(4).png)

Disparando un error en la web se conoce que existe un SpringBoot como servidor backend


![Untitled.png](/assets/img/hackthebox/machines/cozyhosting/Untitled%20(5).png)



# Explotación


### Usuario


Utilizando `ffuf` con un diccionario específico de SpringBoot se pueden obtener directorios relevantes.


![Untitled.png](/assets/img/hackthebox/machines/cozyhosting/Untitled%20(6).png)



Accediendo al panel `actuator/sessions` se observan las diferentes sesiones de los usuarios de la web. Obteniendo la del usuario `kanderson` y utilizándola para acceder al panel `/admin` se obtiene acceso al mismo.


![Untitled.png](/assets/img/hackthebox/machines/cozyhosting/Untitled%20(7).png)

![Untitled.png](/assets/img/hackthebox/machines/cozyhosting/Untitled%20(8).png)

![Untitled.png](/assets/img/hackthebox/machines/cozyhosting/Untitled%20(9).png)

Al realizar la petición de ssh se observa la siguiente petición desde BurpSuite


![Untitled.png](/assets/img/hackthebox/machines/cozyhosting/Untitled%20(10).png)



Suponiendo el siguiente comando en la máquina objetivo:


```bash
ssh -i .ssh/id_rsa [username]@[host]
```


Y que la validez del host es comprobada en el backend, se intenta una inyección en el usuario para obtener RCE.


![Untitled.png](/assets/img/hackthebox/machines/cozyhosting/Untitled%20(11).png)



### Root


ssh kanderson@localhost || whoami || ssh kanderson@localhost
