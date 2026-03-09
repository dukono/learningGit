# git worktree

## ¿Qué es?

`git worktree` permite tener **varias ramas activas al mismo tiempo** en directorios separados, usando el mismo repositorio.

Sin worktree, para cambiar de rama tienes que hacer stash o commit de tus cambios. Con worktree, cada rama vive en su propio directorio y puedes trabajar en ambas simultáneamente.

---

## 1. Ver los worktrees actuales

```bash
git worktree list
```

Salida típica:
```
/home/user/proyecto        abc1234 [main]
/home/user/proyecto-hotfix def5678 [hotfix/bug-critico]
```

---

## 2. Crear un worktree nuevo

```bash
# Crea un directorio nuevo con la rama especificada
git worktree add ../proyecto-hotfix hotfix/bug-critico

# Crea el directorio y una rama nueva al mismo tiempo
git worktree add ../proyecto-feature -b feature/nueva-api

# Crear desde un commit o tag concreto
git worktree add ../proyecto-v1 v1.0.0
```

---

## 3. Caso de uso típico: hotfix urgente mientras trabajas en feature

```bash
# Estás trabajando en tu feature sin querer hacer commit ni stash
# Llega un bug crítico en main

# 1. Creas un worktree para el hotfix en otro directorio
git worktree add ../hotfix-urgente -b hotfix/bug-login main

# 2. Vas al directorio del hotfix
cd ../hotfix-urgente

# 3. Corriges el bug, haces commit y push
git commit -am "fix: bug login crítico"
git push origin hotfix/bug-login

# 4. Vuelves a tu feature sin haber tocado nada
cd ../proyecto
# Tus cambios siguen exactamente como los dejaste
```

---

## 4. Eliminar un worktree

```bash
# Primero sales del directorio del worktree
cd ../proyecto

# Eliminas el worktree
git worktree remove ../hotfix-urgente

# Si tiene cambios sin commitear, forzar eliminación
git worktree remove --force ../hotfix-urgente
```

---

## 5. Limpiar worktrees que ya no existen en disco

```bash
git worktree prune
```

---

## 6. Limitaciones importantes

- **No puedes tener la misma rama en dos worktrees a la vez.** Cada rama solo puede estar activa en un worktree.
- Los worktrees comparten el mismo `.git`, por lo que los commits, branches y tags son visibles desde todos.
- No se pueden crear worktrees en modo bare.

---

## 7. Diferencia con clonar el repo

| | `git worktree` | Clonar repo |
|---|---|---|
| Comparte historial | Sí (mismo .git) | No (copia independiente) |
| Espacio en disco | Menos (solo el working tree) | Más (copia completa) |
| Cambios visibles en ambos | Sí | No |
| Ideal para | Hotfixes, comparar ramas | Entornos completamente separados |

