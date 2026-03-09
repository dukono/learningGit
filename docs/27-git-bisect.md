# git bisect

## ¿Qué es?

`git bisect` usa **búsqueda binaria** para encontrar qué commit introdujo un bug.

En vez de revisar commits uno por uno, Git divide el historial a la mitad repetidamente hasta encontrar el commit exacto que causó el problema.

> Si tienes 1000 commits, solo necesitas revisar ~10 pasos para encontrar el culpable.

---

## 1. Flujo básico

```bash
# 1. Iniciar bisect
git bisect start

# 2. Marcar el estado actual como malo (tiene el bug)
git bisect bad

# 3. Marcar un commit antiguo donde el bug no existía
git bisect good abc1234
# o usando una tag
git bisect good v1.0.0

# Git hace checkout automáticamente a un commit intermedio
# 4. Pruebas si el bug existe en ese commit
#    Si tiene el bug:
git bisect bad
#    Si no tiene el bug:
git bisect good

# Git sigue dividiendo hasta encontrar el commit exacto
# Cuando termina, muestra el primer commit malo:
# "abc1234 is the first bad commit"

# 5. Terminar bisect y volver a HEAD
git bisect reset
```

---

## 2. Ejemplo completo

```
Historial: A - B - C - D - E - F - G (HEAD, tiene bug)
                              ^
                              queremos encontrar este commit

git bisect start
git bisect bad              # G tiene el bug
git bisect good A           # A no tiene el bug

# Git hace checkout a D (mitad del rango A-G)
# Pruebas... el bug existe en D
git bisect bad

# Git hace checkout a B (mitad del rango A-D)
# Pruebas... el bug NO existe en B
git bisect good

# Git hace checkout a C (mitad del rango B-D)
# Pruebas... el bug existe en C
git bisect bad

# Git concluye: C es el primer commit malo
# "c1234567 is the first bad commit"

git bisect reset            # vuelves a HEAD
```

---

## 3. Automatizar bisect con un script

Si puedes detectar el bug con un comando o script, Git puede hacer el bisect automáticamente:

```bash
git bisect start
git bisect bad HEAD
git bisect good v1.0.0

# Git ejecuta el script en cada commit
# Si el script retorna 0 = good, distinto de 0 = bad
git bisect run npm test
git bisect run ./test-bug.sh
git bisect run python -m pytest tests/test_feature.py
```

---

## 4. Ver el progreso

```bash
git bisect log              # muestra los pasos realizados hasta ahora
git bisect visualize        # abre gitk con el rango actual
git bisect view             # igual que visualize
```

---

## 5. Saltar un commit (skip)

Si un commit intermedio no se puede probar (no compila, está roto por otra razón):

```bash
git bisect skip
# o saltar un rango
git bisect skip abc1234..def5678
```

---

## 6. Guardar y restaurar una sesión de bisect

```bash
# Guardar el log de la sesión
git bisect log > bisect-session.txt

# Restaurar una sesión guardada
git bisect replay bisect-session.txt
```

---

## 7. Comparativa con búsqueda manual

| Commits | Búsqueda manual | git bisect |
|---|---|---|
| 10 | 10 pasos | ~4 pasos |
| 100 | 100 pasos | ~7 pasos |
| 1000 | 1000 pasos | ~10 pasos |
| 10000 | 10000 pasos | ~14 pasos |

---

## 8. Después de encontrar el commit malo

```bash
# Ver qué cambió en ese commit
git show abc1234

# Ver los archivos que cambió
git show --stat abc1234

# Ver el blame del archivo afectado
git blame src/archivo.js
```

