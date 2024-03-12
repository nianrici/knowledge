---
title: 'Protocolo HTTP'
description: 'Notas sobre cómo explotarlo'
pubDate: 2023-12-18
author: 'Nicolas Riquelme'
tags: ["Hacking", "Windows", "web", "http", "discover", "Linux"]
tableOfContents:
  minHeadingLevel: 2
  maxHeadingLevel: 3
---
# HTTP
HTTP es un protocolo de comunicación que permite la transferencia de hipertexto y otros datos a través de Internet. Está basado en el modelo cliente-servidor, en el que el cliente envía una solicitud al servidor y el servidor responde con los datos solicitados. Suele usar el puerto TCP 80 cuando no va protegido y el 443 si la comunicación entre el servidor y el cliente va encriptada mediante el protocolo SSL (HTTPS).

## Enumeración genérica
### Enumeración básica
```bash
nmap -p80,443 -sV {ip víctima}			# lista los servicios que hay tras esos puertos e indica las versiones

whatweb {ip víctima}					# nos indica las tecnologías web que usa el servidor web en esa dirección ip

http {ip víctima}						# lo mismo que el anterior, pero añade las cabeceras (se trata de un script de python, no se refiere al protocolo en si)

dirb http://{ip víctima}				# busca recursivamente los directorios de una web usando un diccionario
```
## Servidores web Windows (IIS)
### Enumeración usando nmap
```bash
nmap -sV {ip víctima}																					# lista los puertos abiertos, los servicios que corren tras ellos y sus versiones
nmap -sV -p80 {ip víctima} --script http-enum															# lo mismo que dirb, pero con un diccionario más corto
nmap -sV -p80 {ip víctima} --script http-headers														# muestra las cabeceras
nmap -sV -p80 {ip víctima} --script http-methods --script-args http-methods.url-path=/webdav/			# lista los métodos usados, indicando los que tienen potencial para ser explotados
nmap -sV -p80 {ip víctima} --script http-webdav-scan --script-args http-methods.url-path=/webdav/		# lo mismo que el anterior, pero centrado en WebDav
```
WebDAV es una extensión del protocolo HTTP que permite a los usuarios acceder, crear, modificar y eliminar archivos en un servidor web. Se utiliza para proporcionar acceso a archivos a través de la web, similar a cómo se accede a archivos a través de un sistema de archivos local.

## Servidores web Linux (Apache)
```bash
nmap -p80 -sV --script banner						# nos indica la versión exacta del servidor Apache y nos indica el OS sobre el que corre (pero no su versión exacta)

use auxiliary/scanner/http/http_version				# lo mismo, pero usando metasploit
	set RHOSTS {ip víctima}
	run

use auxiliary/scanner/http/brute_dirs				# igual que dirb, pero usando metasploit
	set RHOSTS {ip víctima}
	set THREADS 5
	run

use auxiliary/scanner/http/robots_txt				# busca el archivo robots.txt y nos lista el contenido del mismo
	set RHOSTS {ip víctima}
	set THREADS 5
	run

curl {ip víctima} | less							# nos muestra el código de la página web, de donde se puede extraer información relevante
```
