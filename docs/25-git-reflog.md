# git reflog

## ¿Qué es?

`git reflog` registra **todos los movimientos de HEAD** en tu repositorio local: commits, merges, rebases, checkouts, resets, etc.

Es como una **red de seguridad**: si borras una rama, haces un reset --hard o pierdes commits, puedes recuperarlos desde el reflog.

> El reflog es **solo local**, no se sube al remoto con `git push`.

---

## 1. Ver el reflog

```bash
git reflog
# o equivalente:
git reflog show HEAD
```

Salida típica:
```
abc1234 HEAD@{0}  commit: feat: nueva funcionalidad
def5678 HEAD@{1}  reset: moving to HEAD~1
9876abc HEAD@{2}  checkout: moving from main to feature
...
```

Cada línea tiene:
- **hash**: el commit al que apuntaba HEAD
- **HEAD@{N}**: referencia relativa (N = hace cuántos movimientos)
- **acción**: lo que ocurrió (commit, reset, checkout, rebase, merge...)

---

## 2. Ver reflog de una rama concreta

```bash
git reflog show feature
git reflog show main
```

---

## 3. Ver reflog con fechas

```bash
git reflog --date=iso
git reflog --date=relative   # "2 hours ago", "3 days ago"
```

---

## 4. Recuperar commits perdidos

### Caso: hiciste un reset --hard y perdiste commits

```bash
# 1. Miras el reflog para encontrar el hash antes del reset
git reflog

# 2. Ves algo como:
# abc1234 HEAD@{0}  reset: moving to HEAD~3   <- estado actual
# def5678 HEAD@{1}  commit: mi commit perdido  <- quiero volver aquí

# 3. Recuperas con reset
git reset --hard def5678

# o creas una rama nueva desde ese punto
git checkout -b recuperada def5678
```

---

## 5. Recuperar una rama borrada

```bash
# 1. Borraste la rama por error
git branch -D mi-rama

# 2. Buscas en el reflog el último commit de esa rama
git reflog | grep "mi-rama"
# o simplemente miras los últimos movimientos
git reflog -20

# 3. Recreas la rama desde ese hash
git checkout -b mi-rama abc1234
```

---

## 6. Usar referencias HEAD@{N}

```bash
# Ver el estado del repo hace 3 movimientos
git show HEAD@{3}

# Hacer diff entre el estado actual y hace 5 movimientos
git diff HEAD@{5} HEAD

# Resetear al estado de hace 2 movimientos
git reset --hard HEAD@{2}
```

---

## 7. Usar referencias por tiempo

```bash
git show HEAD@{1.hour.ago}
git show HEAD@{yesterday}
git show main@{2.days.ago}
git diff HEAD@{1.week.ago} HEAD
```

---

## 8. Limpiar el reflog (no recomendado)

```bash
# Borra entradas antiguas del reflog (más de 90 días por defecto)
git reflog expire --expire=90.days.ago --all

# Borra todo el reflog
git reflog expire --expire=now --all
git gc --prune=now
```

> ⚠️ Una vez limpiado el reflog, los commits huérfanos son irrecuperables.

---

## Resumen de cuándo usarlo

| Situación | Solución con reflog |
|---|---|
| Reset --hard accidental | `git reflog` → `git reset --hard <hash>` |
| Rama borrada por error | `git reflog` → `git checkout -b <rama> <hash>` |
| Rebase que salió mal | `git reflog` → `git reset --hard <hash-antes-rebase>` |
| Commit amend que no querías | `git reflog` → `git reset --hard <hash-original>` |

