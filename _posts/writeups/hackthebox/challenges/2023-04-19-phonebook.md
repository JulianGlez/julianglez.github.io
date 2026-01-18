---
title: Write-up Phonebook [CTF] [Easy] - Hack The Box
date: 2023-04-19
categories: [Writeups, Hack The Box]
tags: [writeup, hackthebox, challenges, easy, ctf, web]
image: /assets/img/hackthebox/challenges/phonebook/phonebook.png
image_link: false
---

Publicación del write-up de sujeto a las [guías generales](https://help.hackthebox.com/en/articles/5188925-streaming-writeups-walkthrough-guidelines) de Hack The Box.

# Write-up Phonebook \[CTF\] \[Easy\] - Hack The Box
\\
**Mi perfil de Hack The Box: [julianglez16](https://app.hackthebox.com/users/1084561)**


La página principal es un login que, según el mensaje que se observa, cuenta con autenticación contra un LDAP.


![Untitled.png](/assets/img/hackthebox/challenges/phonebook/Untitled.png)


Partiendo de esta suposición se puede intentar realizar un bypass del login con una _LDAP injection._ La inyección se realiza con un * en la contraseña ya que de esta manera se realizaría la siguiente petición contra el servidor LDAP:


```bash
(&(username=Reese)(password=*))
```

![Untitled.png](/assets/img/hackthebox/challenges/phonebook/Untitled%20(1).png)


![Untitled.png](/assets/img/hackthebox/challenges/phonebook/Untitled%20(2).png)



La inyección realizada tiene un resultado favorable así que, suponiendo que la contraseña del usuario `Reese` es la flag, se puede crear un script que mediante fuerza bruta obtenga la flag en base a las autenticaciones exitosas.

![Untitled.png](/assets/img/hackthebox/challenges/phonebook/Untitled%20(3).png)
![Untitled.png](/assets/img/hackthebox/challenges/phonebook/Untitled%20(4).png)

