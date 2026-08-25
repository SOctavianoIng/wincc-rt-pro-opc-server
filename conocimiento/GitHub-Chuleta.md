# GitHub — Chuleta de comandos

**Propósito:** comandos Git más usados para este repositorio, en **PowerShell** (PC Windows).

**Repo:** https://github.com/SOctavianoIng/wincc-rt-pro-opc-server  
**Carpeta local (referencia):** `C:\Users\socta\Projects\wincc-rt-pro-opc-server`  
**Rama:** `master` (no crear ramas extra salvo que se pida).

Si la carpeta está en otra ruta, cambiá solo el `cd`.

---

## Mapa rápido

| Quiero… | Comando |
|---|---|
| Traer el proyecto **por primera vez** | `git clone …` |
| **Actualizar** lo que ya tengo en la PC | `git fetch` + `git pull` |
| Ver si hay cambios locales | `git status` |
| Ver el último commit | `git log -1 --oneline` |
| Guardar y **subir** a GitHub | `git add` → `git commit` → `git push` |

`git clone` solo si la carpeta **no existe**. Si ya clonaste, usá `git pull`.

---

## 1. Primera vez: nube → PC

```powershell
cd C:\Users\socta\Projects
git clone https://github.com/SOctavianoIng/wincc-rt-pro-opc-server.git
cd wincc-rt-pro-opc-server
git checkout master
```

**Cuándo:** la carpeta `wincc-rt-pro-opc-server` todavía no está en el disco.

---

## 2. Actualizar desde GitHub (el más usado)

```powershell
cd C:\Users\socta\Projects\wincc-rt-pro-opc-server
git checkout master
git fetch origin
git pull origin master
```

**Cuándo:** el repo ya está en la PC y en GitHub hay commits nuevos.

**Comprobar:**

```powershell
git status
git log -1 --oneline
```

Salida esperada: `On branch master` y `Your branch is up to date with 'origin/master'`.

Si `git pull` pide confirmar un merge, cancelá con `Ctrl+C` y no sigas: en este repo se trabaja solo en `master`.

---

## 3. Ver estado y remoto

```powershell
cd C:\Users\socta\Projects\wincc-rt-pro-opc-server
git status
git remote -v
git branch -vv
```

**Cuándo:** antes de pull o push, o si no estás seguro de en qué carpeta/rama estás.

`origin` debe apuntar a `github.com/SOctavianoIng/wincc-rt-pro-opc-server`.

---

## 4. Ver historial

```powershell
git log -5 --oneline
git diff
```

- `git log`: últimos commits (locales + los que ya trajiste).
- `git diff`: cambios **sin commitear** (vacío si no editaste nada).

---

## 5. Subir cambios locales a GitHub

Este repo es documentación: el push lo hace el usuario a mano.

```powershell
cd C:\Users\socta\Projects\wincc-rt-pro-opc-server
git checkout master
git status
git add -A
git commit -m "Describí el cambio en una frase"
git push origin master
```

**Cuándo:** editaste markdowns (u otros archivos) en la PC y querés que queden en la nube.

- `git add -A` — etapa todos los cambios de esa carpeta.
- `git commit` — guarda el snapshot local. El mensaje va entre comillas.
- `git pull origin master` **antes** del push si alguien más (u otro agente) ya subió commits.

Si `git commit` dice *nothing to commit*, no hay nada nuevo para subir.

---

## 6. Si Git pide identidad (solo la primera vez en esa PC)

```powershell
git config --global user.name "Sebastian Octaviano"
git config --global user.email "soctaviano.ing@gmail.com"
```

Ajustá nombre y mail si GitHub tiene otros datos. Hace falta para `git commit`, no para `git pull`.

---

## 7. Errores frecuentes

| Mensaje / situación | Qué hacer |
|---|---|
| `fatal: destination path ... already exists` | No clones de nuevo. Entrá a la carpeta y usá la sección 2 (`pull`). |
| `Your branch is behind 'origin/master'` | `git pull origin master` |
| `Please commit your changes or stash them` | Hay ediciones locales. `git status` y decidí: commitear (sección 5) o no pisar esos archivos. |
| `git pull` abre un editor de merge | `Ctrl+C` y parar. Avisar; no crear otra rama. |
| Carpeta equivocada | `git remote -v` debe mostrar este repo. Si no, `cd` a `wincc-rt-pro-opc-server`. |

No uses `git push --force` ni `git reset --hard` salvo que se pida explícitamente: pueden borrar trabajo local o en GitHub.
