---
title: Shimcache, el artefacto definitivo para (no) evidenciar ejecución
date: 2026-07-01 22:30:00 +0200
categories: [Blog, Forensics]
tags: [dfir, shimcache, blog]     # TAG names should always be lowercase
author: julian
description: 
# contents:
comments: false
image: /assets/img/blog/shimcache/Shimcache header.png
---

Durante el aprendizaje de DFIR en Windows, todo analista forense (o analista de ciberseguridad en general) aprende de la existencia de artefactos del Sistema Operativo que evidencian diferente actividad. En muchos casos, evidenciar la ejecución de binarios a través de estos artefactos es trivial (por ejemplo, a través de la existencia de un fichero Prefetch). No obstante, cuando en internet se habla de artefactos que evidencian ejecución, muchas veces se nombra a la Shimcache (o AppCompatCache) como indicativo categórico de evidencia de ejecución para aquellos binarios presentes en ella. Bien, pues la realidad es que si bien en un pasado sí era así, actualmente no lo es.

Razonando sobre esto, me pregunté lo siguiente: Si yo fuese perito informático, durante un proceso judicial ¿cómo le defiendo a un juez que Shimcache no evidencia ejecución? 

Para responder a esto, hay que conocer en profundidad la razón de la existencia de la Shimcache y cómo funciona.

# Qué es Shimcache
Antes de entrar en materia, creo que es importante conocer un poco más acerca de los elementos del Sistema Operativo relacionados con la Shimcache. ¿Qué es y para qué sirve un *shim*? ¿Cómo funcionan?

## Shim
Los *shims* en Windows nacen para solucionar el problema de compatibilidad de aplicaciones entre diferentes versiones de Windows. Como en cualquier ciclo de vida del Software, durante la implementación de mejoras y saltos de versiones en un sistema operativo puede provocar que existan funciones de la API de Windows que queden deprecadas, llevándose por delante todo software que haga uso de ellas. 

