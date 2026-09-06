# AGENTS.md

## Ejecución del proyecto

- Sin herramientas de build, sin package.json, sin tests, sin lint. Archivos estáticos puros (Vanilla JS + HTML + CSS).
- Abrir `src/index.html` en un navegador para ejecutar. No se necesita servidor de desarrollo ni compilación.
- La verificación es manual: abrir en navegador y jugar.

## Arquitectura

- **Espacio de nombres global, no ES modules.** Cada archivo JS expone su API vía `window.X = X`. No añadir `import`/`export` ni un bundler.
- **El orden de carga de scripts es obligatorio** (definido en `src/index.html`): `maze.js` → `game.js` → `render.js` → `main.js`. Reordenar rompe la app.
- **`MAZE` es inmutable y nunca se muta.** `createGame()` lo copia a `game.grid`. Toda la lectura/escritura del juego pasa por `game.grid`.
- **Coordenadas de la grid:** `grid[y][x]`, origen arriba-izquierda. x ∈ [0,27], y ∈ [0,30]. TILE = 20px, canvas 560×620.

### Valores de celda

| Valor | Carácter en MAZE_STR | Significado |
|-------|----------------------|-------------|
| 0     | ` ` (espacio)        | Vacío transitable |
| 1     | `#`                  | Muro |
| 2     | `.`                  | Punto (dot, +10 pts) |
| 3     | `-`                  | Puerta de la pen de fantasmas |
| 4     | `@`                  | Power pellet (+50 pts, activa frightened mode) |

### API global por archivo

**maze.js:**
- `window.MAZE` — number[31][28], grid pristina.
- `window.TUNNEL_ROW` — 14 (fila del túnel wrap-around).
- `window.PACMAN_START` — `{x:13, y:23}`.
- `window.GHOST_STARTS` — Array[4] de `{x,y,kind}`, fila 14 cols 12–15.

**game.js:**
- `window.createGame()` → objeto game (estado inicial).
- `window.update(game)` — un tick: mover pacman, activar fantasmas, mover fantasmas, colisiones, win/lose.
- `window.DIRS` — vectores `{left,right,up,down}` → `{x,y}`.

**render.js:**
- `window.draw(ctx, game, frame)` — dibuja todo (muros, puntos, pacman, fantasmas, HUD).

**main.js:** sin exports. Game loop (`requestAnimationFrame`) + input de teclado (flechas) + overlays start/win/lose.

### Estructura del objeto game

```js
{
  state: 'start' | 'playing' | 'won' | 'lost',
  score, lives(3), dotsRemaining, grid(number[][]),
  frightUntil(null|timestamp), ghostsEaten(0-3),
  pacman: {x, y, dir, nextDir, speed},
  ghosts: [{x,y,dir,speed,kind,active,activateAt,eaten,eatenAt}] ×4
}
```

### Velocidades

- Pac-Man: **0.125** celdas/frame (se alinea cada 8 frames).
- Fantasmas: **0.1** celdas/frame (se alinean cada 10 frames).
- Ojos navegando de vuelta a la pen: **2× velocidad del fantasma** (`g.speed * 2`), sin modificar `g.speed`.

### IA de fantasmas

| Fantasma | Color   | Comportamiento |
|----------|---------|----------------|
| Blinky   | Rojo    | Persecución directa a Pac-Man |
| Pinky    | Rosa    | Emboscada: apunta 4 celdas delante de Pac-Man según su dirección |
| Inky     | Cian    | Flanco: refleja posición de Pac-Man a través de Blinky (`tx = bx + 2*(px-bx)`) |
| Clyde    | Naranja | Tímido: si distancia Manhattan ≤8 → aleatorio; si no, persigue |

- **Activación escalonada:** cada fantasma se activa `i × 2000ms` tras el inicio (usa `performance.now()`). Antes de activarse queda visible pero congelado.
- **Salida de pen:** lógica explícita — navega a la columna más cercana de la puerta (13 o 14) y sube. Solo aplica cuando NO está comido; los ojos usan greedy Manhattan directo a su celda exacta dentro del área abierta del pen.
- **Frightened mode** (power pellet): fantasmas azules, dirección invertida al comerse el pellet, comestibles con puntuación 200/400/800/1600 en cadena. Últimos 4s: parpadeo blanco/azul (warning).
- **Ojos de vuelta a la pen** (`g.eaten === true`): al ser comido se hace snap de posición a entero y `eatenAt = performance.now()`. Los ojos navegan por corredores válidos hacia `GHOST_STARTS[i]` a velocidad 2×. Al llegar alineados → reaparecen (`eaten=false`, `dir='up'`). **Timeout de seguridad:** si tras 10 s no llegaron, se teletransportan a la celda de inicio. Los ojos NO provocan colisión ni son re-comidos por Pac-Man.
- **Túnel:** fila 14, wrap-around horizontal cuando x sale de [0,27].

### Reglas de movimiento

- `isWall(grid,x,y,actor)`: pacman bloqueado por muros(1) y puertas(3); fantasmas solo por muros(1).
- `canMove` valida dirección + túnel.
- Colisión: |dx| < 0.5 AND |dy| < 0.5 (se omite si el fantasma está comido).

## Flujo spec-driven

El proyecto usa **dos** sistemas de especificaciones que coexisten; elegir según la tarea:

1. **OpenSpec** (`openspec/`) — workflow experimental con skills `/opsx-*` y CLI `openspec`.
   - Specs principales en `openspec/specs/<capability>/spec.md`; cambios activos en `openspec/changes/<name>/`, archivados en `openspec/changes/archive/YYYY-MM-DD-<name>/`.
   - Ciclo: proponer → implementar (`/opsx-apply`) → archivar (`/opsx-archive`).

2. **Specs legacy** (`specs/NN-slug.md`) — numeradas, con skills `/spec` (crear) y `/spec-impl` (implementar). Una spec debe estar en estado `Approved` antes de implementar. Nomenclatura de ramas: `spec-NN-slug`.

### Specs completadas / en curso

| Spec | Título | Estado |
|------|--------|--------|
| 01   | Four Ghost AI | Merged (PR #1) |
| 02   | Staggered Ghosts & Pen Exit | Merged (PR #2) |
| 03   | Power Pellets & Frightened Mode | Merged (PR #3, en `main`) |
| —    | ghost-eyes-return-path (OpenSpec) | Archivado (`openspec/changes/archive/`), spec en `openspec/specs/ghost-eyes-return/` |
