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
- **Valores de celda:** 0=vacío, 1=muro, 2=punto, 3=puerta de la pen de fantasmas.

## Flujo spec-driven

- Este proyecto sigue una metodología de desarrollo spec-driven.
- Las especificaciones viven en `specs/` (numeradas `NN-slug.md`). Usar la skill `/spec` para crear, `/spec-impl` para implementar.
- Una especificación debe estar en estado `Approved` antes de comenzar la implementación.
- Nomenclatura de ramas: `spec-NN-slug`.
