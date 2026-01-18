---
title: Análisis de un DLL Injection
date: 2026-02-01 00:00:00 +0100
categories: [Blog, Forensics]
tags: [dfir, dll injection, process injection, volatility, blog]     # TAG names should always be lowercase
author: julian
description: Process Injection (T1055) es una técnica utilizada ampliamente en las intrusiones realizadas por actores maliciosos o malware para conseguir ejecutar código en memoria ocultándose en un proceso ajeno. En esta publicación hablaremos de la subtécnica DLL Injection (T1055.001) y el método de análisis y detección que se puede llevar a cabo ante estos casos.
# contents:
comments: false
---

# Process Injection - T1055
*Process Injection* (T1055) es una técnica utilizada ampliamente en las intrusiones realizadas por actores maliciosos o malware para conseguir ejecutar código en memoria ocultándose en un proceso ajeno. Mediante la ejecución de código malicioso en el espacio de memoria de un proceso ajeno se pretende evadir las protecciones del sistema víctima así como también llevar a cabo una posible escalada de privilegios[^1]. 

Normalmente, para realizar un *Process Injection* se llevan a cabo los siguientes pasos:
1. Abrir el proceso víctima donde se va a inyectar el código malicioso
2. Reservar espacio de memoria en el proceso víctima
3. Escribir el código maliciosa en el espacio reservado anteriormente
4. Ejecutar el código en un hilo instanciado en el proceso víctima

![Visualización gráfica de un Process Injection](/assets/img/blog/dll-injection/process-injection-image.png)
*Figura 1: Visualización gráfica de un Process Injection [^2]*

Existen diferentes maneras de llevar a cabo un *Process Injection* en un sistema. En esta publicación se habla de la subtécnica DLL Injection (T1055.001) y el método de análisis y detección que se puede llevar a cabo.

# Dynamic-link Library Injection - T1055.001

*DLL Injection* se refiere a la ejecución de código en un proceso ajeno mediante la inyección de un DLL en el proceso víctima. La manera más simple de realizar un DLL Injection se lleva a cabo a través de la utilización de llamadas a la API de Windows[^3]. De esta forma, la inyección del DLL se realiza de la siguiente manera:
1. **Apertura del proceso víctima:** Mediante la llamada a la función `OpenProcess` se abre un objecto de proceso local existente indicando el Process ID (PID) que se desea abrir y -en caso de éxito- obteniendo como respuesta un *handle* del proceso abierto[^4].
2. **Reserva de memoria:** La función `VirtualAllocEx` permite aumentar el espacio de memoria virtual de un proceso especificado (utilizando el *handle* obtenido en el paso anterior). La memoria reservada se inicializa vacía (a ceros) y se obtiene como resultado de la ejecución de la función la dirección de memoria de inicio del nuevo espacio reservado[^5]. En este tipo de inyección, **no se reserva el espacio en memoria equivalente al tamaño del DLL que se pretende inyectar**, solamente se necesita reservar espacio suficiente para contener la ruta del DLL en disco.
3. **Escritura de código:** Una vez reservado el espacio de memoria necesario en el proceso víctima, hay que escribir la información necesaria en dicho espacio llamando a la función `WriteProcessMemory` indicando el *handle* obtenido en el primer paso y la dirección de memoria devuelta por `VirtualAllocEx` en el paso anterior[^6]. En este caso, se escribirá la ubicación del DLL en disco (el *path* absoluto).
4. **Ejecución del código inyectado:** Finalmente, para ejecutar el DLL inyectado en el proceso víctima se utiliza la función `CreateRemoteThread`[^7]. La función `CreateRemoteThread` se encarga de crear un hilo que ejecute el código inyectado y necesita los siguientes parámetros:
> {: .prompt-info}
> El objeto encargado de ejecutar código en un Sistema Operativo son los hilos, los cuales existen bajo el contexto de los procesos que los crean y contienen el código que debe ser ejecutado. **Los procesos no ejecutan código, son los hilos existentes dentro de los procesos los que lo ejecutan**[^8][^9][^10]
   1. El *handle* del proceso abierto en el primer paso.
   2. La dirección de memoria devuelta por `VirtualAllocEx` en el segundo paso
   3. La dirección de memoria de `LoadLibrary`, la cual se encuentra dentro del DLL `Kernel32.dll`.


## Precondiciones
Se debe contar con el privilegio SeDebug ... [^4]
## Puesta en práctica

## Análisis

### Sysmon

### Análisis dinámico

### Análisis de artefactos

### Volatility


# Referencias
[^1]: [MITRE ATT&CK - Process Injection (T1055)](https://attack.mitre.org/techniques/T1055/)
[^2]: [The Pool Party You Will Never Forget: New Process Injection Techniques Using Windows Thread Pools](https://www.safebreach.com/blog/process-injection-using-windows-thread-pools/)
[^3]: [MITRE ATT&CK - Process Injection: Dynamic-link Library Injection (T1055.001)](https://attack.mitre.org/techniques/T1055/001/)
[^4]: [https://learn.microsoft.com/gl-es/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocess?redirectedfrom=MSDN](https://learn.microsoft.com/gl-es/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocess)
[^5]: [https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualallocex](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualallocex)
[^6]: [https://learn.microsoft.com/gl-es/windows/win32/api/memoryapi/nf-memoryapi-writeprocessmemory](https://learn.microsoft.com/gl-es/windows/win32/api/memoryapi/nf-memoryapi-writeprocessmemory)
[^7]: [https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createremotethread](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createremotethread)
[^8]: [Processes and Threads](https://learn.microsoft.com/en-us/windows/win32/procthread/processes-and-threads)
[^9]: [Processs and Thread Management](https://os.inf.tu-dresden.de/Studium/AusgewaehlteBS/windows/Folien/04_Process/04_Process_4.pdf)
[^10]: [Understanding EPROCESS Structure](https://info-savvy.com/wp-content/uploads/2020/07/Understanding-EProcess-Structure-infosavvy.jpg)
