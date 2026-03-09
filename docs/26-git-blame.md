# git blame

## ¿Qué es?

`git blame` muestra **quién modificó cada línea de un archivo**, en qué commit y cuándo.

Es útil para saber por qué una línea de código está como está, quién la escribió y en qué contexto.

---

## 1. Uso básico

```bash
git blame <archivo>
git blame src/main.js
```

Salida típica:
```
abc1234 (Juan García  2026-01-15 10:23:45 +0100  1) const x = 1;
def5678 (Ana López    2026-02-03 14:11:02 +0100  2) const y = 2;
9876abc (Juan García  2026-01-15 10:23:45 +0100  3) console.log(x + y);
```

Cada línea muestra:
- **hash**: commit que introdujo esa línea
- **autor**: quién hizo el commit
- **fecha**: cuándo
- **número de línea**
- **contenido de la línea**

---

## 2. Ver solo un rango de líneas

```bash
git blame -L 10,20 <archivo>      # líneas 10 a 20
git blame -L 10,+5 <archivo>      # 5 líneas desde la línea 10
git blame -L /función/,+10 <archivo>  # desde la primera línea que contiene "función"
```

---

## 3. Formato corto (hash abreviado)

```bash
git blame --abbrev=7 <archivo>    # hash de 7 caracteres (por defecto)
git blame -s <archivo>            # sin nombre de autor ni fecha
```

---

## 4. Ver el blame de un archivo en un commit concreto

```bash
git blame abc1234 -- <archivo>
git blame HEAD~3 -- <archivo>
```

---

## 5. Ignorar cambios de espacios/formato

```bash
git blame -w <archivo>            # ignora cambios de espacios en blanco
git blame -M <archivo>            # detecta líneas movidas dentro del mismo archivo
git blame -C <archivo>            # detecta líneas copiadas desde otros archivos
```

---

## 6. Ver el commit completo de una línea

Una vez que tienes el hash de una línea con `git blame`, puedes ver el commit completo:

```bash
# 1. Obtienes el hash de la línea con blame
git blame src/main.js

# 2. Ves que la línea viene del commit abc1234
git show abc1234
```

---

## 7. Diferencia entre blame y log

| | `git blame` | `git log` |
|---|---|---|
| Muestra | Quién cambió cada línea | Historial de commits |
| Nivel | Línea a línea | Commit a commit |
| Uso | "¿Quién escribió esto?" | "¿Qué cambió en este commit?" |

---

## 8. Casos de uso típicos

```bash
# Ver quién tocó un archivo de configuración
git blame config/database.yml

# Ver quién introdujo un bug en las últimas 20 líneas de un archivo
git blame -L 50,70 src/auth.js

# Ver el blame de un archivo tal como estaba hace 1 semana
git blame HEAD@{1.week.ago} -- src/auth.js

# Ver el blame de un archivo en una rama concreta
git blame feature/nueva-api -- src/api.js
```

