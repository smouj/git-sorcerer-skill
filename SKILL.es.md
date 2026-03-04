title: "Git Sorcerer"
description: "Encanta flujos de trabajo de git y resuelve conflictos de merge con sabiduría arcana"
version: "1.0.0"
author: "FLICKCLAW-OPS"
tags: ["git", "workflow", "conflict-resolution", "version-control"]
maintainer: "SMOUJBOT"
last_updated: "2026-03-04"
type: "devops"
dependencies: ["git>=2.40", "bash>=5.0", "jq (optional)"]
environment: ["GIT_EDITOR", "GIT_MERGE_AUTOEDIT", "GIT_PAGER"]
```

# Git Sorcerer

## Propósito

Git Sorcerer automatiza y encanta operaciones git complejas para la gestión de repositorios de FlickClaw SaaS. Casos de uso reales:

- **Resurrección automatizada de ramas**: Restaurar ramas eliminadas o huérfanas después de eliminación accidental
- **Arbitraje inteligente de conflictos**: Resolver conflictos de merge usando algoritmos de three-way diff con merge drivers personalizados
- **Orquestación de releases**: Automatizar versionado, generación de changelog y releases etiquetadas con versionado semántico
- **Purificación de historial**: Reescribir historial de commits para eliminar datos sensibles preservando ascendencia
- **Magia de submódulos**: Sincronizar y actualizar dependencias anidadas en múltiples repositorios
- **Cascada de cherry-pick**: Aplicar múltiples commits entre ramas con estrategias automáticas de resolución de conflictos
- **Automatización de hooks**: Desplegar git hooks que enforcing políticas de code review y ejecutan gates de CI

## Alcance

### Comandos Gestionados

| Comando | Uso Principal | Flags/Opciones |
|---------|---------------|----------------|
| `git merge` | Integración de ramas | `--no-commit`, `--no-ff`, `--squash`, `-X ours/theirs` |
| `git rebase` | Historial lineal | `-i`, `--autosquash`, `--exec`, `--rebase-merges` |
| `git cherry-pick` | Transferencia de commits | `-x`, `--skip`, `--continue`, `--abort`, `-n` |
| `git revert` | Deshacer seguro | `-n`, `--no-edit`, `--strategy-option` |
| `git bisect` | Caza de bugs | `start`, `good`, `bad`, `view`, `skip`, `reset` |
| `git filter-branch` / `filter-repo` | Reescribir historial | `--tree-filter`, `--index-filter`, `--commit-filter` |
| `git worktree` | Múltiples checkouts | `add`, `remove`, `list`, `prune` |
| `git submodule` | Repos anidados | `update`, `sync`, `foreach`, `status` |
| `git reflog` | Recuperación | `show`, `delete`, `expire` |

### Herramientas Requeridas

- `git` - Control de versiones core
- `bash` - Shell scripting
- `jq` - Procesamiento JSON (opcional pero recomendado)
- `sed`/`awk` - Manipulación de texto
- `git-filter-repo` - Reescritura moderna de historial (instalar via pip)

## Proceso de Trabajo

### 1. Evaluación Inicial

```bash
# Clone FlickClaw repository into isolated workspace
git clone --recurse-submodules https://github.com/smouj/flickclaw-saas.git /tmp/flickclaw-work

cd /tmp/flickclaw-work

# Analyze repository state
git status --porcelain=v1
git branch -vv
git log --oneline --graph --decorate --all -20
git remote -v
git submodule status
```

### 2. Protocolo de Resolución de Conflictos

Cuando surjan conflictos de merge:

```bash
# Step 1: Identify conflict type
git diff --name-only --diff-filter=U

# Step 2: Classify conflicts
# - Text files: Use custom merge driver
# - JSON/Config: Use jq to merge objects
# - Binary: Use "theirs" or "ours" strategy

# Step 3: Execute resolution strategy
# For JSON files:
jq -s '.[0] * .[1]' file.json.base file.json.theirs > file.json

# For code files with custom markers:
git checkout --conflict=merge file.ts
# Manually edit <<<<<<< markers

# Step 4: Mark as resolved
git add file.json file.ts

