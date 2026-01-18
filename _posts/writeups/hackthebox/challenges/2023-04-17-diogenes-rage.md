---
title: Write-up Diogene's Rage [CTF] [Easy] - Hack The Box
date: 2023-04-17
categories: [Writeups, Hack The Box]
tags: [writeup, hackthebox, challenges, easy, ctf, web]
image: /assets/img/hackthebox/challenges/diogenes rage/diogenesrage.png
image_link: false
---

Publicación del write-up de sujeto a las [guías generales](https://help.hackthebox.com/en/articles/5188925-streaming-writeups-walkthrough-guidelines) de Hack The Box.

# Write-up Diogene's Rage \[CTF\] \[Easy\] - Hack The Box
\\
**Mi perfil de Hack The Box: [julianglez16](https://app.hackthebox.com/users/1084561)**


![Página principal](/assets/img/hackthebox/challenges/diogenes%20rage/Untitled.png)


Al añadir un código de cupón o al intentar comprar un producto se asigna un JWT de sesión con un usuario recién creado en la base de datos.


![Al insertar el cupón](/assets/img/hackthebox/challenges/diogenes%20rage/Untitled%20(1).png)


![Compra de un producto](/assets/img/hackthebox/challenges/diogenes%20rage/Untitled%20(2).png)


![JWT del usuario](/assets/img/hackthebox/challenges/diogenes%20rage/Untitled%20(3).png)


![Al comprar un producto más caro de nuestro presupuesto](/assets/img/hackthebox/challenges/diogenes%20rage/Untitled%20(4).png)


Analizando el código javascript del servidor se observa que es vulnerable a un ataque de _Race Condition_ por lo que con el plugin Turbo Intruder de BurpSuite se generan peticiones concurrentes para realizar la petición de añadir un cupón a un usuario del que conozcamos su JWT


![Obtención del JWT del usuario](/assets/img/hackthebox/challenges/diogenes%20rage/Untitled%20(5).png)


![Turbo Intruder](/assets/img/hackthebox/challenges/diogenes%20rage/Untitled%20(6).png)


Si hay suerte con la velocidad de conexión se podrá obtener una cantidad de dinero suficiente para comprar el producto C8 y obtener la flag


![Untitled.png](/assets/img/hackthebox/challenges/diogenes%20rage/Untitled%20(7).png)
