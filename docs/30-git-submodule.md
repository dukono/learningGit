# git submodule

## ¿Qué es?

`git submodule` permite incluir **otro repositorio Git dentro de tu repositorio**. El repo externo se trata como una dependencia con una versión fija (un commit concreto).

Casos de uso típicos:
- Librerías compartidas entre varios proyectos
- Dependencias que quieres controlar tú mismo
- Temas/plugins de terceros

---

## 1. Añadir un submodule

```bash
git submodule add <url-del-repo> <directorio>

# Ejemplos
git submodule add https://github.com/org/libreria.git libs/libreria
git submodule add https://github.com/org/theme.git themes/default
```

Esto crea:
- El directorio `libs/libreria/` con el repo clonado
- El archivo `.gitmodules` con la configuración

---

## 2. Ver los submodules del proyecto

```bash
git submodule
# Salida: -abc1234 libs/libreria (el - indica que no está inicializado)

git submodule status
# Salida:  abc1234 libs/libreria (v1.2.0)
```

---

## 3. Clonar un repo que tiene submodules

```bash
# Opción 1: clonar y luego inicializar submodules
git clone <url>
git submodule init
git submodule update

# Opción 2: clonar con submodules de una sola vez (recomendado)
git clone --recurse-submodules <url>
```

---

## 4. Actualizar un submodule a la última versión

```bash
# Entras al directorio del submodule
cd libs/libreria
git pull origin main

# Vuelves al repo principal y haces commit del nuevo puntero
cd ../..
git add libs/libreria
git commit -m "chore: actualizar submodule libreria"
```

### Actualizar todos los submodules de una vez
```bash
git submodule update --remote
git submodule update --remote --merge    # hace merge de los cambios
```

---

## 5. Inicializar submodules después de un pull

Cuando alguien actualiza el puntero de un submodule y haces pull, necesitas actualizar:

```bash
git pull
git submodule update --init --recursive
# --init: inicializa submodules nuevos
# --recursive: también actualiza submodules de submodules
```

---

## 6. Eliminar un submodule

```bash
# 1. Eliminar la entrada en .gitmodules
git submodule deinit libs/libreria

# 2. Eliminar el directorio del submodule del índice
git rm libs/libreria

# 3. Hacer commit
git commit -m "chore: eliminar submodule libreria"

# 4. Limpiar el directorio .git/modules (opcional)
rm -rf .git/modules/libs/libreria
```

---

## 7. Ver el archivo .gitmodules

```bash
cat .gitmodules
```

Contenido típico:
```ini
[submodule "libs/libreria"]
    path = libs/libreria
    url = https://github.com/org/libreria.git
    branch = main
```

---

## 8. Diferencia con otras alternativas

| | `git submodule` | `npm/pip/maven` | Copiar código |
|---|---|---|---|
| Control de versión | Commit concreto | Semver | Manual |
| Incluye historial | Sí | No | No |
| Actualizaciones | Manual | Automático | Manual |
| Complejidad | Alta | Baja | Baja |

> **Recomendación:** Si tu lenguaje tiene un gestor de paquetes (npm, pip, etc.), úsalo en lugar de submodules. Los submodules son útiles cuando necesitas incluir código sin gestor de paquetes o cuando necesitas el historial completo de la dependencia.

