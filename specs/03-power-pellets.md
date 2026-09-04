# SPEC 03 — Power Pellets y modo asustado

> **Status:** Approved
> **Depends on:** SPEC 01 (cuatro fantasmas con IA), SPEC 02 (fantasmas escalonados)
> **Date:** 2026-09-03
> **Objective:** Añadir 4 power pellets que activan un modo asustado de 12 s donde los fantasmas se vuelven azules, invierten dirección y pueden ser comidos por PacMan con puntos crecientes (200/400/800/1600).

## Scope

**In:**

- 4 power pellets en posiciones fijas: `(3,1)`, `(24,1)`, `(3,29)`, `(24,29)`.
- Modo asustado global de 12 s: fantasmas azules, dirección invertida al activarse.
- PacMan puede comer fantasmas azules; puntos por fantasma comido: 200/400/800/1600 según orden en la misma ronda.
- Fantasma comido → estado "ojos" fijo durante 3 s, luego reaparece en su pen de origen.
- Si el timer sigue activo al reaparecer, vuelve asustado; si expiró, vuelve normal.
- Comer un segundo power pellet reinicia el timer a 12 s y resetea el contador (vuelve a empezar en 200).
- Últimos 4 s: fantasmas parpadean blanco/azul alternando cada ~500 ms como advertencia.
- Power pellets cuentan hacia `dotsRemaining` para completar nivel.

**Out of scope:**

- Niveles adicionales o duración variable por nivel.
- Sonidos al comer power pellet / fantasma.
- Animación de "ojos" viajando a través del laberinto (quedan fijos 3 s y reaparecen en pen).
- Velocidad distinta para fantasmas asustados o comiendo.

## Data model

```js
// maze.js — nuevo valor: 4 = power pellet ('@' en MAZE_STR)
// Posiciones: (3,1), (24,1), (3,29), (24,29)

// game.js — campos nuevos en el objeto game:
game.frightUntil    // timestamp absoluto o null si no activo
game.ghostsEaten    // contador de fantasmas comidos en la ronda actual (0–3)

// game.js — campos nuevos por fantasma:
g.eaten       // boolean, true mientras está en modo "ojos"
g.reappearAt  // timestamp cuando reaparece (eat_time + 3000), null si no aplica
```

Convenciones:

- `frightUntil` es global: todos comparten el mismo estado. Un fantasma está asustado si `now < game.frightUntil && !g.eaten`.
- El contador se resetea a 0 al comer power pellet o cuando expira el timer.
- Puntos por índice: `[200, 400, 800, 1600][game.ghostsEaten]`, luego incrementa.
- Advertencia (parpadeo): activa durante los últimos 4 s (`frightUntil - now < 4000`).

## Implementation plan

1. **maze.js:** cambiar celdas `(3,1)`, `(24,1)`, `(3,29)`, `(24,29)` de `'.'` a `'@'`; añadir `parseTile('@') → 4`. Verificar: ver 4 puntos grandes en esas posiciones.

2. **render.js — power pellets:** nueva función `drawPowerPellets(ctx, grid, frame)`: círculo mayor (radio ~5) con parpadeo basado en `frame`. Llamarla desde `draw()` tras `drawDots`. Verificar: ver 4 puntos grandes parpadeantes.

3. **game.js — comer power pellet:** en `movePacman()`, si celda tiene valor 4 → comerlo (`score += 50`), activar modo asustado, resetear contador, invertir dirección de todos los fantasmas activos no comidos. Verificar: al comer un power pellet los fantasmas cambian a azul y giran.

4. **game.js — colisión modificada:** en el bucle de colisiones dentro de `update()`: si fantasma está asustado → comerlo (puntos, eaten=true). Si NO está asustado o ya comido → comportamiento actual. Verificar: tocar fantasma azul suma 200 pts y desaparece.

5. **game.js — reaparición:** en `update()`, para cada fantasma con `g.eaten === true && now >= g.reappearAt` → resetear posición a GHOST_STARTS[i], dir='up', eaten=false, reappearAt=null. Verificar: tras 3 s el fantasma vuelve a la pen.

6. **game.js — expiración del timer:** en `update()`, si `frightUntil !== null && now >= frightUntil` → setear `frightUntil = null`, resetear contador. Verificar: tras 12 s los fantasmas vuelven a colores normales.

7. **render.js — fantasma asustado:** modificar `drawGhost()` para aceptar estado visual: si asustado → cuerpo azul claro, ojos blancos sin pupilas; si en advertencia (últimos 4s) y frame par/impar → alternar blanco/azul. Verificar: ver fantasmas azules con cara de miedo.

8. **render.js — modo "ojos":** nueva función `drawGhostEyes(ctx, g)` que dibuja solo los dos ojos en la posición del fantasma comido. En el bucle principal, si `g.eaten` → llamar a esta en vez de `drawGhost`. Verificar: ver un par de ojos fijos donde fue comido.

9. **Verificación final:** abrir `src/index.html`, jugar; comer power pellet, perseguir y comer los 4 fantasmas (200+400+800+1600=3000 pts), verificar reaparición en pen, parpadeo de advertencia al final del timer, y que tras expiración todo vuelve a normal.

## Acceptance criteria

- [ ] El juego carga sin errores en la consola.
- [ ] Se ven 4 power pellets (círculos grandes) en las posiciones `(3,1)`, `(24,1)`, `(3,29)`, `(24,29)`.
- [ ] Los power pellets parpadean (visibles/invisibles alternando con el frame).
- [ ] Al comer un power pellet: los 4 fantasmas activos se vuelven azules e invierten su dirección.
- [ ] Tocar un fantasma azul NO resta vida; suma puntos según contador (200/400/800/1600).
- [ ] El fantasma comido muestra solo ojos en la posición donde fue tocado durante 3 s, luego reaparece en su pen.
- [ ] Si el timer sigue activo al reaparecer, el fantasma vuelve asustado; si expiró, vuelve normal.
- [ ] Comer un segundo power pellet reinicia el timer a 12 s y resetea el contador de puntos a 0.
- [ ] Durante los últimos 4 s del efecto, los fantasmas parpadean alternando blanco/azul cada ~500 ms.
- [ ] Al expirar el timer (12 s), todos los fantasmas vuelven a su color normal y comportamiento de IA original.
- [ ] Los power pellets cuentan hacia `dotsRemaining` (completarlos es necesario para ganar).

## Decisions

- **Sí:** Timer global (`frightUntil`) en vez de flag por fantasma — más simple, todos comparten el mismo estado.
- **No:** Ojos viajando a través del laberinto hacia la pen — requiere pathfinding; 3 s fijos + reaparición es suficiente.
- **Sí:** Reiniciar timer Y contador al comer segundo power pellet (comportamiento clásico).
- **No:** Velocidad distinta para fantasmas asustados o en modo ojos — simplifica la mecánica.
- **Sí:** Power pellets cuentan hacia `dotsRemaining` — coherente con "completar el nivel = comer todo".
- **No:** Parpadeo infinito del power pellet (siempre visible) — el parpadeo clásico ayuda a distinguirlo de los puntos normales.

## What is not in this spec

Niveles adicionales o duración variable por nivel, sonidos, animación de ojos viajando por el laberinto, velocidad distinta para fantasmas asustados/comiendo, power pellets en posiciones distintas a las 4 definidas. Cada uno va en su propia spec si llega.
