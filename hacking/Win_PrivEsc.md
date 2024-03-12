---
title: 'Escalada de privilegios en sistemas Windows'
description: 'Notas sobre diversas formas de escalar privilegios en sistemas Windows'
pubDate: 2024-01-26
author: 'Nicolas Riquelme'
tags: ["Hacking", "Windows", "PrivEsc"]
tableOfContents:
  minHeadingLevel: 2
  maxHeadingLevel: 3
---
# Windows Kernel Exploits
El kernel de Windows NT es el componente central del sistema operativo Windows NT. Es responsable de gestionar los recursos del sistema, como la memoria, el procesador y el almacenamiento. También proporciona las funciones básicas para que los programas se ejecuten, como la planificación de procesos, la gestión de memoria y la comunicación entre procesos.

El kernel de Windows NT está dividido en dos capas principales:
+ **Kernel Mode**: Esta capa tiene acceso completo a los recursos del sistema y es responsable de las funciones más críticas del kernel, como la planificación de procesos, la gestión de memoria y la comunicación entre procesos.
+ **User Mode**: Esta capa es responsable de ejecutar los programas de usuario y proporcionarles acceso a los recursos del sistema.

## Herramientas usadas
Se sobre-entiende que para realizar la escalada de privilegios, primero se ha de acceder al sistema víctima con un usuario con pocos privilegios y una vez dentro correr estas herramientas:

+ (Windows Exploit Suggester)[https://github.com/AonCyberLabs/Windows-Exploit-Suggester]
+ (Windows Kernel Exploits)[https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-135]
+ Metasploit
+ (UACMe)[https://github.com/hfiref0x/UACME]


## Explotación y escalada de privilegios:
**ATENCIÓN:** _El uso de módulos de este tipo no está exento de fallos y errores potencialmente catastróficos (¿Me he cargado algo? si, pero San Stallman bendiga los snapshots)._
Habiendo usado Metasploit para acceder a la máquina con Windows, ya sea usando alguno de los exploits para ganar acceso ya sea con una Powershell o una shell con Meterpreter (asumimos que el uso de "getsystem" ha fallado), correremos el siguiente módulo que nos listará vulnerabilidades que podríamos explotar.

```ruby
use post/multi/recon/local_exploit_suggester
	set session {sesión de metasploit con acceso al sistema víctima}
	run			# o exploit
```	
Esto nos dará una lista de módulos a los que el sistema atacado sería susceptible de ser vulnerable, por lo que buscaremos uno que nos convenga (para el fin de este punto, buscaremos uno que escale privilegios usando vulerabilidades en el kernel de Windows).