# Step 5: Continue merge
git merge --continue

# Step 6: Verify resolution
git diff --check
```

### 3. Ritual de Resurrección de Ramas

```bash
# Find lost commit (reflog)
git reflog | grep "deleted branch"

# Create branch from commit hash
git branch recovered-branch <commit-hash>

# Push if remote needed
git push origin recovered-branch
```

### 4. Ingeniería de Releases

```bash
# 1. Ensure clean state
git status
git pull origin develop

# 2. Create release branch
git checkout -b release/v1.2.3

# 3. Bump version (example for Node.js)
npm version patch -m "chore(release): %s"

# 4. Generate changelog
git log --pretty=format:'- %s' $(git describe --tags --abbrev=0)..HEAD > CHANGELOG.md

# 5. Merge to main with squash
git checkout main
git merge --squash release/v1.2.3
git commit -m "release: v1.2.3"

# 6. Tag release
git tag -a v1.2.3 -m "Release v1.2.3"

# 7. Push
git push origin main --tags
git branch -D release/v1.2.3
```

### 5. Purificación de Historial (Seguridad)

```bash
# Install git-filter-repo if not present
pip3 install git-filter-repo

# Remove sensitive data from entire history
git filter-repo --replace-text <(echo "SECRET_KEY==>REDACTED")

# Or remove specific file
git filter-repo --path path/to/secrets.env --invert-paths

# Force push (CAUTION)
git push origin --force --all
git push origin --force --tags
```

## Reglas de Oro

1. **Nunca hacer force push a ramas compartidas** sin aprobación explícita del equipo
2. **Siempre crear rama de backup** antes de operaciones destructivas:
   ```bash
   git branch backup/main-$(date +%Y%m%d-%H%M%S) main
   ```
3. **Probar rebase/cherry-pick en worktree aislado** primero:
   ```bash
   git worktree add /tmp/test-rebase <base-branch>
   ```
4. **Preservar commits de merge** cuando sea posible (`--rebase-merges`) para audit trail
5. **Usar commits firmados** (`-S`) para releases de producción
6. **Ejecutar validación de CI** antes de push de cualquier historial reescrito
7. **Comunicar** cualquier force push al equipo via Slack antes de ejecución
8. **Mantener** backup de reflog por mínimo 30 días:
   ```bash
   git config --global gc.reflogExpireUnreachable 30.days
   ```

## Ejemplos

### Ejemplo 1: Resolución Automática de Conflictos

**Prompt del Usuario:**
"Resuelve conflictos de merge en src/auth/ usando estrategia 'theirs' para binarios, manual para código"

**Ejecución:**
```bash
cd /tmp/flickclaw-work
git merge origin/feature/auth-refactor

# Configure merge driver for binaries (once per repo)
echo "*.png binary merge=ours" >> .gitattributes
echo "*.jpg binary merge=ours" >> .gitattributes

# For specific files:
git checkout --theirs src/auth/logo.png
git add src/auth/logo.png

# For code files, launch mergetool
git mergetool src/auth/login.ts  # Configured vimdiff or VS Code

# Complete
git merge --continue
```

**Verificación:**
```bash
git diff --name-only --diff-filter=U  # Should be empty
git log -1 --pretty=fuller  # Shows merge commit with both parents
```

### Ejemplo 2: Cascada de Cherry-Pick

**Prompt del Usuario:**
"Aplicar commits abc123, def456, ghi789 de develop a staging, omitir los que fallen"

**Ejecución:**
```bash
cd /tmp/flickclaw-work
git checkout staging

# Create script for sequential cherry-picks
cat > /tmp/cherry-script.sh << 'EOF'
#!/bin/bash
commits=("abc123" "def456" "ghi789")
for commit in "${commits[@]}"; do
  echo "Applying $commit..."
  if ! git cherry-pick $commit 2>/dev/null; then
    echo "Conflict on $commit, skipping..."
    git cherry-pick --skip
  fi
done
EOF

bash /tmp/cherry-script.sh

