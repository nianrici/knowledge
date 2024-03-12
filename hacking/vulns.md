---
title: 'Vulnerabilidades'
description: 'Notas sobre vulnerabilidades'
pubDate: 2024-01-22
author: 'Nicolas Riquelme'
tags: ["Hacking", "Windows", "Linux"]
tableOfContents:
  minHeadingLevel: 2
  maxHeadingLevel: 3
---
# ¿Que es una vulnerabilidad?
Según el NIST es una debilidad en la lógica computacional (por ejemplo, en el código) que se encuentra en componentes de software (ya sean programas o del mismo OS) y/o hardware que, cuando se explota, tiene un impacto negativo en la confidencialidad, la integridad o la disponibilidad de los datos.

## Documentación sobre las vulnerabilidades públicas
+ **[CVE](https://www.cve.org)**: Common Vulnerabilities and Exposures. Método de referencia usado en las vulnerabilidades públicas.
+ **[NVD](https://nvd.nist.gov)**: National Vulnerability Database. Repositorio del govierno estadounidense que contiene todas las vulnerabilidades publicadas.
+ **[Exploit DB](https://www.exploit-db.com)**: Base de datos mantenida por los creadores de la distro Kali Linux. Contiene tanto una descripción de las vulnerabilidades como scripts y POCs para explotarlas.

# Auditorías
## Windows
### Recursos
+ [Security Content Automation Protocol (SCAP)](https://public.cyber.mil/stigs/scap)
+ [STIG Viewer](https://public.cyber.mil/stigs/srg-stig-tools/)
+ [Policies](https://public.cyber.mil/stigs/gpo/)

```ruby
# Paso 1: Auditoría
SCC #SCAP Compliance Checker
-->	Start Scan --> View Results --> (botón derecho sobre los resultados) Open Summary Viewer...

# Paso 2: Visualización de los resultados
STIG Viewer
--> Checklists -> Oper Checklists from file
--> Import -> XCCD Results File
	--> Desktop
		--> SCAP Scan Results
			--> Sessions
				--> {sesión}
					--> Results
						--> SCAP
							--> XML
								--> {archivo.XML}
# Paso 3: Mitigación
LGPO (portable)
LGPO /g {directorio con las políticas}

# Repetir pasos 1 y 2 para ver los nuevos resultados

```
