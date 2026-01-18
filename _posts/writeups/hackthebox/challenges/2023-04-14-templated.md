---
title: Write-up Templated [CTF] [Easy] - Hack The Box
date: 2023-04-14
categories: [Writeups, Hack The Box]
tags: [writeup, hackthebox, challenges, easy, ctf, web]
image: /assets/img/hackthebox/challenges/templated/templated.png
image_link: false
---

Publicación del write-up de sujeto a las [guías generales](https://help.hackthebox.com/en/articles/5188925-streaming-writeups-walkthrough-guidelines) de Hack The Box.

# Write-up Templated \[CTF\] \[Easy\] - Hack The Box
\\
**Mi perfil de Hack The Box: [julianglez16](https://app.hackthebox.com/users/1084561)**


![Untitled.png](/assets/img/hackthebox/challenges/templated/Untitled.png)


Al buscar algún directorio web, como por ejemplo el fichero _robots.txt_, se obtiene un error HTTP 404 con un HTML en el que se muestra un mensaje con el fichero que se acaba de intentar obtener.

![Untitled.png](/assets/img/hackthebox/challenges/templated/Untitled%20(1).png)



Como se trata de un servidor Flask/Jinja2 y se observa que nuestra búsqueda se muestra en la plantilla HTML se intenta un ataque SSTI con el payload {% raw %} `{{ 7 * 7 }}` {% endraw %}, obteniendo el resultado 49, por lo que se determina vulnerable.

![Untitled.png](/assets/img/hackthebox/challenges/templated/Untitled%20(2).png)


Para obtener las clases disponibles se utiliza el payload {% raw %}`{{ ().__class__.__base__.__subclasses__() }}`{% endraw %}. El objetivo es utilizar la clase `<class 'subprocess.Popen'>` para poder ejecutar comandos en la máquina.

![Untitled.png](/assets/img/hackthebox/challenges/templated/Untitled%20(3).png)



Con el payload {% raw %}`{{ ''.__class__.__mro__[1].__subclasses__()[414]('ls -a',shell=True,stdout=-1).communicate()[0].strip() }}`{% endraw %} se consigue ejecutar el comando `ls -a` en la máquina obteniendo el nombre del fichero que contiene la flag.

![Untitled.png](/assets/img/hackthebox/challenges/templated/Untitled%20(4).png)



Finalmente, se utiliza el mismo payload con el comando `cat flag.txt` para obtener la flag

![Untitled.png](/assets/img/hackthebox/challenges/templated/Untitled%20(5).png)

