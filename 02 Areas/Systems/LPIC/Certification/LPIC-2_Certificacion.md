# LPIC-2 — Certificación

## Linux Professional Institute

LPIC-2 es el segundo nivel de certificación profesional Linux de LPI.

## Requisito

Para recibir LPIC-2:

```text
LPIC-1 activa
     +
201-450
     +
202-450
     ↓
LPIC-2
```

LPI indica que LPIC-1 debe estar activa para obtener LPIC-2. Los exámenes pueden realizarse en cualquier orden, pero todos los requisitos deben cumplirse para recibir la certificación.

## Exámenes

| Examen | Código | Enfoque |
|---|---|---|
| 201 | 201-450 | Advanced Linux administration |
| 202 | 202-450 | Network services / infrastructure |

Cada examen tiene 60 preguntas y 90 minutos. 

---

# 201-450

## 1. Capacity Planning

Aprender a analizar:

- recursos del sistema;
- consumo de CPU;
- memoria;
- almacenamiento;
- crecimiento;
- disponibilidad;
- rendimiento.

## 2. Linux [[Kernel]]

```text
Kernel
 ├── Modules
 ├── Drivers
 ├── /proc
 ├── /sys
 └── sysctl
```

Comandos:

```bash
uname
lsmod
modprobe
modinfo
insmod
rmmod
depmod
sysctl
dmesg
```

Los objetivos de LPI incluyen arquitectura del kernel, módulos, DKMS, udev y parámetros del kernel. citeturn0search13

## 3. System Startup

```text
Firmware
 ↓
Bootloader
 ↓
Kernel
 ↓
initramfs
 ↓
systemd
 ↓
Services
```

Dominar:

```bash
systemctl
journalctl
grub
dracut
```

## 4. Filesystems & Storage

Profundizar en:

- particiones;
- RAID;
- LVM;
- filesystems;
- montaje;
- cuotas;
- almacenamiento avanzado.

Modelo:

```text
Disk
 ↓
Partition / RAID
 ↓
LVM
 ↓
Filesystem
 ↓
Mount
 ↓
Service
```

## 5. Networking

Administración avanzada:

```text
Routing
Bridges
Interfaces
TCP/IP
Network troubleshooting
```

Herramientas:

```bash
ip
ss
ip route
tcpdump
```

## 6. Troubleshooting

Debe poder investigarse un problema sin reinstalar:

```text
Symptom
 ↓
Collect evidence
 ↓
Identify layer
 ↓
Test hypothesis
 ↓
Fix
 ↓
Verify
```

---

# 202-450

## 1. DNS

Conceptos:

```text
Resolver
Authoritative server
Forwarder
Zone
Record
```

Registros:

```text
A
AAAA
CNAME
MX
NS
SOA
TXT
PTR
```

Herramientas:

```bash
dig
host
nslookup
```

---

## 2. Web Services

Administración de servidores web.

Conceptos:

```text
HTTP
HTTPS
Virtual hosts
TLS
Logs
Reverse proxy
```

Debe poder instalar, configurar, probar y diagnosticar un web service.

---

## 3. File Sharing

### NFS

```text
Linux ↔ Linux
```

### Samba

```text
Linux ↔ Windows
```

Conceptos:

- shares;
- permissions;
- authentication;
- mounts;
- troubleshooting.

---

## 4. DHCP

Conceptos:

```text
DHCP server
 ↓
Discover
 ↓
Offer
 ↓
Request
 ↓
ACK
```

Entender:

- leases;
- scopes;
- reservations;
- options;
- gateways;
- DNS.

---

## 5. Authentication

Profundizar en:

```text
PAM
LDAP
NSS
authentication
authorization
```

---

## 6. E-mail

Conceptos:

```text
MTA
MDA
MUA
SMTP
IMAP
POP3
```

Comprender el flujo:

```text
Client
 ↓
MTA
 ↓
SMTP
 ↓
Remote MTA
 ↓
Mailbox
```

---

## 7. Security

Administración segura:

```text
Access control
Authentication
Network security
Service hardening
Logs
```

---

# Qué diferencia LPIC-2 de LPIC-1

```text
LPIC-1
────────────
"Administro un sistema Linux"

LPIC-2
────────────
"Administro una infraestructura Linux"
```

LPIC-1 se centra en fundamentos y administración general.

LPIC-2 aumenta la profundidad en:

```text
Kernel
Storage
Networking
DNS
Web
NFS
Samba
DHCP
Authentication
Mail
Security
Troubleshooting
```

LPI describe LPIC-2 como orientada a administrar sitios pequeños/medianos y redes mixtas. citeturn0search4

---

# Perfil profesional

LPIC-2 encaja especialmente con:

- Linux System Administrator
- System Engineer
- Infrastructure Engineer
- Network/System Administrator
- Linux Support avanzado
- Junior DevOps
- Infrastructure Security

---

# LPIC-2 + Cybersecurity

```text
LPIC-1
   ↓
Linux fundamentals
   ↓
LPIC-2
   ↓
Advanced administration
   ↓
Networking + services
   ↓
Security
   ↓
Cybersecurity
```

Una base LPIC-2 ayuda especialmente a entender la superficie real de ataque de una infraestructura Linux:

```text
DNS
Web
SSH
NFS
Samba
DHCP
Authentication
Services
Networking
Logs
```

