---
title: Write-up Pilgrimage [Easy] - Hack The Box (Incompleto)
date: 2023-11-24
categories: [Writeups, Hack The Box]
tags: [writeup, hackthebox, machines, easy, pentesting]
image: /assets/img/hackthebox/machines/pilgrimage/pilgrimage.png
image_link: false
---

Publicación del write-up de sujeto a las [guías generales](https://help.hackthebox.com/en/articles/5188925-streaming-writeups-walkthrough-guidelines) de Hack The Box.

# Write-up Pilgrimage \[Easy\] - Hack The Box (Incompleto)
\\
**Mi perfil de Hack The Box: [julianglez16](https://app.hackthebox.com/users/1084561)**

# Enumeración


### Puertos


![Untitled.png](/assets/img/hackthebox/machines/pilgrimage/Untitled.png)
![Untitled.png](/assets/img/hackthebox/machines/pilgrimage/Untitled%20(1).png)


### Directorios web

![Untitled.png](/assets/img/hackthebox/machines/pilgrimage/Untitled%20(2).png)


### Tecnologías web

![Untitled.png](/assets/img/hackthebox/machines/pilgrimage/Untitled%20(3).png)


### Funcionalidad web


![Untitled.png](/assets/img/hackthebox/machines/pilgrimage/Untitled%20(4).png)
![Untitled.png](/assets/img/hackthebox/machines/pilgrimage/Untitled%20(5).png)
![Untitled.png](/assets/img/hackthebox/machines/pilgrimage/Untitled%20(6).png)



Accediendo con una cuenta recién creada:

![Untitled.png](/assets/img/hackthebox/machines/pilgrimage/Untitled%20(7).png)


Subiendo una imagen:

![Untitled.png](/assets/img/hackthebox/machines/pilgrimage/Untitled%20(8).png)
![Untitled.png](/assets/img/hackthebox/machines/pilgrimage/Untitled%20(9).png)



### Análisis con BurpSuite


# Explotación


### Usuario


Siguiendo los pasos de este blog: [https://blog.pentesteracademy.com/mining-exposed-git-directory-in-3-simple-steps-b6cfaf80b89b](https://blog.pentesteracademy.com/mining-exposed-git-directory-in-3-simple-steps-b6cfaf80b89b)


![Untitled.png](/assets/img/hackthebox/machines/pilgrimage/Untitled%20(10).png)
![Untitled.png](/assets/img/hackthebox/machines/pilgrimage/Untitled%20(11).png)


Se puede ver cual es el proceso que se sigue al subir una imagen a la web. El binario que se utiliza se ha obtenido desde el git y se puede ver su versión.

![Untitled.png](/assets/img/hackthebox/machines/pilgrimage/Untitled%20(12).png)
![Untitled.png](/assets/img/hackthebox/machines/pilgrimage/Untitled%20(13).png)



La versión de este binario tiene una vulnerabilidad de LFI [https://github.com/Sybil-Scan/imagemagick-lfi-poc](https://github.com/Sybil-Scan/imagemagick-lfi-poc). Siguiendo los pasos de la PoC se pueden leer ficheros del sistema.


![Untitled.png](/assets/img/hackthebox/machines/pilgrimage/Untitled%20(14).png)



### Root
