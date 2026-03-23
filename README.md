# multipc-sync

Comando de Claude Code para sincronizar repositorios Git entre multiples PCs.

Detecta automaticamente el modo (PUSH/PULL/SYNC), commitea cambios pendientes, y hace push/pull de todos los repos configurados + cualquier subrepo descubierto dentro de los directorios monitorizados.

## Instalacion

### 1. Copiar el comando a tu Claude Code

```bash
# Crear directorio de comandos si no existe
mkdir -p ~/.claude/commands

# Copiar el comando
cp multipc.md ~/.claude/commands/multipc.md
```

### 2. Crear tu configuracion

```bash
# Copiar el template
cp config.example.yaml ~/.claude/multipc-config.yaml

# Editar con tus rutas y repos
nano ~/.claude/multipc-config.yaml
```

### 3. Usar

Dentro de Claude Code:
```
/multipc
```

El comando:
1. Lee tu `~/.claude/multipc-config.yaml`
2. Detecta si hay cambios locales (PUSH) o remotos (PULL)
3. Sincroniza todos los repos configurados
4. Auto-descubre subrepos git dentro de los directorios monitorizados
5. Muestra un reporte final

## Configuracion

Edita `~/.claude/multipc-config.yaml`:

```yaml
# Repos principales a sincronizar
repos:
  - name: "Mi Vault"
    path: "D:/Mi Vault"        # Ruta absoluta
    auto_discover: true         # Buscar subrepos dentro
    discover_depth: 5           # Profundidad maxima de busqueda
    conflict_strategy: "theirs" # Para archivos de cache (ej: .obsidian/)
    conflict_paths:             # Rutas donde aplicar conflict_strategy
      - ".obsidian/"
      - ".makemd/"

  - name: "claude-config"
    path: "~/.claude"
    preserve_local:             # Archivos que NO se sobreescriben en pull
      - "mcp.json"

  - name: "mi-proyecto"
    path: "~/mi-proyecto"

# Verificaciones post-sync (opcional)
checks:
  - name: "Python"
    command: "python --version"
  - name: "Git"
    command: "git --version"
```

### Campos de cada repo

| Campo | Requerido | Default | Descripcion |
|-------|-----------|---------|-------------|
| `name` | si | - | Nombre para el reporte |
| `path` | si | - | Ruta absoluta al repo (soporta `~`) |
| `auto_discover` | no | false | Buscar subrepos git dentro |
| `discover_depth` | no | 5 | Profundidad maxima de busqueda |
| `conflict_strategy` | no | - | "theirs" o "ours" para conflictos |
| `conflict_paths` | no | [] | Rutas donde aplicar la estrategia |
| `preserve_local` | no | [] | Archivos que no se tocan en pull |

## Modos de operacion

- **PUSH**: Hay cambios locales sin commitear -> commitea y sube
- **PULL**: No hay cambios locales -> baja cambios del remoto
- **SYNC**: Hay cambios en ambos lados -> push primero, luego pull

## Notas

- Los archivos en `preserve_local` (como `mcp.json`) nunca se sobreescriben porque pueden contener credenciales o config especifica del PC.
- Los subrepos descubiertos con `auto_discover` se sincronizan solo si tienen un remote configurado. Si no tienen remote, se reporta para que el usuario lo configure.
- El commit message incluye fecha y hostname para trazabilidad.
