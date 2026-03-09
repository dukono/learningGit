# git format-patch / git am

## ¿Qué son?

- **`git format-patch`**: exporta uno o varios commits como archivos `.patch`. Cada archivo contiene el diff completo + metadatos (autor, fecha, mensaje).
- **`git am`**: importa esos archivos `.patch` y los aplica como commits reales en la rama actual.

Son la pareja perfecta para "mover" commits entre ramas o repositorios **sin hacer merge ni cherry-pick**.

---

## 1. git format-patch — exportar commits

### Exportar el último commit
```bash
git format-patch -1 HEAD
# Genera: 0001-mensaje-del-commit.patch
```

### Exportar los últimos N commits
```bash
git format-patch -3 HEAD
# Genera 3 archivos .patch numerados
```

### Exportar commits de una rama respecto a otra
```bash
git format-patch main
# Exporta todos los commits de la rama actual que no están en main
```

### Exportar a un directorio específico (-o)
```bash
git format-patch -o patches/ main
# Guarda los .patch en la carpeta patches/
```

### Exportar un commit concreto por hash
```bash
git format-patch -1 abc1234
```

### Exportar un rango de commits
```bash
git format-patch abc1234..def5678
```

---

## 2. git am — importar commits desde .patch

### Aplicar un solo archivo .patch
```bash
git am 0001-mi-feature.patch
```

### Aplicar todos los .patch de un directorio
```bash
git am patches/*.patch
```

### Opciones útiles
```bash
git am --signoff        # Añade "Signed-off-by" al mensaje del commit
git am --3way           # Usa merge a 3 bandas si hay conflictos (recomendado)
git am --abort          # Cancela si hay conflictos
git am --continue       # Continúa tras resolver conflictos
git am --skip           # Salta el patch con conflicto
```

---

## 3. Flujo completo — caso de uso real

### Escenario: quieres mover commits de una rama a otra (o a otro repo)

```bash
# 1. Estás en tu rama feature con commits propios
git checkout feature

# 2. Exportas los commits que no están en main
git format-patch -o patches/ main

# 3. Cambias a la rama destino
git checkout otra-rama

# 4. Aplicas los commits
git am patches/*.patch
```

---

## 4. Diferencia con cherry-pick

| | `cherry-pick` | `format-patch` + `am` |
|---|---|---|
| Necesitas acceso al repo | Sí | No (los .patch son archivos) |
| Conserva autor original | Sí | Sí |
| Sirve para compartir por email/zip | No | Sí |
| Varios commits a la vez | Sí | Sí |
| Conflictos | Posible | Posible |

---

## 5. Ver contenido de un .patch antes de aplicar

```bash
cat 0001-mi-feature.patch
# o
git apply --stat 0001-mi-feature.patch    # resumen de cambios
git apply --check 0001-mi-feature.patch   # verifica si se puede aplicar sin errores
```

---

## 6. Si hay conflicto al hacer git am

```bash
# Git se detiene y avisa del conflicto
# 1. Resuelves el conflicto en el archivo
# 2. Lo añades al staging
git add <archivo-con-conflicto>
# 3. Continúas
git am --continue
```

---

## 7. Guardar hashes en un archivo y aplicar con cherry-pick

Si prefieres usar cherry-pick en vez de `am`, puedes exportar los hashes a un archivo y luego aplicarlos:

```bash
# Exportar hashes a un archivo
git log main..HEAD --format="%H" > commits.txt

# Ver el archivo
cat commits.txt

# Aplicar todos los commits del archivo con cherry-pick
# (hay que invertir el orden para aplicar del más antiguo al más nuevo)
tac commits.txt | xargs git cherry-pick
```

> **Nota:** `tac` invierte el orden de las líneas (el log muestra el más reciente primero, pero cherry-pick necesita aplicarlos del más antiguo al más nuevo).

