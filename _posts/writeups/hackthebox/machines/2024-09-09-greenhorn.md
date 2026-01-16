---
title: Write-up GreenHorn [Easy] - Hack The Box
date: 2024-09-09
categories: [Writeups, Hack The Box]
tags: [writeup, hackthebox, machines, easy, pentesting]
image: /assets/img/hackthebox/machines/greenhorn/greenhorn.png
image_link: false
---

Publicación del write-up de sujeto a las [guías generales](https://help.hackthebox.com/en/articles/5188925-streaming-writeups-walkthrough-guidelines) de Hack The Box.

# Write-up GreenHorn \[Easy\] - Hack The Box
\\
**Mi perfil de Hack The Box: [julianglez16](https://app.hackthebox.com/users/1084561)**


# Enumeración


### Puertos


![image.png](/assets/img/hackthebox/machines/greenhorn/image.png)


![image.png](/assets/img/hackthebox/machines/greenhorn/image%20(1).png)


### Tecnologías web


![image.png](/assets/img/hackthebox/machines/greenhorn/image%20(2).png)



### Funcionalidad web


![image.png](/assets/img/hackthebox/machines/greenhorn/image%20(3).png)
![image.png](/assets/img/hackthebox/machines/greenhorn/image%20(4).png)


![image.png](/assets/img/hackthebox/machines/greenhorn/image%20(5).png)
![image.png](/assets/img/hackthebox/machines/greenhorn/image%20(6).png)



### Análisis con BurpSuite


# Explotación


### Usuario


Explorando el código fuente de Pluck subido al repositorio GIT auto hosteado en el puerto 3000 se encuentra el fichero `pass.php` , el cual contiene la contraseña del administrador de la web hasheada en SHA512. Tras crackearla se obtiene la contraseña `iloveyou1`.


![image.png](/assets/img/hackthebox/machines/greenhorn/image%20(7).png)
![image.png](/assets/img/hackthebox/machines/greenhorn/image%20(8).png)



La versión de Pluck implementada en la web (v4.7.18) es vulnerable al CVE-2023-50564. Dicho CVE está relacionado con una vulnerabilidad de subida de archivos sin verificar al servidor. Utilizando la lógica del PoC encontrado en GitHub ([https://github.com/Rai2en/CVE-2023-50564_Pluck-v4.7.18_PoC](https://github.com/Rai2en/CVE-2023-50564_Pluck-v4.7.18_PoC)) se obtiene un reverse shell.

![image.png](/assets/img/hackthebox/machines/greenhorn/image%20(9).png)
![image.png](/assets/img/hackthebox/machines/greenhorn/image%20(10).png)



El usuario `junior`tiene configurada la misma contraseña, por lo que al realizar `su junior` e introducir `iloveyou1`como contraseña, obtendremos pivotar a dicho usuario.


![image.png](/assets/img/hackthebox/machines/greenhorn/image%20(11).png)


### Root


Para escalar privilegios al usuario root hay que hacer uso de la herramienta `pdfimages` en el fichero PDF que se encuentra en `/home/junior/` y despixelar la imagen con la herramienta [https://github.com/spipm/Depix](https://github.com/spipm/Depix) tras haber transfomado el fichero PDF a PPM.


![image.png](/assets/img/hackthebox/machines/greenhorn/image%20(12).png)
![image.png](/assets/img/hackthebox/machines/greenhorn/image%20(13).png)
![image.png](/assets/img/hackthebox/machines/greenhorn/image%20(14).png)

