---
name: multipc
description: Sincroniza repositorios Git entre PCs. Lee config de ~/.claude/multipc-config.yaml, detecta modo PUSH/PULL/SYNC, commitea, push/pull, auto-descubre subrepos. Ejecutar antes de cambiar de PC o al empezar sesion en PC nuevo.
---

# /multipc — Sincronizacion Multi-PC

Sincroniza todos los repositorios configurados entre ordenadores.

## Proceso

### 0. Leer configuracion

Lee `~/.claude/multipc-config.yaml` para obtener la lista de repos y sus opciones.

Si el archivo no existe, avisar al usuario:
- "No encontre ~/.claude/multipc-config.yaml. Copia config.example.yaml del repo multipc-sync y editalo con tus rutas."

Expandir `~` a `$HOME` en todas las rutas.

### 1. Detectar modo (PUSH o PULL)

Para cada repo configurado, comprobar si hay cambios locales sin commitear:
- Si hay cambios locales → modo **PUSH** (commitear y subir)
- Si no hay cambios locales → modo **PULL** (bajar cambios del otro PC)
- Si hay ambos → modo **SYNC** (push primero, luego pull)

Pregunta al usuario si no esta claro:
- "Vas a DEJAR este PC (push) o ACABAS DE LLEGAR (pull)?"

### 2. PUSH — Antes de cambiar de PC

Para cada repo en la config:

1. `cd [path]` — navegar al repo
2. `git status` — detectar archivos sin trackear y cambios
3. Revisar si hay archivos nuevos importantes
4. `git add -A`
5. `git commit -m "sync: [name] backup $(date +%Y-%m-%d_%H:%M) from $(hostname)"`
6. `git push`

#### Auto-discover subrepos
Para repos con `auto_discover: true`:
```bash
find "[path]" -name ".git" -maxdepth [discover_depth] -type d
```
Para cada subrepo encontrado:
- Verificar que tiene remote configurado (`git remote -v`)
- Si tiene remote: `git add -A && git commit && git push` si hay cambios
- Si NO tiene remote: reportar para que el usuario lo configure

### 3. PULL — Al llegar a un PC

Para cada repo en la config:

1. `cd [path]` — navegar al repo
2. `git fetch origin`
3. Comprobar si hay commits nuevos en remoto
4. Si el repo tiene `conflict_strategy` y hay conflictos en `conflict_paths`: resolver con la estrategia configurada
5. Si el repo tiene `preserve_local`: hacer backup de esos archivos antes del pull, restaurar despues
6. `git pull`
7. Reportar archivos nuevos/modificados que llegaron

#### Auto-discover subrepos
Para repos con `auto_discover: true`:
Para cada subrepo: `git pull` si tiene remote

### 4. Verificaciones post-sync

Si hay `checks` en la config, ejecutar cada uno y reportar resultado.

### 5. Reporte final

Mostrar resumen:
```
=== MULTIPC SYNC COMPLETE ===

Mode: [PUSH/PULL/SYNC]
Hostname: [nombre PC]

[Para cada repo:]
[name]:
  Status: [synced/pushed N commits/pulled N commits]
  New files: [lista si hay]

[Para cada subrepo descubierto:]
Subrepos:
  [path]: [status]

[Si hay checks:]
Checks:
  [name]: [OK/FAIL]

Ready to work!
```

## Notas importantes

- Archivos de cache de Obsidian (`.makemd/`, `.obsidian/`) generan conflictos frecuentes. Usar `conflict_strategy: "theirs"` para resolverlos automaticamente.
- Archivos en `preserve_local` (como `mcp.json`) contienen credenciales o config especifica del PC. Nunca se sobreescriben.
- El commit message incluye fecha y hostname para trazabilidad entre PCs.
- Subrepos sin remote se reportan pero no se sincronizan.
