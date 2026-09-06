## Por qué (Why)

La Spec 03 (Power Pellets & Frightened Mode) pospuso la funcionalidad de "ojos viajando por el laberinto" como fuera de alcance. Actualmente, cuando Pac-Man se come un fantasma asustado este queda congelado en su posición durante 3 s y luego es teletransportado a su pen — visualmente brusco y menos envolvente que el comportamiento clásico donde los ojos navegan por los pasillos hasta llegar a casa.

## Qué cambia (What Changes)

- Los "ojos" del fantasma (estado comido) ahora **navegan por los corredores del laberinto** hacia su celda de origen, en vez de quedarse congelados en el sitio.
- Los ojos se mueven al **doble de la velocidad normal del fantasma** (0.2 celdas/frame vs 0.1).
- La navegación reutiliza el pathfinding greedy por distancia Manhattan existente, apuntando a `GHOST_STARTS[i]`.
- Dentro de la zona de pen (y∈[12..15], x∈[11..16]), los ojos navegan directamente hacia su celda exacta de inicio en lugar de usar la lógica de "salida".
- **Detección de llegada:** cuando un ojo alcanza alineado su celda objetivo, reaparece normalmente (`eaten=false`, `dir='up'`).
- **Timeout de seguridad (10 s):** si un ojo no ha llegado tras 10 s desde que fue comido, se teletransporta directamente a su celda de inicio como respaldo.
- El timer fijo de 3 segundos es reemplazado por el mecanismo de navegación + llegada; la exclusión de colisión para los ojos permanece sin cambios.

## Capacidades (Capabilities)

### Nuevas capacidades

- `ghost-eyes-return`: Los ojos navegan por los corredores del laberinto al doble de velocidad hasta su celda de origen, con un timeout de seguridad de 10 s como respaldo.

### Capacidades modificadas

<!-- Ninguna — los requisitos de la spec 03 no cambian; esto agrega la funcionalidad pospuesta como una capacidad propia. -->

## Impacto (Impact)

- **game.js:** `moveGhost()` (eliminar el early-return para comidos), `decideGhost()` (objetivo = celda de inicio cuando está comido, navegación directa en zona pen), nueva detección de llegada + lógica de reaparición, timeout fallback en `update()`.
- **render.js:** Sin cambios — ya dibuja los ojos vía `drawGhostEyes(ctx, g)` que usa `g.x`/`g.y`; simplemente se animarán a medida que el fantasma se mueva.
- **maze.js / main.js:** Sin cambios.