La Infraestructura de *shims* de Windows realiza un API hooking sobre la Import Address Table (IAT)[^IAT] de la aplicación (ver [Cómo se crean los shims](#cómo-se-crean-los-shims)). De esta manera, las llamadas a las funciones de la API de Windows son redirigidas al Shim asociado para la aplicación "Shimmeada". El *shim* de la aplicación es el encargado de manejar la llamada a la función correspondiente compatible de la API de Windows actualizada[^UnderstandingShims].

![Llamada de aplicación a API de Windows](/assets/img/blog/shimcache/IATcall.jpg)
*Llamada de aplicación a API de Windows*
![Llamada de aplicación shimmeada a API de Windows](/assets/img/blog/shimcache/IATcall_shimmed.jpg)
*Llamada de aplicación shimmeada a API de Windows*

## Cómo se crean los shims
Windows cuenta con un servicio llamado Servicio Asistente para la compatibilidad de programas (PcaSvc) el cual se encarga de inspeccionar las aplicaciones disponibles en el sistema y determinar si estas cuentan con problemas de compatiblidad que deban ser solucionados [^PcaSvc][^ProgramCompatibilityAssistant1].

El mapeo de shims que utilizará la aplicación existe en un fichero de base de datos de shims (Shim Database File), la cual por defecto se ubica en C:\Windows\apppatch\sysmain.sdb (esta base de datos contiene entradas preestablecidas por Windows, es decir, estas entradas están pensadas para software conocido que debe ser shimmeado). También se pueden crear bases de datos de shims personalizadas que pueden estar ubicadas en diferentes rutas del sistema. Además, para aquellos ejecutables que se shimmeen y no existan listados en ningún fichero SDB instalado, los shims aparecerán en la clave de registro ``HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\Layers``.

![Registro AppCompatFlags\Lagers](/assets/img/blog/shimcache/appcompaflags_layers.png)
*Registro AppCompatFlags\Lagers*

El contenido de los ficheros SDB almacena diferente información relativa al ejecutable shimmeado por el sistema[^ShimDatabaseFiles][^ShimDBformat]. Entre otras cosas, esta base de datos establece los shims que aplican a cada ejecutable shimmeado. Por ejemplo, para el ejecutable tarzan.exe se observa que existe un shim llamado "PaletteRestore". Este será el shim que aplique al binario durante su ejecución. Para este caso en concreto, "PaletteRestore" aplica una corrección sobre la paleta de colores utilizada por la aplicación forzando el uso correcto de la misma para la versión del sistema [^shim-research].

![Entrada de tarzan.exe en sysmain.sdb](/assets/img/blog/shimcache/shim_ref-tarzan.png)
*Entrada de tarzan.exe en sysmain.sdb inspeccionada con SDBExplorer[^EZTools]*

Los *shims* también pueden ser creados manualmente utilizando "Compatibility Administrator" disponible en la herramienta Windows Assessment and Deployment Kit (Windows ADK), seleccionando una aplicación y los shims que se quieran aplicar[^Demystifying-shims].

## Shimcache
Shimcache es un elemento que forma parte de la Infraestructura de Compatibilidad de Aplicaciones el cual registra las rutas de los ejecutables **evaluados** por dicha infraestructura y se encuentra almacenado en el registro ``HKLM\SYSTEM\CurrentControlSet\Control\SessionManager\AppCompatCache\AppCompatCache``. En versiones anteriores de Windows existía un flag que indicaba si dicha aplicación había sido ejecutada o no, pero actualmente no es así. El hecho de que una aplicación aparezca listada en la Shimcache no implica que haya sido ejecutada, sino que ha sido **referenciada** por la Infraestructura de Compatiblidad de Aplicaciones, y esto pudo haberse dado debido a que se lista en un directorio, se ha intentado ejecutar, la aplicación haya sido instalada, etc.

Existen hipótesis de que ciertos bytes (los últimos 4) de las entradas de Shimcache podrían indicar una ejecución para ciertos binarios si tienen un valor en concreto[^13cShimcache][^ShimcacheDeepDive]. No obstante, este comportamiento no se presenta ante todos los binarios ejecutados en el sistema. Es por esto, que este artefacto no puede ser tratado como una evidencia de ejecución ya que carece de consistencia en su estructura y no ofrece garantías de ejecución.

Adicionalmente, en 2024 Microsoft realizó una [publicación](https://www.microsoft.com/en-us/security/blog/2024/04/23/new-microsoft-incident-response-guide-helps-simplify-cyberthreat-investigations/?utm_source=chatgpt.com) donde aclaró conceptos relacionados con diferentes artefactos de Windows, entre ellos, Shimcache. 
> **Shimcache’s forensic evolution:** The Shimcache has long served as a source of forensic information, particularly as evidence of program execution. However, the changes in Windows 10 and later have significantly impacted the forensic meaning of Shimcache artifacts: indicating file presence, and not indicating execution. This misunderstanding can mislead investigators, especially since Shimcache logs the last modification timestamp, not execution time, and data is only committed to disk upon shutdown or reboot.

También lo indica explícitamente en esta [guía de respuesta a incidentes](https://cdn-dynmedia-1.microsoft.com/is/content/microsoftcorp/microsoft/final/en-us/microsoft-brand/documents/IR-Guidebook-Final.pdf)

## Conclusiones
La existencia de rutas de ejectuables en la Shimcache puede darse por mútliples motivos determinados por el servicio de Compatibilidad de Aplicaciones PcaSvc; desde que haya sido listado el ejecutable en un directorio hasta que este haya sido ejecutado. Adicionalmente, la incosistencia de la estructura de datos almacenados utilizada en las entradas de la Shimcache (útlimos 4 bytes) no permite establecer un veredicto seguro de la evidencia de ejecución del mismo. Además, Microsoft se ha pronunciado al respecto mediante guías de respuesta a incidentes indicando que Shimcache **no** evidencia ejecución, sino presencia.

A pesar de que no evidencie ejecución, Shimcache es un artefacto magnífico para evidenciar la existencia de un ejecutable en el sistema. Cuando un ejecutable se elimina, su entrada queda registrada hasta que la shimcache se llena y debe eliminar entradas antiguas. Shimcache es un artefacto magnífico para utilizarse en conjunto con otros artefactos como Prefetch, Amcache, etc.


## Próximos pasos
Existen elementos relacionados con la Infraestructura de Compatibilidad de Aplicaciones que pueden ser aprovechados para realizar diferentes técnicas de *DLL Injection*, *DLL Hijacking* o similares. En futuras publicaciones se analizarán estas técnicas en detalle, así como los métodos de detección que pueden emplearse.

# Referencias
[^PcaSvc]: [The Windows Concept Journey — PCA (Program Compatibility Assistant)](https://medium.com/@boutnaru/the-windows-concept-journey-pca-program-compatibility-assistant-bb996edb22c9)
[^ProgramCompatibilityAssistant1]: [The Program Compatibility Assistant - Part One](https://techcommunity.microsoft.com/blog/askperf/the-program-compatibility-assistant---part-one/372538)
[^UnderstandingShims]: [Understanding Shims](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-7/dd837644(v=ws.10))
[^IAT]: [IAT _ Understanding Import Address Table](https://madhinimurali94.medium.com/iat-understanding-import-address-table-acc542d74cc5)
[^shim-research]: [32Bit Application Shim Fixes](https://research.nasbench.dev/research/other/documented-compat-application-shims)
[^Demystifying-shims]: [Demystifying Shims -or- Using the App Compat Toolkit to make your old stuff work with your new stuff](https://techcommunity.microsoft.com/blog/askperf/demystifying-shims---or---using-the-app-compat-toolkit-to-make-your-old-stuff-wo/374947)
[^ShimDatabaseFiles]: [Shim Database (SDB ) Files](https://www.geoffchappell.com/studies/windows/win32/apphelp/sdb/index.htm?tx=84)
[^EZTools]: [Eric Zimmerman Tools](https://ericzimmerman.github.io/#tldr)
[^ShimDBformat]: [Shim Database (SDB) format specification](https://github.com/libyal/assorted/blob/main/documentation/Shim%20Database%20(SDB)%20format%20specification.asciidoc)
[^13cShimcache]: [Shimcache Execution Is Back](https://www.youtube.com/watch?v=DsqKIVcfA90)
[^ShimcacheDeepDive]: [AppCompatCache Deep Dive](https://nullsec.us/windows-10-11-appcompatcache-deep-dive)
