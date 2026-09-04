# AGENTS.md

## Ejecución del proyecto

- Sin herramientas de build, sin package.json, sin tests, sin lint. Archivos estáticos puros.
- Abrir `src/index.html` en un navegador para ejecutar. No se necesita servidor de desarrollo.
- La verificación es manual: abrir en navegador y jugar.

## Arquitectura

- **Espacio de nombres global, no ES modules.** Cada archivo JS expone su API vía `window.X = X`. No añadir `import`/`export` ni un bundler.
- **El orden de carga de scripts es obligatorio** (definido en `index.html`): `maze.js` → `game.js` → `render.js` → `main.js`. Reordenar rompe la app.
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
  ghosts: [{x,y,dir,speed,kind,active,activateAt,eaten,reappearAt}] ×4
}
```

### Velocidades

- Pac-Man: **0.125** celdas/frame (se alinea cada 8 frames).
- Fantasmas: **0.1** celdas/frame (se alinean cada 10 frames).

### IA de fantasmas

| Fantasma | Color   | Comportamiento |
|----------|---------|----------------|
| Blinky   | Rojo    | Persecución directa a Pac-Man |
| Pinky    | Rosa    | Emboscada: apunta 4 celdas delante de Pac-Man según su dirección |
| Inky     | Cian    | Flanco: refleja posición de Pac-Man a través de Blinky (`tx = bx + 2*(px-bx)`) |
| Clyde    | Naranja | Tímido: si distancia Manhattan ≤8 → aleatorio; si no, persigue |

- **Activación escalonada:** cada fantasma se activa `i × 2000ms` tras el inicio (usa `performance.now()`). Antes de activarse queda visible pero congelado.
- **Salida de pen:** lógica explícita — navega a la columna más cercana de la puerta (13 o 14) y sube.
- **Frightened mode** (power pellet): fantasmas azules, dirección invertida al comerse el pellet, comestibles con puntuación 200/400/800/1600 en cadena. Al ser "comidos" → estado ojos (3s) → reaparecen en pen. Últimos 4s: parpadeo blanco/azul (warning).
- **Túnel:** fila 14, wrap-around horizontal cuando x sale de [0,27].

### Reglas de movimiento

- `isWall(grid,x,y,actor)`: pacman bloqueado por muros(1) y puertas(3); fantasmas solo por muros(1).
- `canMove` valida dirección + túnel.
- Colisión: |dx| < 0.5 AND |dy| < 0.5.

## Flujo spec-driven

- Este proyecto sigue una metodología de desarrollo spec-driven.
- Las especificaciones viven en `specs/` (numeradas `NN-slug.md`). Usar la skill `/spec` para crear, `/spec-impl` para implementar.
- Una especificación debe estar en estado `Approved` antes de comenzar la implementación.
- Nomenclatura de ramas: `spec-NN-slug`.

### Specs completadas / en curso

| Spec | Título | Estado |
|------|--------|--------|
| 01   | Four Ghost AI | Merged (PR #1) |
| 02   | Staggered Ghosts & Pen Exit | Merged (PR #2) |
| 03   | Power Pellets & Frightened Mode | En implementación (`spec-03-power-pellets`) |
