# git gc / git fsck

## ¿Qué son?

- **`git gc`** (garbage collector): limpia y optimiza el repositorio local. Comprime objetos, elimina objetos huérfanos y compacta el historial.
- **`git fsck`** (file system check): verifica la integridad del repositorio, detecta objetos corruptos o huérfanos.

> Git ejecuta `gc` automáticamente de vez en cuando. Normalmente no necesitas ejecutarlo manualmente salvo en repos muy grandes o tras operaciones masivas.

---

## 1. git gc — limpiar y optimizar

### Uso básico
```bash
git gc
```

Lo que hace:
- Comprime objetos sueltos en packfiles
- Elimina objetos huérfanos (sin referencia) más antiguos de 2 semanas
- Limpia el reflog de entradas expiradas
- Optimiza la base de datos de objetos

### Opciones
```bash
git gc --aggressive    # compresión más intensa (más lento, mejor compresión)
git gc --prune=now     # elimina objetos huérfanos inmediatamente (sin esperar 2 semanas)
git gc --auto          # solo ejecuta si hay suficientes objetos sueltos
git gc --quiet         # sin output
```

### Cuándo ejecutarlo manualmente
```bash
# Después de un rebase o filter-branch masivo
git gc --prune=now

# Para reducir el tamaño del repo
git gc --aggressive

# Ver el tamaño del repo antes y después
du -sh .git/
git gc --aggressive
du -sh .git/
```

---

## 2. git fsck — verificar integridad

### Uso básico
```bash
git fsck
```

Salida típica (repo sano):
```
Checking object directories: 100%
Checking connectivity: done.
```

Si hay problemas:
```
broken link from  commit abc1234
              to  blob def5678
missing blob def5678
```

### Opciones útiles
```bash
git fsck --full          # verificación completa (más lenta)
git fsck --unreachable   # muestra objetos sin referencias (huérfanos)
git fsck --lost-found    # guarda objetos huérfanos en .git/lost-found/
git fsck --dangling      # muestra objetos "colgantes" sin referencia
git fsck --no-reflogs    # ignora el reflog en la verificación
```

---

## 3. Recuperar objetos huérfanos con fsck

Si perdiste commits o blobs y el reflog no ayuda:

```bash
# 1. Buscar objetos sin referencia
git fsck --unreachable

# Salida:
# unreachable commit abc1234
# unreachable blob def5678

# 2. Ver el contenido de un objeto huérfano
git show abc1234
git cat-file -p abc1234

# 3. Recuperar un commit huérfano creando una rama
git checkout -b recuperado abc1234
```

### Usar --lost-found para recuperar blobs

```bash
git fsck --lost-found
# Guarda todos los objetos huérfanos en .git/lost-found/
# .git/lost-found/commit/  -> commits huérfanos
# .git/lost-found/other/   -> blobs y trees huérfanos

# Ver el contenido de un blob recuperado
cat .git/lost-found/other/abc1234
```

---

## 4. Ver el tamaño y estadísticas del repo

```bash
# Tamaño total del directorio .git
du -sh .git/

# Estadísticas de objetos
git count-objects
git count-objects -v       # detallado

# Salida de -v:
# count: 15          <- objetos sueltos
# size: 60           <- tamaño en KB
# in-pack: 5423      <- objetos en packfiles
# packs: 2           <- número de packfiles
# size-pack: 2341    <- tamaño de packfiles en KB
# prune-packable: 0  <- objetos que se pueden limpiar
# garbage: 0         <- archivos basura
```

---

## 5. Flujo de mantenimiento recomendado

```bash
# 1. Verificar integridad
git fsck

# 2. Limpiar y optimizar
git gc

# 3. Ver el resultado
git count-objects -v
```

---

## 6. Resumen de cuándo usar cada uno

| Situación | Comando |
|---|---|
| Repo muy lento o pesado | `git gc --aggressive` |
| Liberar espacio después de borrar ramas/tags | `git gc --prune=now` |
| Sospechas que el repo está corrupto | `git fsck --full` |
| Quieres recuperar commits sin referencia | `git fsck --lost-found` |
| Mantenimiento rutinario | `git gc` |

