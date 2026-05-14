# TShark Challenge — Informe Técnico

## Información de la Sala

| Campo | Valor |
|---|---|
| Plataforma | TryHackMe |
| Sala | TShark Challenge I: Teamwork |
| Dificultad | Fácil |
| Temas | TShark, DNS Analysis, IOC Extraction, VirusTotal, Threat Hunting |

---

# Objetivo

Analizar un archivo PCAP utilizando TShark para identificar dominios sospechosos, extraer indicadores de compromiso (IOCs) y verificar amenazas mediante VirusTotal.

---

# Escenario

El equipo de investigación de amenazas detectó un dominio sospechoso potencialmente relacionado con phishing.

El objetivo consistía en analizar el tráfico de red capturado y generar artefactos útiles para herramientas de detección.

---

# Evidencia Analizada

## Archivo PCAP

```text
teamwork.pcap
```

Ubicación:

```text
~/Desktop/exercise-files
```

---

# Enumeración DNS

Se utilizó TShark para extraer todas las consultas DNS presentes en la captura.

### Comando

```bash
tshark -r teamwork.pcap -Y "dns.qry.name" -T fields -e dns.qry.name
```

---

# Dominios Identificados

### Comando

```bash
tshark -r teamwork.pcap -Y "dns.qry.name" -T fields -e dns.qry.name | sort | uniq
```

### Resultado

```text
toolbarqueries.google.com
wittyserver
wittyserver.hsd1.md.comcast.net
www.paypal.com4uswebappsresetaccountrecovery.timeseaways.com
```

---

# Identificación del Dominio Malicioso

El dominio sospechoso identificado fue:

```text
www.paypal.com4uswebappsresetaccountrecovery.timeseaways.com
```

Este dominio intentaba suplantar el servicio legítimo de PayPal mediante técnicas de phishing.

---

# Verificación con VirusTotal

El dominio fue analizado utilizando VirusTotal.

### URL Analizada

```text
http://www.paypal.com4uswebappsresetaccountrecovery.timeseaways.com/
```

---

# Resultados VirusTotal

| Campo | Valor |
|---|---|
| Detecciones | 2/93 |
| Categoría | Phishing |
| Primera sumisión | 2017-04-17 22:52:53 UTC |
| Último análisis | 2026-05-13 15:55:00 UTC |

---

# Servicio Suplantado

```text
PayPal
```

---

# Extracción de Dirección IP

Se utilizó TShark para identificar la IP asociada al dominio malicioso.

### Comando

```bash
tshark -r teamwork.pcap -Y "dns.a" -T fields -e dns.qry.name -e dns.a
```

### Resultado

```text
184.154.127.226
```

---

# Extracción de Correo Electrónico

Se buscaron cadenas relacionadas con formularios y direcciones de correo.

### Comando

```bash
tshark -r teamwork.pcap -V | grep "@"
```

### Correo Encontrado

```text
johnny5alive@gmail.com
```

---

# Indicadores de Compromiso (IOCs)

| Tipo | Valor |
|---|---|
| Dominio | www.paypal.com4uswebappsresetaccountrecovery.timeseaways.com |
| IP | 184.154.127.226 |
| Correo | johnny5alive@gmail.com |
| Tipo de Amenaza | Phishing |

---

# Resumen del Análisis

1. Apertura del archivo PCAP
2. Enumeración de consultas DNS
3. Identificación de dominio sospechoso
4. Verificación mediante VirusTotal
5. Extracción de IP maliciosa
6. Obtención de correo asociado
7. Generación de IOCs

---

# Herramientas Utilizadas

| Herramienta | Uso |
|---|---|
| TShark | Análisis de tráfico |
| VirusTotal | Verificación de amenazas |
| Grep | Búsqueda de cadenas |
| Linux CLI | Enumeración y filtrado |

---

# Habilidades Practicadas

- Análisis PCAP
- Extracción de IOCs
- DNS Traffic Analysis
- Threat Hunting
- OSINT
- Investigación de phishing
- Uso de TShark

---

# Lecciones Aprendidas

- Los dominios de phishing suelen imitar servicios legítimos.
- TShark permite extraer rápidamente indicadores de compromiso.
- VirusTotal es una herramienta fundamental para validación de amenazas.
- El análisis DNS puede revelar infraestructura maliciosa.

---

# Conclusión

El análisis del tráfico permitió identificar una campaña de phishing orientada a suplantar PayPal. Se extrajeron indicadores de compromiso relevantes, incluyendo dominio, dirección IP y correo electrónico asociado, facilitando futuras tareas de detección y respuesta ante incidentes.