# Commit skipped ones manually later
```

**Verificación:**
```bash
git log --oneline staging | grep -E "abc123|def456|ghi789"
git cherry -v develop staging  # Shows which commits are missing
```

### Ejemplo 3: Hotfix Producción

**Prompt del Usuario:**
"Crear hotfix para CVE-2024-1234 en producción, mergear de vuelta a develop"

**Ejecución:**
```bash
cd /tmp/flickclaw-work

# 1. Create hotfix branch from production tag
git checkout -b hotfix/CVE-2024-1234 production/v2.1.0

# 2. Apply security patch
# (edit files...)

git add .
git commit -S -m "fix(security): patch CVE-2024-1234

- Close potential RCE vulnerability in user upload
- Sanitize filename extensions
- Add content-type validation"

# 3. Bump version
npm version patch -m "fix(security): %s [CVE-2024-1234]"

# 4. Merge to production
git checkout production
git merge --no-ff hotfix/CVE-2024-1234 -m "release: hotfix CVE-2024-1234"
git tag -s v2.1.1 -m "Security hotfix v2.1.1"

# 5. Merge back to develop
git checkout develop
git merge --no-ff hotfix/CVE-2024-1234

# 6. Push all
git push origin production --tags
git push origin develop
git push origin hotfix/CVE-2024-1234

# 7. Cleanup
git branch -D hotfix/CVE-2024-1234
```

### Ejemplo 4: Recuperar Trabajo Perdido

**Prompt del Usuario:**
"Recuperar commits de ayer que se perdieron después de un reset malo"

**Ejecución:**
```bash
cd /tmp/flickclaw-work

# Search reflog for dangling commits
git reflog --all --since="1 day ago" | head -20

# Find commit hash (e.g., abc1234)
git show abc1234  # Verify it's the right one

# Create branch
git branch recovered/feature-work abc1234

