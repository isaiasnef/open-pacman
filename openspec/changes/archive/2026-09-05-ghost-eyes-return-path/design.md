## Contexto (Context)

El sistema de movimiento de fantasmas existente (`moveGhost` + `decideGhost`) en game.js usa:
- Verificación de alineación (`aligned(v)`: dentro de 1e-3 del entero más cercano) antes de tomar decisiones de dirección.
- Pathfinding greedy por distancia Manhattan hacia una celda objetivo.
- Incremento por frame de `g.speed` (0.1 para fantasmas, se alinea cada 10 frames desde un inicio en entero).

Actualmente la línea 200 de `moveGhost()` tiene: `if (!g.active || g.eaten) return;` — esto es lo que congela los ojos en su sitio. La lógica de reaparición (líneas 261-268) teletransporta tras un timer fijo.

## Objetivos / No objetivos (Goals / Non-Goals)

**Objetivos:**
- Que los ojos naveguen por corredores hasta su celda de origen usando la infraestructura de pathfinding existente.
- Velocidad 2x (0.2 celdas/frame) para un retorno ágil y perceptible.
- Integración limpia con el ciclo alineación/decisión sin romper precisión de punto flotante.
- Timeout de seguridad como último recurso.

**No objetivos:**
- Pathfinding óptimo A* o BFS — greedy Manhattan es suficiente y consistente con el estilo de IA existente.
- Renderizado visual distinto durante la navegación (los ojos ya se dibujan correctamente vía `drawGhostEyes`).
- Efectos de sonido, estelas de partículas u otro pulido.

## Decisiones (Decisions)

### 1. Velocidad: multiplicar en el momento del movimiento, no almacenar en `g.speed`

**Elección:** En `moveGhost()`, cuando `g.eaten === true`, usar `const spd = g.speed * 2;` para el incremento por frame en vez de modificar `g.speed`.

**Por qué:** Evita restaurar estado al llegar. La velocidad canónica del fantasma (0.1) permanece intacta en `g.speed`; simplemente nos movemos más rápido mientras navegamos de vuelta. Sin riesgo de olvidar restaurarla.

**Alternativa considerada:** Establecer `g.speed = 0.2` al comer y restaurar a 0.1 al llegar. Rechazada — más estado que gestionar, fácil de olvidar casos límite (ruta del timeout, resetPositions).

### 2. Ajuste de posición (snap) en la transición al estado comido

**Elección:** Cuando un fantasma es comido (manejador de colisión), redondear inmediatamente `g.x = Math.round(g.x)` y `g.y = Math.round(g.y)`.

**Por qué:** El ciclo de alineación requiere que el fantasma esté en coordenada entera antes de tomar decisiones. A velocidad 0.2, partir desde x=3.7 produciría: 3.9, 4.1 — saltándose 4.0 y nunca disparando `aligned()`. El snap garantiza alineación limpia cada 5 frames (entero + N×0.2 siempre cae en enteros).

**Impacto:** Salto visual máximo de 0.5 celdas (~10px) en el momento del comer. Imperceptible porque simultáneamente el cuerpo desaparece y pasa a modo "ojos".

### 3. Reutilizar `decideGhost()` con un override de objetivo

**Elección:** En `decideGhost()`, añadir una rama inicial: si `g.eaten === true` → establecer `(tx, ty) = GHOST_STARTS[ghostIndex]` y saltar toda la lógica de IA por tipo (apuntado Blinky/Pinky/Inky/Clyde). El bucle greedy Manhattan existente elige entonces la mejor dirección hacia ese objetivo fijo.

**Por qué:** Cambio mínimo — un `if` al inicio de la función, reutilizando 20+ líneas de pathfinding que ya funciona. No se necesita ningún algoritmo nuevo.

**Manejo de zona pen:** La lógica de salida del pen existente (líneas ~149-157) empuja a los fantasmas "arriba" para salir. Para ojos entrando al pen, necesitamos que vayan A su celda exacta en vez. Solución: cuando `g.eaten`, saltarse el bloque genérico de salida del pen y dejar que la selección greedy por objetivo lo maneje — dentro de la zona pen todas las celdas están abiertas (sin muros), así que la distancia Manhattan guiará directamente a la celda de inicio.

**Alternativa considerada:** Escribir una función separada `decideEyes()`. Rechazada — duplica el bucle de selección de dirección sin beneficio; la única diferencia son las coordenadas objetivo y saltar la lógica de personalidad, ambas manejables con una rama condicional.

### 4. Detección de llegada: alineado + en objetivo

**Elección:** En `moveGhost()`, tras la verificación de alineación y antes/después de decidir dirección, añadir: si `g.eaten && g.x === targetX && g.y === targetY` → reaparecer (establecer `eaten=false`, `dir='up'`).

**Por qué:** El fantasma está garantizado en coordenadas enteras cuando está alineado. Comparación de igualdad simple contra la posición de inicio conocida. No se necesita epsilon porque ambos son enteros exactos tras el redondeo.

### 5. Timeout: almacenar timestamp, verificar en bucle update

**Elección:** Reemplazar `g.reappearAt = now + 3000` con `g.eatenAt = performance.now()`. En `update()`, para cada fantasma comido: si `now - g.eatenAt > 10000 && aún no en objetivo` → teletransportar al inicio, limpiar estado comido.

**Por qué:** Reutiliza el patrón de verificación por frame existente en `update()` (líneas ~260-268). El cambio de nombre del campo (`reappearAt` → `eatenAt`) hace la intención más clara: es "cuándo fue comido este fantasma" no "cuándo reaparece".

### 6. Sin cambios en render.js, maze.js ni main.js

**Por qué:** `drawGhostEyes(ctx, g)` ya lee `g.x`/`g.y` y dibuja en esa posición — a medida que el ojo se mueve frame a frame, se anima naturalmente. La exclusión de colisión (`if (!collides(...) || g.eaten) continue;`) ya previene re-colisiones. No se necesita nueva superficie de datos ni API en otros archivos.

## Riesgos / Compromisos (Risks / Trade-offs)

- **[Pathfinding greedy puede dar vueltas en corredores complejos]** → Mitigado por el timeout de 10 s como respaldo. En la práctica, el laberinto es pequeño (28×31) y greedy Manhattan rara vez da vueltas; en el peor caso toma una ruta más larga pero aún llega bien dentro de los 10 s a velocidad 2x (~5 celdas/frame × 60fps = capacidad de recorrido de ~300 celdas/s).

- **[El snap al comer causa un pequeño salto visual]** → Desplazamiento máximo de 0.5 celda (10px), simultáneo con la transición cuerpo→ojos. Compromiso aceptable a cambio de matemática de alineación limpia.

- **[Los ojos pasan por la fila del túnel (fila 14)]** → `wrapTunnel()` ya se llama en `moveGhost()`. Si el camino de un ojo cruza la fila del túnel, envolverá correctamente — igual que los fantasmas normales. No requiere manejo especial.
