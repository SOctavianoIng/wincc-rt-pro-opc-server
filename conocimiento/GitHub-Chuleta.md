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
| ¿El `push` sube **todos** los archivos de la carpeta? | **No.** Solo sube lo que ya está en un **commit**. |

`git clone` solo si la carpeta **no existe**. Si ya clonaste, usá `git pull`.

---

## Qué viaja a GitHub (regla clave)

Git **no** copia la carpeta entera cada vez que hacés `git push`. Hay **tres capas**:

| Capa | Dónde está | Comando que mueve a la siguiente |
|---|---|---|
| 1. Disco (carpeta local) | Editaste el archivo en Cursor o el Bloc de notas | `git add` |
| 2. Commit (historial **local**) | Quedó guardado en git de esta PC | `git commit` |
| 3. GitHub (nube) | Visible en el repo web | `git push` |

**`git push` solo sube commits.** Si un archivo está modificado o es nuevo y todavía no hay `git add` + `git commit`, **no llega a GitHub**.

`git status` es el que aclara en qué capa estás:

| Salida de `git status` | ¿Está en GitHub? |
|---|---|
| `nothing to commit, working tree clean` **y** `up to date with 'origin/master'` | Sí: local y GitHub coinciden. |
| `modified:` o `Changes not staged` | No: el cambio está solo en el disco. |
| `Untracked files` | No: archivo nuevo; git todavía no lo sigue. |
| `Changes to be committed` | No todavía: está en `add`, falta `commit` y `push`. |
| `Your branch is ahead of 'origin/master'` | El commit está en la PC; falta `git push`. |

**Trampa frecuente:** `Your branch is up to date with 'origin/master'` **no** significa que los archivos abiertos en el editor estén en GitHub. Significa que los **commits** locales y los de GitHub coinciden. Podés tener archivos `modified` o `untracked` y GitHub sigue en la versión anterior.

Si `git commit` dice *nothing to commit*, un `git push` **no va a cambiar nada** en GitHub.

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

Los tres comandos van **juntos y en este orden**. Omitir `add` o `commit` deja el archivo solo en la PC.

```powershell
cd C:\Users\socta\Projects\wincc-rt-pro-opc-server
git checkout master
git status
git add -A
git commit -m "Describí el cambio en una frase"
git push origin master
```

**Cuándo:** editaste markdowns (u otros archivos) en la PC y querés que queden en la nube.

- `git add -A` — etapa **todos** los cambios de esa carpeta (modificados y archivos nuevos).
- `git add conocimiento\Chat-Resumen-OPC.md` — etapa **un** archivo (el resto no viaja).
- `git commit` — crea el snapshot **local**. El mensaje va entre comillas. Sin este paso, `push` no lleva el archivo.
- `git push origin master` — copia esos commits a GitHub.
- `git pull origin master` **antes** del push si alguien más (u otro agente) ya subió commits.

Si `git commit` dice *nothing to commit*, no hay nada nuevo para subir: el `push` no actualiza GitHub.

**Comprobar después del push:**

```powershell
git status
```

Debe decir `nothing to commit, working tree clean` y `Your branch is up to date with 'origin/master'`. Recién ahí local y GitHub son la misma versión.

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
| Hice `git push` y en GitHub no aparece el archivo nuevo | Faltó `git add` + `git commit` **antes** del push. `git status`: si figura `modified` o `untracked`, todavía no viajó. |
| `up to date with origin/master` pero el archivo local es distinto | Los commits coinciden; el cambio está solo en el disco. Usá la sección 5. |

No uses `git push --force` ni `git reset --hard` salvo que se pida explícitamente: pueden borrar trabajo local o en GitHub.
