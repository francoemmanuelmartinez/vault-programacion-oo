---
tags: [meta, guia, git, sincronizacion]
---

# Sincronización del vault en la notebook

> Guía para clonar este vault en la notebook y mantenerlo sincronizado con GitHub.
> Repo: `https://github.com/francoemmanuelmartinez/vault-programacion-oo` (privado)

---

## Fase C — Notebook (una sola vez)

### 1. Instalar Git y Obsidian
Si no están instalados:
- **Git:** https://git-scm.com/download/win
- **Obsidian:** https://obsidian.md

### 2. Clonar el vault
Ajustá la ruta a tu gusto (`<usuario>` es tu usuario de Windows en la notebook):

```powershell
git clone https://github.com/francoemmanuelmartinez/vault-programacion-oo.git "C:\Users\<usuario>\Obsidian\vaults\vault-programacion-oo"
```

### 3. Configurar la identidad de git
Usá la **misma** que en la desktop, para que el historial quede consistente:

```powershell
git config --global user.name "Franco Emmanuel Martinez"
git config --global user.email "framco.martinez0738@gmail.com"
```

### 4. Abrir el vault en Obsidian
Obsidian → **Open folder as vault** → elegís la carpeta clonada.

---

## Fase D — Plugin Obsidian Git (en **ambas** máquinas)

### Instalación y configuración (una vez por máquina)

1. Ajustes → **Community plugins** → desactivar **Restricted mode**.
2. **Browse** → buscar **Git** (autor *Vinzent03*) → **Install** → **Enable**.
3. Configuración sugerida:
   - **Auto pull on startup** = ON
   - **Auto commit-and-sync** cada ~10 min
4. **Autenticación:** en escritorio, `obsidian-git` delega el login al git del
   sistema (Git Credential Manager). La primera vez que sincroniza te pide las
   credenciales de GitHub. El token **no** viaja en el repo: hay que
   autenticar **una vez en cada máquina**.

### Qué commitear del plugin (para que la notebook ya lo tenga)

Al instalarlo se crean:

```text
.obsidian/community-plugins.json          ← qué plugins están habilitados
.obsidian/plugins/obsidian-git/
    main.js                               ← CÓDIGO del plugin
    manifest.json                         ← CÓDIGO del plugin
    styles.css                            ← CÓDIGO del plugin
    data.json                             ← config propia de cada máquina (NO commitear)
```

> [!warning] `community-plugins.json` no alcanza solo
> Ese archivo únicamente registra cuáles plugins están *habilitados*; Obsidian
> **no** descarga un plugin si falta su carpeta. Para que el plugin viaje hay que
> commitear también su **código** (`main.js`, `manifest.json`, `styles.css`).

> [!danger] No commitear `data.json`
> Guarda configuración propia de cada máquina (y puede contener el token de
> GitHub). Está excluido en el `.gitignore` del vault
> (`.obsidian/plugins/*/data.json`). Si se commitea, el secreto queda en el
> historial de git.

---

## Regla de oro

- **Antes de editar** en cualquier máquina: `pull` (o dejá que lo haga el plugin al iniciar).
- **No edites en las dos máquinas a la vez** sin sincronizar: aparecen conflictos.

## Comandos manuales (si preferís no depender del plugin)

```powershell
# Antes de trabajar
git pull

# Al terminar
git add -A
git commit -m "notas: <resumen>"
git push
```

---

## Notas

- `.gitignore` del vault excluye lo propio de cada máquina: `.obsidian/workspace.json`, `.obsidian/workspace-mobile.json`, `.obsidian/cache/` y `.trash/`.
