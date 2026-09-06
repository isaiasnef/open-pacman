## 1. Cambios en el modelo de datos (game.js)

- [x] 1.1 En `createGame()`, renombrar el campo del fantasma `reappearAt` a `eatenAt` (valor inicial `null`) — almacena el timestamp cuando fue comido, usado para el timeout de 10 s
- [x] 1.2 Actualizar todas las referencias de `g.reappearAt` a `g.eatenAt` en todo game.js

## 2. Movimiento: ojos navegan en vez de congelarse (game.js)

- [x] 2.1 En `moveGhost()`, eliminar el early-return para `g.eaten`; reemplazar con una rama que usa multiplicador de velocidad 2x (`const spd = g.speed * 2`) cuando está comido
- [x] 2.2 Tras la verificación de alineación en `moveGhost()`, añadir detección de llegada: si `g.eaten` y la posición redondeada es igual a `GHOST_STARTS[i]` → establecer `eaten=false`, `dir='up'`, return (reaparición completa)

## 3. Override del objetivo de navegación (game.js — decideGhost)

- [x] 3.1 Al inicio de `decideGhost()`, añadir: si `g.eaten === true` → establecer `(tx, ty)` a las coordenadas `GHOST_STARTS[i]` de ese fantasma y saltar toda la lógica de IA por tipo (apuntado Blinky/Pinky/Inky/Clyde)
- [x] 3.2 Cuando `g.eaten`, omitir el bloque de salida del pen (~líneas 149-157) para que greedy Manhattan guíe directamente a la celda exacta de inicio dentro del área abierta del pen

## 4. Manejador de comer: snap de posición + timestamp (game.js — bucle de colisión en update)

- [x] 4.1 En el manejador de colisión donde se establece `g.eaten = true`, añadir snap inmediato de posición: `g.x = Math.round(g.x); g.y = Math.round(g.y)`
- [x] 4.2 Establecer `g.eatenAt = performance.now()` en vez de `g.reappearAt = now + 3000`

## 5. Timeout fallback (game.js — bucle update)

- [x] 5.1 Reemplazar el bloque de reaparición existente (~líneas 260-268): para cada fantasma con `eaten === true`, verificar si `now - g.eatenAt > 10000` Y aún no está en objetivo → teletransportar a `GHOST_STARTS[i]`, establecer `dir='up'`, limpiar estado comido

## 6. Verificación (manual)

- [x] 6.1 Abrir `src/index.html` en navegador, comer un power pellet, perseguir y comer los 4 fantasmas — verificar que los ojos navegan por corredores a velocidad visiblemente mayor hasta el pen
- [x] 6.2 Verificar que los ojos no atraviesan muros; siguen rutas válidas de corredor
- [x] 6.3 Verificar que al llegar, el fantasma reaparece normalmente (cuerpo completo visible, reanuda IA)
- [x] 6.4 Verificar que Pac-Man puede superponerse a ojos navegando sin perder vida ni volver a comerlos
- [x] 6.5 Confirmar ausencia de errores en consola durante una sesión completa de juego
