Esta lección explica cómo verificar, instalar y configurar Git en diferentes sistemas operativos, incluyendo la configuración de usuario, correo electrónico y editor preferido.

**Detalles Clave:**

- **Verificación:** Ejecuta `git --version` en la terminal; si muestra un número de versión, Git ya está instalado
- **Instalación por sistema:**
    - Linux: Suele venir preinstalado; si no, usa `sudo apt-get install git` o `sudo pacman -S git`
    - Mac: Instala con `brew install git` (Homebrew) o descarga el instalador del sitio web
    - Windows: Descarga el instalador del sitio web, usa `choco install git.install` (Chocolatey) o instala Git Bash para un entorno tipo Unix
- **Configuración esencial:** Usa `git config` para establecer nombre de usuario y correo electrónico, datos que Git asocia a cada commit
- **La bandera `--global`:** Aplica la configuración a todos los proyectos del sistema; omitirla permite sobrescribirla solo para un proyecto específico
- **Editor preferido:** Puedes configurar Emacs, Vim, Nano o VS Code; en Windows debes proporcionar la ruta completa al ejecutable

**Por Qué Importa:** Configurar correctamente tu identidad y editor son pasos únicos que garantizan que cada commit quede correctamente atribuido y que tu flujo de trabajo con Git sea cómodo desde el inicio.