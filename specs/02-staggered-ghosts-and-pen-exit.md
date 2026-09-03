# SPEC 02 — Fantasmas escalonados y salida de pen

> **Status:** Aprobado
> **Depends on:** SPEC 01 (cuatro fantasmas con IA)
> **Date:** 2026-09-03
> **Objective:** Hacer que los 4 fantasmas arranquen uno al lado del otro, se activen escalonados cada 2 s y salgan de la pen por sí mismos desde el inicio.

## Scope

**In:**

- Reubicar `GHOST_STARTS` en fila 14, cols 12–15 (uno al lado del otro).
- Activación escalonada: fantasma *i* se activa a los `i·2 s` tras el arranque; antes queda **visible pero congelado**.
- Medición por **tiempo real** (`performance.now()`), no por frames.
- El escalonado se **repite cada vez que se reubican** (inicio y tras perder vida).
- Lógica explícita de salida: dentro de la pen el fantasma navega a la columna de puerta más cercana (13 o 14) y sube hasta salir; luego usa su IA normal.

**Out of scope:**

- Power pellets / comer fantasmas, estados de miedo, velocidades distintas por fantasma, niveles/progresión. (Ya excluidos en SPEC 01.)

## Data model

```js
// maze.js — GHOST_STARTS pasa a fila 14, cols 12–15 (orden y kinds intactos)
const GHOST_STARTS = [
  { x: 12, y: 14, kind: 'blinky' },   // ghosts[0] -> Inky lo usa
  { x: 13, y: 14, kind: 'pinky'  },
  { x: 14, y: 14, kind: 'inky'   },
  { x: 15, y: 14, kind: 'clyde'  },
];

// game.js — cada fantasma gana dos campos de activación
ghost = { x, y, dir:'up', speed, kind, active:false, activateAt:<ms> }
```

Convenciones:

- `activateAt` es un timestamp absoluto (`performance.now()` + offset). Offset del fantasma *i* = `i·2000 ms`.
- `active === false` → el fantasma no se mueve (congelado, pero sí dibujado).
- Blinky sigue siendo `game.ghosts[0]`; Inky lo necesita para su cálculo.

## Implementation plan

1. **maze.js:** actualizar `GHOST_STARTS` a fila 14 cols 12–15 manteniendo kinds y orden. Verificar: ver los 4 fantasmas en línea sobre la pen al cargar.
2. **game.js — activación:** en `createGame()` fijar `T0 = performance.now()`, dar a cada fantasma `active:false` y `activateAt:T0 + i·2000`. En `update()`, antes de mover, pasar a `active:true` los fantasmas cuyo `now >= activateAt`; en `moveGhost()` devolver sin mover si `!g.active`. Verificar: al pulsar Start el fantasma 1 se mueve ya y cada uno siguiente aparece ~2 s después.
3. **game.js — salida de pen:** en `decideGhost()`, añadir rama explícita: si el fantasma está dentro de la región de pen (y ∈ [13,15], x ∈ [11,16]) → dirigirse a la columna de puerta más cercana (13 o 14) y luego subir hasta salir; al salir, reanudar su IA normal. Verificar: ningún fantasma queda oscilando dentro de la pen.
4. **game.js — reinicio:** en `resetPositions()` volver a fijar `T0 = performance.now()`, `active:false` y offsets escalonados para cada fantasma (repite el escalonado). Verificar: tras perder una vida, los 4 vuelven a fila 14 cols 12–15 y se activan de nuevo en cascada.
5. **Verificación final:** abrir `src/index.html`, jugar; confirmar consola sin errores, entrada escalonada y salida limpia de la pen tanto al inicio como tras reinicio.

## Acceptance criteria

- [ ] El juego carga sin errores en la consola.
- [ ] Al iniciar, los 4 fantasmas están uno al lado del otro (fila 14, cols 12–15).
- [ ] Antes de su turno cada fantasma es visible pero no se mueve.
- [ ] El fantasma 1 se activa al arrancar; el *i*-ésimo a los `i·2 s` (medido en tiempo real).
- [ ] Ningún fantasma queda atascado dentro de la pen: todos salen por arriba, a través de la puerta.
- [ ] Tras salir de la pen cada fantasma usa su IA normal (Blinky/Pinky/Inky/Clyde).
- [ ] Al perder una vida los fantasmas vuelven a fila 14 cols 12–15 y se reactivan en cascada escalonada.

## Decisions

- **Sí:** Activación visible pero congelada — el jugador ve la "fila" de fantasmas esperando su turno (más fiel al PacMan clásico).
- **No:** Invisible hasta activarse — oculta información que ayuda a anticipar la dificultad.
- **Sí:** Tiempo real (`performance.now()`) en vez de contar frames — estable aunque cambie el framerate del navegador.
- **Sí:** Repetir el escalonado tras perder vida ("siempre al reubicarse") — coherente con "desde el inicio" y evita que un reinicio los suelte todos a la vez.
- **No:** Escalonado solo en el primer arranque — rompería la expectativa de entrada ordenada en cada ronda.
- **Sí:** Lógica explícita de salida (navegar a col 13/14 y subir) — determinista; el greedy puro nunca elige "subir" porque PacMan está abajo, que es justo el bug observado.
- **No:** Solo ajustar desempate del greedy — frágil: sigue dependiendo de la posición relativa de PacMan.

## What is not in this spec

Power pellets / comer fantasmas, estados de miedo o puntos bonus, velocidades distintas por fantasma, niveles/progresión, sonidos/efectos visuales. Cada uno va en su propia spec si llega.