# If commit not in reflog, use fsck
git fsck --lost-found | grep commit
cat .git/lost-found/commit/* | git show  # Browse recovered

# Push to remote for safety
git push origin recovered/feature-work
```

## Comandos de Rollback

### Rollback de Merge
```bash
# Abort merge in progress
git merge --abort

# If merge already committed
git revert -m 1 <merge-commit-hash>  # Keep both parents
# OR for complete undo:
git reset --hard ORIG_HEAD  # CAUTION: discards working tree
```

### Rollback de Rebase
```bash
# Abort rebase
git rebase --abort

# If rebase completed and pushed
git revert <oldest-rebased-commit>..HEAD
# OR (dangerous):
git reset --hard <commit-before-rebase>
git push --force-with-lease  # If remote updated
```

### Rollback de Cherry-Pick
```bash
git cherry-pick --abort  # If in progress
git revert <cherry-picked-commit>  # If completed
```

### Rollback de Filter-Repo/Rewrite
```bash
# If you backed up first:
git checkout backup/pre-filter-<timestamp>

# If not, recover from reflog within 30 days:
git reflog | grep "filter-repo"
git checkout -b recovery <reflog-hash>
```

### Rollback de Submodule
```bash
git submodule deinit -f path/to/submodule
git rm -f path/to/submodule
rm -rf .git/modules/path/to/submodule
git commit -m "chore: remove broken submodule"
```

### Reset Completo de Proyecto (Nuclear)
```bash
# WARNING: Irreversible without remote backup
git reset --hard $(git reflog | head -1 | cut -d' ' -f1)
git clean -fdx  # Remove untracked files too
git submodule foreach --recursive git reset --hard
git submodule foreach --recursive git clean -fdx
```

## Solución de Problemas

### "Refusing to merge unrelated histories"
```bash
# Use with caution - only if repositories were independently created
git merge other-branch --allow-unrelated-histories
```

### "error: Your local changes to the following files would be overwritten by merge"
```bash
# Option A: Stash changes
git stash push -m "pre-merge-stash"
git merge <branch>

# Option B: Force checkout (discard changes)
git checkout --force <branch>

# Option C: Merge with ours strategy for specific file
git checkout --ours path/to/file
```

### "fatal: refusing to merge unrelated histories" with submodules
```bash
# Initialize submodules properly
git submodule update --init --recursive
git submodule sync --recursive
```

### "detached HEAD" state after cherry-pick
```bash
# Create branch to preserve work
git checkout -b temp/cherry-pick-work
# Then merge or rebase as needed
```

### "Permission denied (publickey)" when pushing
```bash
# Verify SSH agent
ssh-add -l  # List loaded keys
ssh-add ~/.ssh/id_rsa_flickclaw  # Add correct key

# Check remote URL
git remote -v
# If using HTTPS but want SSH:
git remote set-url origin git@github.com:smouj/flickclaw-saas.git
```

### Merge conflicts in `.gitmodules`
```bash
# Treat as ini file
git config --global merge.renormalize true
git checkout --ours .gitmodules
git add .gitmodules
```

## Pasos de Verificación

Después de cualquier operación git:

1. **Verificar estado de rama**:
   ```bash
   git branch -vv  # Todas las ramas con upstream tracking
   git status -sb   # Estado corto con info de rama
   ```

2. **Validar grafo de commits**:
   ```bash
   git log --graph --oneline --all -30
   # Buscar divergencias inesperadas o commits perdidos
   ```

3. **Verificar que no hay cambios sin commitear**:
   ```bash
   git diff-index --quiet HEAD --  # Returns 0 if clean
   git status --porcelain | wc -l  # Should be 0
   ```

4. **Probar build/test suite**:
   ```bash
   npm test
   npm run build
   # Or project-specific: pnpm test, yarn build, etc.
   ```

5. **Validar sync con remoto**:
   ```bash
   git fetch origin
   git log --oneline HEAD..origin/main  # Should be empty if up-to-date
   ```

6. **Verificar integridad de reflog**:
   ```bash
   git reflog --date=iso | head -10
   ```

## Variables de Entorno

| Variable | Propósito | Default | Requerido |
|----------|-----------|---------|-----------|
| `GIT_EDITOR` | Editor de commit messages | `vim` | No |
| `GIT_MERGE_AUTOEDIT` | Auto-open editor on merge | `yes` | No |
| `GIT_PAGER` | Pager para logs/diffs | `less` | No |
| `GIT_SSH_COMMAND` | SSH custom for git | - | No |
| `GIT_USER_NAME` | Override git user.name | from config | No |
| `GIT_USER_EMAIL` | Override git user.email | from config | No |

Setup example:
```bash
export GIT_EDITOR="code --wait"
export GIT_MERGE_AUTOEDIT=no  # For automated workflows
export GIT_SSH_COMMAND="ssh -i ~/.ssh/id_rsa_flickclaw -o IdentitiesOnly=yes"
```

## Dependencias & Setup

### Prerrequisitos
```bash
# Verify git version
git --version  # Must be >= 2.40

# Install git-filter-repo (for history rewriting)
pip3 install --user git-filter-repo

# Optional: jq for JSON manipulation
apt-get install jq  # Debian/Ubuntu
brew install jq     # macOS

# Configure git for this project
git config --local user.name "FlickClaw Bot"
git config --local user.email "devops@flickclaw.com"
git config --local commit.gpgsign true
```

### One-time Setup for FlickClaw Repository
```bash
cd /tmp/flickclaw-work

# Set upstream tracking
git branch --set-upstream-to=origin/main main

# Configure merge drivers for common file types
cat > .gitattributes << 'EOF'
*.json merge=json
*.md merge=union
*.lock merge=ours
*.png binary
*.jpg binary
EOF

# Install custom merge driver for JSON
git config --local merge.json.name "JSON merge driver"
git config --local merge.json.driver "jq -s '.[0] * .[1]' %A %O > %A"
```

## Notas

- Todas las operaciones deben ejecutarse en `/tmp/flickclaw-work` para evitar corrupción del entorno OpenClaw en uso
- Siempre crear ramas de backup antes de operaciones destructivas
- Para deployments de producción, usar `--force-with-lease` en vez de `--force` al push de historial reescrito
- Coordinar con equipo antes de reescribir historial de ramas compartidas
- Mantener esta skill idempotente - ejecutar la misma operación dos veces no debe causar efectos secundarios
- Loggear todas las operaciones a `/tmp/flickclaw-work/git-sorcerer.log` para audit trail
```