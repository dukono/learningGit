# git notes

## ¿Qué es?

`git notes` permite **añadir anotaciones a commits existentes sin modificarlos**. Las notas se almacenan separadas del commit, por lo que el hash del commit no cambia.

Casos de uso:
- Añadir información extra a commits ya pusheados (revisiones, tickets, notas de deploy)
- Anotar commits de terceros sin alterar su historial
- CI/CD: guardar resultados de tests o builds junto al commit

---

## 1. Añadir una nota a un commit

```bash
# Añadir nota al commit actual (HEAD)
git notes add -m "Revisado por Ana el 2026-03-09"

# Añadir nota a un commit concreto
git notes add -m "Deploy en producción: 2026-03-09 14:00" abc1234

# Abrir el editor para escribir la nota
git notes add abc1234
```

---

## 2. Ver notas

```bash
# Ver la nota de un commit concreto
git notes show abc1234
git notes show HEAD

# Ver notas en el log
git log --show-notes
git log --notes
```

Salida de `git log --show-notes`:
```
commit abc1234
Author: Juan García
Date:   Mon Mar 9 10:00:00 2026

    feat: nueva funcionalidad

Notes:
    Revisado por Ana el 2026-03-09
    Deploy en producción: 2026-03-09 14:00
```

---

## 3. Editar una nota existente

```bash
git notes edit abc1234
# Abre el editor con la nota actual para modificarla
```

---

## 4. Añadir texto a una nota existente (append)

```bash
git notes append -m "Aprobado en code review" abc1234
```

---

## 5. Eliminar una nota

```bash
git notes remove abc1234
git notes remove HEAD
```

---

## 6. Listar todos los commits con notas

```bash
git notes list
# Salida: <hash-nota> <hash-commit>
```

---

## 7. Compartir notas con el remoto

Las notas **no se pushean automáticamente** con `git push`. Hay que hacerlo explícitamente:

```bash
# Pushear notas al remoto
git push origin refs/notes/commits

# Hacer fetch de notas del remoto
git fetch origin refs/notes/commits:refs/notes/commits
```

### Configurar push automático de notas
```bash
# En .git/config o ~/.gitconfig
git config --add remote.origin.fetch '+refs/notes/*:refs/notes/*'
git config --add remote.origin.push 'refs/notes/commits'
```

---

## 8. Namespaces de notas

Por defecto las notas van al namespace `commits`. Puedes usar namespaces personalizados:

```bash
# Añadir nota en namespace personalizado
git notes --ref=review add -m "LGTM" abc1234
git notes --ref=jira add -m "PROJ-1234" abc1234

# Ver notas de un namespace concreto
git log --notes=review
git notes --ref=jira show abc1234
```

---

## 9. Diferencia con git commit --amend

| | `git notes` | `git commit --amend` |
|---|---|---|
| Cambia el hash del commit | No | Sí |
| Sirve para commits pusheados | Sí | No (reescribe historia) |
| Visible en `git log` | Con `--notes` | Siempre |
| Se pushea automáticamente | No | Sí |

