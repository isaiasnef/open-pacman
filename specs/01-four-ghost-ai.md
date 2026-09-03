# SPEC 01 — Cuatro fantasmas con IA distinta

> **Status:** Approved
> **Depends on:** —
> **Date:** 2026-09-02
> **Objective:** Añadir 4 fantasmas con comportamientos de IA distintos (Blinky persigue, Pinky embosca, Inky flanquea, Clyde es tímido) a velocidad uniforme.

## Scope

**In:**

- 4 fantasmas en la pen con los colores ya definidos en `GHOST_COLORS`.
- 4 estrategias de selección de objetivo en `decideGhost`.
- Blinky (rojo) persigue directamente a PacMan.
- Pinky (rosa) apunta 4 casillas delante de PacMan según su dirección.
- Inky (cian) usa la posición de Blinky para flanquear.
- Clyde (naranja) persigue si la distancia Manhattan > 8 y elige dirección aleatoria si ≤ 8.
- Todos a velocidad 0.1 celda/frame.
- Tocar cualquier fantasma resta una vida.

**Out of scope (for future specs):**

- Power pellets / comer fantasmas.
- Velocidades distintas por fantasma.
- Estados de miedo (frightened) o puntos bonus por comer.
- Niveles adicionales o progresión de dificultad.

## Data model

```js
// maze.js — GHOST_STARTS pasa de 2 a 4 entradas
const GHOST_STARTS = [
  { x: 13, y: 14, kind: 'blinky' },
  { x: 14, y: 14, kind: 'pinky' },
  { x: 13, y: 13, kind: 'inky' },
  { x: 14, y: 13, kind: 'clyde' },
];
```

Convenciones:

- `kind` es una string: `'blinky'`, `'pinky'`, `'inky'`, `'clyde'`.
- Blinky es siempre `game.ghosts[0]` (primer elemento del array). Inky lo necesita para su cálculo.
- La distancia se calcula en Manhattan sobre coordenadas enteras redondeadas.
- `render.js` y `main.js` no cambian: `GHOST_COLORS` ya tiene 4 colores y el bucle itera sobre `game.ghosts`.

## Implementation plan

1. Actualizar `GHOST_STARTS` en `src/js/maze.js` a 4 entradas con kinds `blinky`, `pinky`, `inky`, `clyde`. Verificar: abrir en navegador, ver 4 fantasmas en la pen.
2. Reescribir `decideGhost` en `src/js/game.js` con las 4 estrategias de objetivo. Verificar: cada fantasma se mueve de forma distinta.
3. Verificar en navegador: Blinky persigue, Pinky embosca, Inky flanquea, Clyde se dispersa cerca.

## Acceptance criteria

- [ ] El juego carga sin errores en la consola.
- [ ] Se ven 4 fantasmas de colores distintos (rojo, rosa, cian, naranja) en la pen al inicio.
- [ ] Blinky (rojo) se mueve siempre hacia la celda actual de PacMan.
- [ ] Pinky (rosa) apunta a la celda 4 pasos delante de PacMan según su dirección.
- [ ] Inky (cian) calcula su objetivo usando la posición de Blinky.
- [ ] Clyde (naranja) persigue a PacMan cuando la distancia Manhattan > 8 y elige dirección aleatoria cuando ≤ 8.
- [ ] Los 4 fantasmas se mueven a 0.1 celda/frame.
- [ ] Tocar cualquier fantasma resta 1 vida y reinicia posiciones.
- [ ] Al perder la última vida, el estado pasa a `'lost'`.

## Decisions

- **Sí:** Modelo clásico de 4 fantasmas (Blinky/Pinky/Inky/Clyde). Es el comportamiento esperado por cualquier jugador de PacMan.
- **No:** Velocidades distintas. Simplifica la IA y mantiene la dificultad equilibrada.
- **Sí:** Tocar = perder vida. Sin power pellets ni estados de miedo.
- **No:** Power pellets / comer fantasmas. Especie separada si se quiere.
- **Sí:** Blinky como `game.ghosts[0]`. Inky necesita su posición; el índice 0 es estable.
- **No:** Posiciones de pen distintas a las actuales. Las 4 caben en el interior de la pen (rows 13-14, cols 13-14).

## What is **not** in this spec

- Power pellets y comer fantasmas.
- Estados de miedo (frightened) o puntos bonus.
- Velocidades distintas por fantasma.
- Niveles adicionales o progresión.
- Sonidos o efectos visuales adicionales.

Cada uno de esos, si llega, va en su propia spec.
