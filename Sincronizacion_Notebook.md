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

1. Ajustes → **Community plugins** → activar el modo e instalar **Git** (autor *Vinzent03*).
2. Configuración sugerida:
   - **Auto pull on startup** = ON
   - **Auto commit-and-sync** cada ~10 min
3. Al instalarlo se crean:
   - `.obsidian/community-plugins.json`
   - `.obsidian/plugins/obsidian-git/`

   Esos archivos hay que commitearlos para que el plugin quede instalado y activo solo en la otra máquina.

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
