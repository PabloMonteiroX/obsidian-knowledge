## Linux Professional Institute

**LPIC-1** es la primera certificación profesional Linux de LPI y es **vendor-neutral**. La versión actual es **5.0**, con los exámenes **101-500** y **102-500**. Para obtenerla hay que aprobar ambos. Cada examen dura 90 minutos y contiene 60 preguntas de opción múltiple y completar espacios. No hay prerrequisitos. La validez indicada por LPI es de 5 años. 

### 101-500

- Arquitectura del sistema
- Instalación de Linux y gestión de paquetes
- Comandos GNU y Unix
- Dispositivos, sistemas de archivos y FHS

### 102-500

- Shells y Shell Scripting
- Interfaces y escritorios
- Tareas administrativas
- Servicios esenciales
- Fundamentos de redes
- Seguridad

Los objetivos oficiales tienen pesos; los objetivos con mayor peso tienen mayor presencia relativa en el examen.
### Perfil que acredita

```text
Linux command line
       +
System administration
       +
Packages
       +
Filesystems
       +
Users / permissions
       +
Networking
       +
Shell scripting
       +
Security fundamentals
```

---

# LPIC-2 — Certificación

**LPIC-2** es el segundo nivel profesional de LPI y se orienta a administración Linux avanzada y redes pequeñas/medianas.

La versión publicada actualmente por LPI es **4.5**, con **201-450** y **202-450**. Para recibir LPIC-2 debes tener una certificación LPIC-1 activa y aprobar ambos exámenes. Cada examen dura 90 minutos y contiene 60 preguntas. La validez indicada por LPI es de 5 años. 

### 201-450

El examen cubre administración avanzada, incluyendo:

- Kernel
- Arranque del sistema
- Filesystems y dispositivos
- Administración avanzada del almacenamiento
- Networking
- Capacidad y planificación
- Troubleshooting

### 202-450

Se centra en servicios de red y administración de infraestructura, incluyendo:

- DNS
- Web services
- File sharing
- DHCP
- Authentication
- E-mail
- Security
- Troubleshooting

LPI define LPIC-2 como una certificación para administrar redes mixtas pequeñas y medianas. 

---

# Ruta completa

```text
Linux
  │
  ▼
LPIC-1
101-500 + 102-500
  │
  ▼
LPIC-2
201-450 + 202-450
  │
  ▼
LPIC-3
```

### LPIC-1

**Linux Administrator — foundation**

### LPIC-2

**Advanced Linux Administrator**

### LPIC-3

Especializaciones avanzadas.

---

# LPIC-1 + LPIC-2 frente a RHCSA + RHCE

| Ruta                                 | Filosofía                      |
| ------------------------------------ | ------------------------------ |
| LPIC-1                               | Linux general, vendor-neutral  |
| LPIC-2                               | Linux avanzado, vendor-neutral |
| [[RHCSA_EX200_Certificacion\|RHCSA]] | Administración práctica RHEL   |
| RHCE                                 | RHEL + Ansible                 |

```text
LPIC-1
   ↓
Linux breadth
   ↓
LPIC-2
   ↓
Advanced Linux administration
```

Frente a:

```text
RHCSA
   ↓
RHEL administration
   ↓
RHCE
   ↓
Automation
```

No son equivalentes. **LPIC** busca amplitud independiente de distribución; **RHCSA/RHCE** están vinculadas al ecosistema Red Hat.

---

# Valor para cybersecurity

LPIC-1 y LPIC-2 no son certificaciones de pentesting.

Sí proporcionan una base importante:

```text
Linux
 ├── Users
 ├── Permissions
 ├── Processes
 ├── Services
 ├── Networking
 ├── Storage
 ├── DNS
 ├── Authentication
 ├── Logs
 └── Security
          ↓
   Cybersecurity
```

Para seguridad Linux avanzada, LPI ofrece posteriormente **LPIC-3 Security**.

---

# Resumen

```text
LPIC-1
  101-500
  102-500

      ↓

LPIC-2
  201-450
  202-450

      ↓

LPIC-3
```

**LPIC-1:** base profesional Linux.

**LPIC-2:** administración Linux avanzada.

**LPIC-3:** especialización.

Fuentes principales: Linux Professional Institute. 
