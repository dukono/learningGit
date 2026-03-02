# 9. git rebase - Reescribiendo Historia

[🏠 Volver al Índice](../../GIT_COMANDOS_GUIA_PRACTICA.md)

---

## 9. git rebase - Reescribiendo Historia
[⬆️ Top](#9-git-rebase---reescribiendo-historia)

**¿Qué hace?**
Reaplica commits de una rama encima de otra, reescribiendo la historia. Es como "mover" tus commits a otro punto de partida, haciendo que parezca que siempre trabajaste desde ahí.

**Funcionamiento interno:** [🔙](#9-git-rebase---reescribiendo-historia)

```
ANTES del rebase:
  main:    A---B---C
                \
  feature:       D---E---F

  Situación: quieres que feature parta de C (el último de main),
  pero cuando la creaste, main solo tenía B.

DESPUÉS del rebase (git rebase main estando en feature):
  main:    A---B---C
                    \
  feature:           D'--E'--F'

  Git hace internamente:
  1. Encuentra el ancestro común (B)
  2. Guarda los commits únicos de feature (D, E, F) como patches temporales
  3. Mueve el puntero de feature a C (último de main)
  4. Aplica cada patch uno a uno → crea D', E', F' (nuevos hashes)
  5. Los commits originales D, E, F ya no son accesibles (pero sí desde reflog)

  → Historia lineal: parece que siempre trabajaste desde C
  → Los hashes cambian (D ≠ D', E ≠ E', F ≠ F')
  → Si ya habías pusheado D, E, F, necesitarás --force-with-lease
```

**Todas las opciones importantes:** [🔙](#9-git-rebase---reescribiendo-historia)

```bash
# ============================================
# 1. Rebase básico sobre otra rama
# ============================================
# Situación: Llevas varios días en feature/login y main ha avanzado.
# Quieres incorporar los cambios de main en tu rama antes del merge.
#
# Estás en feature/login:
git rebase main
# → Tus commits se reaplican encima del último commit de main
# → Si hay conflictos, Git para y te pide resolverlos

# ─────────────────────────────────────────────
# ¿MERGE o REBASE para actualizar tu rama con main?
# ─────────────────────────────────────────────
# Hay dos formas de traer los cambios de main a tu feature branch.
# La diferencia es si quieres o no un merge commit en tu historial.
#
# OPCIÓN A: git merge main  → crea un merge commit (ensucia el historial)
#
#   Antes:                        Después de merge:
#   main:    A---B---C            main:    A---B---C
#   feature:      D---E           feature:      D---E---M
#                                                       ↑ merge commit
#
#   El merge commit M aparecerá luego en rebase -i si no tienes cuidado.
#   Si haces rebase -i HEAD~3, verás: pick D, pick E, pick M → lío.
#
# OPCIÓN B: git rebase origin/main  → NO crea merge commit ✅
#
#   Antes:                        Después de rebase:
#   main:    A---B---C            main:    A---B---C
#   feature:      D---E           feature:          D'---E'
#                                                   ↑ tus commits recolocados
#
#   Tus commits quedan ENCIMA de C. Historial limpio y lineal.
#   Cuando luego hagas rebase -i HEAD~2, solo verás D' y E' → sin sorpresas.
#
# ✅ FLUJO RECOMENDADO para actualizar tu rama con main:
git fetch origin              # descarga los cambios remotos sin aplicarlos
git rebase origin/main        # recoloca tus commits encima de main
# Si hay conflictos → resolverlos → git rebase --continue
# Si quieres cancelar → git rebase --abort

# ─────────────────────────────────────────────
# ¿Y si ya hice git merge main por error?
# ─────────────────────────────────────────────
# Deshaz el merge commit y luego rebasea limpiamente:
git reset --hard HEAD~1       # deshace el merge commit (los cambios de main desaparecen del historial)
git fetch origin
git rebase origin/main        # ahora sí, sin merge commit


# ============================================
# 2. Rebase interactivo - EXPLICACIÓN DIDÁCTICA
# ============================================
#
# ┌─────────────────────────────────────────────────────────────────┐
# │  ANALOGÍA: imagina que git rebase -i es una MESA DE MONTAJE     │
# │  donde Git saca todos tus commits en tarjetas, los pone         │
# │  encima de la mesa en orden, y TÚ decides qué hacer con cada   │
# │  uno ANTES de que Git los vuelva a "grabar" en la historia.     │
# └─────────────────────────────────────────────────────────────────┘
#
# ─────────────────────────────────────────────
# PASO 0: ¿Qué es HEAD~N y cómo elegir el número?
# ─────────────────────────────────────────────
# HEAD     = el commit en el que estás ahora (el más reciente)
# HEAD~1   = el commit anterior a HEAD
# HEAD~2   = 2 commits antes de HEAD
# HEAD~5   = 5 commits antes de HEAD  ← "dame los últimos 5 commits"
#
# IMPORTANTE: HEAD~N NO significa "mis últimos N commits propios".
# Significa "los N commits más recientes del historial de esta rama",
# incluidos los merge commits que trajiste de main si los hay.
#
# ¿Cómo saber cuántos commits incluir?
# Usa git log --oneline para ver el historial ANTES de hacer el rebase:
git log --oneline
# Ejemplo de salida:
#
#   f3a9b1c (HEAD) fix: typo en formulario        ← (1) tuyo
#   d7e2f4a fix: validación email                 ← (2) tuyo
#   c1b0e3d Merge branch 'main' into feature      ← (3) ⚠️ merge commit
#   a4f8c2e feat: add login form                  ← (4) tuyo
#   9341630 (main) feat: initial setup            ← (5) de main → STOP aquí
#
# En este caso tienes 3 commits propios + 1 merge commit de main.
# Para trabajar SOLO con tus commits propios (sin el merge), tienes dos opciones:
#
#   Opción A: contar cuántos están ENCIMA del merge y usar ese número
#             → HEAD~2  (solo los 2 commits propios encima del merge)
#
#   Opción B: referenciar directamente el hash del último commit que NO es tuyo
#             → git rebase -i 9341630  (todo lo posterior a ese commit)
#
# ─────────────────────────────────────────────
# PASO 1: Abrir el editor interactivo
# ─────────────────────────────────────────────
git rebase -i HEAD~4
# Git abre el editor (vim/nano) con algo así:
#
# ┌──────────────────────────────────────────────────────────────┐
# │  pick a1b2c3 feat: add login form                            │
# │  pick d4e5f6 wip                                             │
# │  pick g7h8i9 fix typo                                        │
# │  pick j0k1l2 feat: login validation                          │
# │                                                              │
# │  # Rebase 9341630..j0k1l2 onto 9341630 (4 commands)         │
# │  # Commands:                                                 │
# │  # p, pick   = use commit                                    │
# │  # r, reword = use commit, but edit the commit message       │
# │  # e, edit   = use commit, but stop for amending             │
# │  # s, squash = use commit, but meld into previous commit     │
# │  # f, fixup  = like squash, but discard this commit's message│
# │  # d, drop   = remove commit                                 │
# └──────────────────────────────────────────────────────────────┘
#
# ⚠️ OJO AL ORDEN: el editor muestra los commits del MÁS ANTIGUO (arriba)
#                  al MÁS RECIENTE (abajo). Es el INVERSO de git log.
#
#   git log:      f3a (más reciente arriba)
#                 d7e
#                 a1b (más antiguo abajo)
#
#   rebase -i:    a1b (más antiguo arriba)  ← así lo muestra el editor
#                 d7e
#                 f3a (más reciente abajo)
#
# ─────────────────────────────────────────────
# PASO 2: Entender cada opción UNA POR UNA
# ─────────────────────────────────────────────
#
# ┌──────┬──────────────────────────────────────────────────────────┐
# │OPCIÓN│ QUÉ HACE                                                 │
# ├──────┼──────────────────────────────────────────────────────────┤
# │ pick │ Usa el commit TAL CUAL. No cambia nada.                  │
# │      │ Es la opción por defecto. Si no tocas nada, todos son    │
# │      │ pick y el rebase no cambia nada.                         │
# ├──────┼──────────────────────────────────────────────────────────┤
# │reword│ Usa el commit con sus cambios, PERO te abre el editor    │
# │      │ para que cambies el MENSAJE del commit.                  │
# │      │ Los archivos modificados NO cambian, solo el texto.      │
# ├──────┼──────────────────────────────────────────────────────────┤
# │ edit │ Para el rebase EN ESE COMMIT y te deja en modo "pausa".  │
# │      │ Puedes hacer git commit --amend para modificar el commit │
# │      │ o incluso dividirlo en varios (ver CASO 3 más abajo).    │
# │      │ Cuando acabes: git rebase --continue                     │
# ├──────┼──────────────────────────────────────────────────────────┤
# │squash│ FUSIONA este commit con el que está JUSTO ENCIMA en el   │
# │      │ editor (el anterior cronológicamente).                   │
# │      │ Te abre el editor para que elijas/combines los mensajes. │
# │      │ Resultado: 1 solo commit con los cambios de ambos.       │
# ├──────┼──────────────────────────────────────────────────────────┤
# │fixup │ IGUAL que squash, PERO descarta el mensaje de ESTE       │
# │      │ commit y conserva solo el mensaje del de arriba.         │
# │      │ No abre editor para mensajes. Más rápido que squash.     │
# │      │ Úsalo cuando el commit es un "wip" o "fix typo" que no   │
# │      │ aporta información en el mensaje.                        │
# ├──────┼──────────────────────────────────────────────────────────┤
# │ drop │ ELIMINA completamente este commit. Sus cambios           │
# │      │ desaparecen del historial como si nunca hubieran         │
# │      │ existido. También puedes borrar la línea entera.         │
# └──────┴──────────────────────────────────────────────────────────┘
#
# ─────────────────────────────────────────────
# REGLA FUNDAMENTAL de squash y fixup
# ─────────────────────────────────────────────
# squash y fixup SIEMPRE se fusionan con el commit de ARRIBA en el editor.
# "El de arriba" = el commit anterior en el tiempo.
#
# Visualización:
#
#   pick   a1b2c3 feat: add login form    ← BASE (no puede tener squash/fixup)
#   fixup  d4e5f6 wip                    ← se pega a a1b2c3 (el de arriba)
#   fixup  g7h8i9 fix typo               ← se pega al resultado anterior
#   pick   j0k1l2 feat: login validation ← nuevo commit independiente
#   squash k2l3m4 fix validation bug     ← se pega a j0k1l2 (el de arriba)
#
# Resultado final: 2 commits
#   a1b2c3' feat: add login form         (= a1b2c3 + d4e5f6 + g7h8i9)
#   j0k1l2' feat: login validation       (= j0k1l2 + k2l3m4, con editor de mensaje)
#
# ❌ ERROR COMÚN: poner squash/fixup en el PRIMER commit de la lista
#   squash a1b2c3 feat: add login form   ← ¡ERROR! No hay nadie "arriba"
#   pick   d4e5f6 wip                   ← ¿con quién se fusiona a1b2c3?
#
# ─────────────────────────────────────────────
# ¿Qué pasa con los MERGE COMMITS en el editor?
# ─────────────────────────────────────────────
# Un merge commit es el que se crea cuando haces "git merge main" en tu rama.
# En git log se ve como:  "Merge branch 'main' into feature"
#
# Si ese merge commit cae dentro del rango HEAD~N que elegiste,
# también aparece en el editor. Aquí hay que tener cuidado:
#
#   pick   a4f8c2e feat: add login form        ← tuyo
#   pick   c1b0e3d Merge branch 'main'...      ← merge commit ⚠️
#   pick   d7e2f4a fix: validación email       ← tuyo
#   pick   f3a9b1c fix: typo en formulario     ← tuyo
#
# ¿Qué puedes hacer con el merge commit?
#   → pick:  déjalo como está. El historial se preserva con el merge.
#   → drop:  lo eliminas. Git "aplana" la historia como si nunca
#            hubieras hecho merge de main. Cuidado: los cambios de
#            main que resolviste en ese merge se perderán del historial
#            pero sus commits ya están incluidos de otra forma.
#   → squash/fixup sobre él: ❌ NO LO HAGAS.
#            squash fusiona el merge commit con el commit de ARRIBA (a4f8c2e).
#            Resultado: tu commit de login form absorbe TODOS los cambios
#            que vinieron de main → un commit enorme y confuso.
#
# ✅ CONSEJO PRÁCTICO: para evitar tener merge commits en el editor,
#    antes de hacer rebase -i limpia la rama con rebase sobre main:
#
#    git rebase origin/main        ← esto elimina el merge commit y
#                                     recoloca tus commits encima de main
#    git rebase -i HEAD~N          ← ahora N son SOLO tus commits propios


# ─────────────────────────────────────────────
# EJEMPLO COMPLETO PASO A PASO (práctica guiada)
# ─────────────────────────────────────────────
# Situación: Llevas 3 días trabajando en feature/login.
# Tu historial actual (git log --oneline):
#
#   k9j8h7 (HEAD) fix: otro typo más         ← tuyo (más reciente)
#   i6h5g4 wip: guardando por si acaso       ← tuyo
#   e3d2c1 fix: typo en el form              ← tuyo
#   b0a9z8 feat: add login validation        ← tuyo
#   x7y6w5 feat: add login form              ← tuyo (más antiguo)
#   9341630 (main) chore: project setup      ← de main ← STOP
#
# Tienes 5 commits propios. Quieres dejarlos en 2 commits limpios:
#   1. "feat: add login form + validation"
#   2. "fix: typo en formulario de login"
#
# PASO 1: Abre el editor
git rebase -i HEAD~5
#
# PASO 2: El editor muestra (del más antiguo al más reciente):
#
#   pick x7y6w5 feat: add login form
#   pick b0a9z8 feat: add login validation
#   pick e3d2c1 fix: typo en el form
#   pick i6h5g4 wip: guardando por si acaso
#   pick k9j8h7 fix: otro typo más
#
# PASO 3: Editas el archivo para que quede así:
#
#   pick  x7y6w5 feat: add login form          ← BASE del primer commit
#   squash b0a9z8 feat: add login validation   ← se fusiona con x7y6w5 (el de arriba)
#                                                 te abre editor para combinar mensajes
#   pick  e3d2c1 fix: typo en el form          ← BASE del segundo commit
#   fixup i6h5g4 wip: guardando por si acaso  ← se fusiona con e3d2c1 (silenciosamente)
#   fixup k9j8h7 fix: otro typo más            ← se fusiona con el resultado anterior
#
# PASO 4: Guardas y cierras el editor.
#
# Git empieza a procesar de ARRIBA a ABAJO:
#
#   [1/5] Aplica x7y6w5  → OK
#   [2/5] Aplica b0a9z8  → squash: abre editor para el mensaje combinado
#         Tú escribes:   "feat: add login form with validation"
#         Guardas y cierras.
#   [3/5] Aplica e3d2c1  → OK
#   [4/5] Aplica i6h5g4  → fixup: se fusiona sin preguntar
#   [5/5] Aplica k9j8h7  → fixup: se fusiona sin preguntar
#
# RESULTADO FINAL (git log --oneline):
#
#   NEW2 (HEAD) fix: typo en el form            ← contiene e3d2c1 + i6h5g4 + k9j8h7
#   NEW1 feat: add login form with validation   ← contiene x7y6w5 + b0a9z8
#   9341630 (main) chore: project setup
#
# De 5 commits desordenados → 2 commits limpios y descriptivos ✅


# ============================================
# 3. --onto: mover rama a una base diferente
# ============================================
# Situación: Creaste feature-b A PARTIR DE feature-a por error,
# pero en realidad feature-b debía salir de main.
#
# Antes:
#   main:      A---B
#   feature-a:      C---D
#   feature-b:           E---F  ← creada desde feature-a
#
# Quieres:
#   main:      A---B
#   feature-a:      C---D
#   feature-b:  B---E'--F'  ← ahora parte de main
#
git rebase --onto main feature-a feature-b
# Sintaxis: git rebase --onto <nueva-base> <desde-donde> <hasta-donde>
# → feature-b ahora parte de main, sin incluir los commits de feature-a


# ============================================
# 4. --autosquash: squash automático por mensajes
# ============================================
# Situación: Estás en mitad de feature/payment y descubres un bug.
# Haces un commit de fix y lo marcas para que se fusione automáticamente:
git commit -m "fixup! feat: add payment form"
# ó
git commit -m "squash! feat: add payment form"

# Luego, al hacer rebase interactivo:
git rebase -i --autosquash main
# → Git automáticamente reorganiza y marca ese commit como fixup/squash
# → No necesitas editar manualmente el editor


# ============================================
# 5. --autostash: stash automático
# ============================================
# Situación: Tienes cambios en working directory sin commitear
# y quieres hacer rebase sin perderlos.
git rebase --autostash main
# → Hace stash automáticamente antes del rebase
# → Aplica el stash al terminar
# → Sin esto, Git te daría error si tienes cambios sin commitear


# ============================================
# 6. --exec: ejecutar comando tras cada commit
# ============================================
# Situación: Quieres asegurarte de que los tests pasan
# en CADA commit intermedio (útil antes de merge a main).
git rebase -i --exec "npm test" HEAD~5
# → Tras aplicar cada commit, ejecuta npm test
# → Si falla, el rebase se pausa para que puedas arreglar


# ============================================
# 7. Continuar, saltar o abortar tras conflicto
# ============================================
# Cuando hay conflicto durante el rebase, Git para y muestra:
# "CONFLICT (content): Merge conflict in archivo.js"

# Paso 1: Ver qué archivos tienen conflicto
git status

# Paso 2: Editar los archivos y resolver los conflictos
# (eliminar los marcadores <<<<<<, ======, >>>>>>>)

# Paso 3: Marcar como resuelto
git add archivo-resuelto.js

# Paso 4: Continuar
git rebase --continue

# Si el conflicto de este commit no tiene sentido y quieres saltarlo:
git rebase --skip

# Si quieres cancelar todo el rebase y volver al estado inicial:
git rebase --abort


# ============================================
# 8. --update-refs: actualizar ramas dependientes (Git 2.38+)
# ============================================
# Situación: Tienes varias ramas apiladas (feature-a → feature-b → feature-c)
# y rebases main. Quieres que todas las ramas intermedias se actualicen.
git rebase --update-refs main
# → Actualiza automáticamente todas las ramas que apuntan a commits
#   que fueron reescritos durante el rebase
```

**Casos de uso reales:** [🔙](#9-git-rebase---reescribiendo-historia)

```bash
# ─────────────────────────────────────────────────────────────────
# CASO 1: Preparar PR limpio antes de merge
# ─────────────────────────────────────────────────────────────────
# Situación: Llevas 2 semanas en feature/user-profile.
# Tienes 12 commits: muchos "wip", "fix", "tmp"...
# El equipo pide que el PR tenga máximo 3 commits bien descritos.
#
# Paso 1: Actualizar con los últimos cambios de main
git fetch origin
git rebase origin/main

# Paso 2: Limpiar historial (últimos 12 commits)
git rebase -i HEAD~12
# En el editor:
# - Deja el primero como "pick"
# - Los "wip/fix" los pones como "fixup"
# - Los hitos importantes como "pick" o "reword"

# Paso 3: Push (necesita force porque reescribiste historia)
git push --force-with-lease origin feature/user-profile


# ─────────────────────────────────────────────────────────────────
# CASO 2: Actualizar feature branch cada día
# ─────────────────────────────────────────────────────────────────
# Situación: Tu rama feature/checkout lleva 1 semana.
# Cada mañana, antes de empezar, sincronizas con main.
git fetch origin
git rebase origin/main
# Si hay conflictos: resolverlos y git rebase --continue
# → Tus commits siempre están encima de lo último de main
# → Cuando hagas el merge final, no habrá conflictos ni divergencias


# ─────────────────────────────────────────────────────────────────
# CASO 3: Dividir un commit enorme en dos
# ─────────────────────────────────────────────────────────────────
# Situación: Hiciste un commit con cambios del modelo Y la vista,
# y quieres separarlos en dos commits distintos.
git rebase -i HEAD~1
# Cambia "pick" por "edit" en ese commit

# Git para en ese commit. Ahora deshaces el commit (sin perder cambios):
git reset HEAD~1
# Ahora tienes todos los cambios en working directory

# Añades solo los archivos del modelo:
git add models/user.js
git commit -m "feat: add user model"

# Añades los de la vista:
git add views/user.html
git commit -m "feat: add user profile view"

# Continúas el rebase:
git rebase --continue


# ─────────────────────────────────────────────────────────────────
# CASO 4: Mover rama creada desde la rama equivocada
# ─────────────────────────────────────────────────────────────────
# Situación: Creaste feature/hotfix desde feature/big-refactor por error.
# Quieres que salga directamente de main.
git rebase --onto main feature/big-refactor feature/hotfix
# → feature/hotfix ahora parte de main
# → Los commits de big-refactor no están incluidos
```

**Rebase vs Merge:** [🔙](#9-git-rebase---reescribiendo-historia)

```bash
# ─────────────────────────────────────────────────────────────────
# MERGE: preserva la historia tal como ocurrió
# ─────────────────────────────────────────────────────────────────
git checkout main
git merge feature/login
#
# Resultado:
#   main:  A---B---C---M   ← M = merge commit
#                    \ /
#   feat:             D---E
#
# ✓ Muestra exactamente cuándo se integró la feature
# ✓ No reescribe commits existentes
# ✓ Seguro para ramas públicas compartidas
# ✗ Historial más complejo de leer (ramificaciones)


# ─────────────────────────────────────────────────────────────────
# REBASE: historia lineal más limpia
# ─────────────────────────────────────────────────────────────────
git checkout feature/login
git rebase main
git checkout main
git merge feature/login  # Fast-forward automático
#
# Resultado:
#   main:  A---B---C---D'--E'   ← historia lineal
#
# ✓ Historial fácil de leer y seguir con git log
# ✓ Fácil de buscar cuándo se introdujo un bug (git bisect)
# ✗ Reescribe commits (nuevos hashes)
# ✗ Peligroso si ya habías pusheado esos commits
```

**⚠️ Regla de oro del rebase:** [🔙](#9-git-rebase---reescribiendo-historia)

```
NUNCA hagas rebase de commits que ya están en un repositorio
compartido (público) y que otros han descargado.

¿Por qué? Porque rebase crea commits NUEVOS (nuevos hashes).
Si alguien tiene los commits originales y tú cambias la historia,
sus repositorios divergen y los merge posteriores son un caos.

✅ CORRECTO: rebase de commits solo en tu máquina local
✅ CORRECTO: rebase de tu feature branch antes del primer push
✅ CORRECTO: rebase de tu feature branch personal (nadie más trabaja en ella)

❌ INCORRECTO: rebase de main, develop o cualquier rama compartida
❌ INCORRECTO: rebase después de haber pusheado y otros descargaron
```

**Troubleshooting:** [🔙](#9-git-rebase---reescribiendo-historia)

```bash
# ─────────────────────────────────────────────────────────────────
# Problema 1: Conflicto durante rebase
# ─────────────────────────────────────────────────────────────────
# Git muestra:
# CONFLICT (content): Merge conflict in src/auth.js
# error: could not apply abc1234... feat: add token validation

# Solución:
git status                        # Ver qué archivos tienen conflicto
# Edita los archivos marcados con <<<<<<< HEAD ... >>>>>>> abc1234
git add src/auth.js               # Marcar como resuelto
git rebase --continue             # Continuar

# Si no sabes cómo resolver y quieres volver al estado inicial:
git rebase --abort                # Cancela todo, vuelves a donde estabas


# ─────────────────────────────────────────────────────────────────
# Problema 2: Commits duplicados tras rebase
# ─────────────────────────────────────────────────────────────────
# Situación: Hiciste push, luego rebase, luego push --force.
# Al hacer pull, los commits aparecen duplicados.
# Causa: otro compañero hizo pull antes de tu force push.
# Solución: hablar con el equipo y hacer:
git pull --rebase                 # Rebase local sobre la historia reescrita


# ─────────────────────────────────────────────────────────────────
# Problema 3: Me arrepiento del rebase, quiero deshacer
# ─────────────────────────────────────────────────────────────────
git reflog                        # Ver el estado antes del rebase
# Busca la línea: "HEAD@{5}: rebase: start"
# Justo ANTES de eso está tu estado original
git reset --hard HEAD@{5}         # Vuelve al estado anterior al rebase


# ─────────────────────────────────────────────────────────────────
# Problema 4: Push rechazado después de rebase
# ─────────────────────────────────────────────────────────────────
# Error: "Updates were rejected because the tip of your current branch is behind"
# Causa: Reescribiste historia, el remoto tiene commits que tú ya no tienes.
git push --force-with-lease       # Fuerza el push, pero solo si nadie más actualizó
# NUNCA usar --force a secas en ramas que otros usan
```

**Configuración recomendada:** [🔙](#9-git-rebase---reescribiendo-historia)

```bash
# Usar rebase por defecto en git pull (en vez de merge)
git config --global pull.rebase true

# Activar autosquash siempre en rebase interactivo
git config --global rebase.autoSquash true

# Activar autostash automático durante rebase
git config --global rebase.autoStash true

# Ver configuración actual
git config --list | grep rebase
```

**Mejores prácticas:** [🔙](#9-git-rebase---reescribiendo-historia)

```bash
✓ Usa rebase para limpiar tu historia LOCAL antes de hacer PR
✓ Haz rebase de tu feature sobre main antes del merge para evitar conflictos tardíos
✓ Usa --force-with-lease nunca --force
✓ Usa rebase -i para preparar commits limpios y descriptivos
✓ Ante duda, git rebase --abort cancela sin consecuencias

✗ NUNCA hagas rebase de ramas que otros estén usando (main, develop, shared)
✗ NUNCA hagas rebase después de que otros descargaron tus commits
✗ No uses rebase si no entiendes qué commits vas a reescribir
```

---

## Navegación

- [⬅️ Anterior: git merge](08-git-merge.md)
- [🏠 Volver al Índice](../../GIT_COMANDOS_GUIA_PRACTICA.md)
- [➡️ Siguiente: git clone](10-git-clone.md)
