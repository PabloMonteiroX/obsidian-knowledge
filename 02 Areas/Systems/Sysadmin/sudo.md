---
type: concept
domain: systems
language: linux
status: learning
---
# sudo

> [!NOTE] ¿Qué es `sudo`?  
> `sudo` permite a un usuario autorizado ejecutar un comando con los privilegios de otro usuario, normalmente `root`.
> 
> ```bash
> sudo comando
> ```
> 
> Ejemplo:
> 
> ```bash
> sudo dnf update
> ```
> 
> El comando se ejecuta con privilegios elevados, pero **no convierte permanentemente al usuario en `root`**.

---

> [!TIP] Buen uso de `sudo`  
> Utiliza `sudo` únicamente cuando el comando necesite privilegios elevados.
> 
> Mejor:
> 
> ```bash
> dnf search nginx
> ```
> 
> que:
> 
> ```bash
> sudo dnf search nginx
> ```
> 
> Y utiliza:
> 
> ```bash
> sudo dnf install nginx
> ```
> 
> cuando la operación realmente necesite modificar el sistema.
> 
> **Principle of least privilege:** utiliza el mínimo nivel de privilegios necesario.

---

> [!WARNING] Riesgo de `sudo`  
> `sudo` permite ejecutar comandos con privilegios de `root`.
> 
> Un comando incorrecto puede modificar o eliminar partes críticas del sistema.
> 
> Por ejemplo:
> 
> ```bash
> sudo rm -rf /ruta/incorrecta
> ```
> 
> puede provocar pérdida de datos.
> 
> Antes de ejecutar un comando con `sudo`, comprueba:
> 
> - qué comando vas a ejecutar;
>     
> - qué argumentos estás proporcionando;
>     
> - qué archivos o directorios modificará;
>     
> - si realmente necesitas privilegios de `root`.
>     
> 
> **Nunca ejecutes comandos con `sudo` que no entiendas.**

---

## Comandos fundamentales

### Ejecutar un comando como root

```bash
sudo comando
```

### Abrir una shell como root

```bash
sudo -i
```

Esto proporciona una sesión interactiva como `root`.

### Comprobar quién eres

```bash
whoami
```

Con:

```bash
sudo whoami
```

normalmente obtendrás:

```text
root
```

---

## ¿Cómo funciona?

Conceptualmente:

```text
usuario
   │
   │ sudo comando
   ▼
sudo
   │
   ├── comprueba autorización
   ├── solicita autenticación si es necesario
   └── ejecuta el comando con otros privilegios
                       │
                       ▼
                     root
```

La configuración de quién puede utilizar `sudo` y qué puede ejecutar se controla mediante la política de `sudo`, tradicionalmente configurada en:

```text
/etc/sudoers
```

y archivos relacionados.

---

## `sudo` vs `su`

|Comando|Función|
|---|---|
|`sudo comando`|Ejecuta un comando con privilegios elevados|
|`sudo -i`|Abre una sesión interactiva como `root`|
|`su`|Cambia de usuario|
|`su -`|Cambia de usuario cargando su entorno de login|

Ejemplo:

```bash
sudo systemctl restart sshd
```

es preferible cuando solamente necesitas privilegios elevados para **ese comando**.

---

## Regla mental

```text
sudo
 │
 ├── no = root permanente
 │
 ├── sí = ejecución con privilegios elevados
 │
 └── requiere autorización
```

> [!NOTE] Concepto clave  
> `sudo` es una herramienta de **privilege escalation controlada** para usuarios autorizados.
> 
> En administración Linux, el objetivo no es utilizar `sudo` constantemente, sino utilizar privilegios elevados **solo cuando sean necesarios**.

---

## Comandos para estudiar

```bash
sudo -l
```

Muestra qué comandos puede ejecutar el usuario mediante `sudo`.

```bash
sudo whoami
```

Comprueba que el comando se está ejecutando como `root`.

```bash
sudo -i
```

Inicia una shell de login como `root`.

```bash
exit
```

Sale de la shell elevada.

---

# Resumen

```text
sudo
│
├── Ejecutar comandos como otro usuario
├── Normalmente se utiliza para root
├── Requiere autorización
├── Permite aplicar mínimo privilegio
├── No convierte permanentemente al usuario en root
└── Un uso incorrecto puede dañar el sistema
```

**Keywords:** `sudo`, `root`, `privileges`, `privilege escalation`, `least privilege`, `/etc/sudoers`# sudo

### Relacionado

- [[]]
# sudo

### Relacionado

- [[Linux]]
