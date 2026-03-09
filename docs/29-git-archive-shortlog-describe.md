# git archive / git shortlog / git describe

## Comandos de utilidad

---

## 1. git archive — exportar código fuente

Exporta el contenido del repo (o un commit/rama) como `.zip` o `.tar.gz`, **sin incluir el historial de Git**.

### Exportar HEAD como zip
```bash
git archive --format=zip HEAD > proyecto.zip
git archive --format=zip --output=proyecto.zip HEAD
```

### Exportar HEAD como tar.gz
```bash
git archive --format=tar.gz HEAD > proyecto.tar.gz
```

### Exportar una rama o tag concreto
```bash
git archive --format=zip main > main.zip
git archive --format=zip v1.0.0 > release-v1.0.zip
```

### Exportar solo un subdirectorio
```bash
git archive --format=zip HEAD src/ > solo-src.zip
```

### Exportar con prefijo en los archivos
```bash
git archive --format=zip --prefix=proyecto-v1.0/ v1.0.0 > v1.0.zip
# Los archivos dentro del zip tendrán el prefijo: proyecto-v1.0/archivo.js
```

### Diferencia con copiar la carpeta
| | `git archive` | Copiar carpeta |
|---|---|---|
| Incluye `.git` | No | Sí |
| Incluye archivos ignorados | No | Sí |
| Respeta `.gitattributes` | Sí | No |

---

## 2. git shortlog — resumen de commits por autor

Muestra un resumen del historial agrupado por autor. Útil para ver la contribución de cada persona.

### Uso básico
```bash
git shortlog
```

Salida típica:
```
Ana López (12):
      feat: login con Google
      fix: validación de email
      ...

Juan García (8):
      feat: nuevo dashboard
      ...
```

### Opciones útiles
```bash
git shortlog -s          # solo el número de commits por autor (sin mensajes)
git shortlog -s -n       # ordenado por número de commits (de más a menos)
git shortlog -s -n -e    # incluye el email del autor
```

### Ver shortlog de un rango
```bash
git shortlog v1.0.0..HEAD        # commits desde v1.0.0 hasta HEAD
git shortlog main..feature       # commits de feature que no están en main
git shortlog --since="1 month ago"
```

---

## 3. git describe — describir un commit usando tags

Muestra una descripción legible de un commit basada en el **tag más cercano** en el historial.

### Uso básico
```bash
git describe
# Salida: v1.2.0-3-gabc1234
# Significa: tag v1.2.0, 3 commits después, hash abc1234
```

### Si HEAD está exactamente en un tag
```bash
git describe
# Salida: v1.2.0
```

### Opciones útiles
```bash
git describe --tags          # incluye tags sin anotación (lightweight tags)
git describe --always        # si no hay tags, muestra el hash
git describe --abbrev=10     # usa 10 caracteres del hash
git describe --long          # siempre muestra el formato largo (tag-N-ghash)
```

### Describir un commit concreto
```bash
git describe abc1234
git describe HEAD~5
```

### Caso de uso: versioning automático en CI/CD
```bash
VERSION=$(git describe --tags --always)
echo "Construyendo versión: $VERSION"
# Output: v1.2.0-3-gabc1234
```

