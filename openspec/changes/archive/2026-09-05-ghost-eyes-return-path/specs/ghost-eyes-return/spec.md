## Purpose

Define el comportamiento de los "ojos" del fantasma tras ser comido por Pac-Man: navegan por los corredores del laberinto a velocidad acelerada hasta su celda de origen, reemplazando el mecanismo anterior de congelar y teletransportar.

## ADDED Requirements

### Requirement: Eyes navigate through corridors toward origin cell
Cuando un fantasma asustado es comido por Pac-Man, sus ojos DEBEN (SHALL) moverse a través de corredores válidos del laberinto (respetando muros) hacia la posición de inicio designada de ese fantasma (`GHOST_STARTS[i]`), en lugar de permanecer estacionarios en el punto de colisión.

#### Scenario: Eye moves along corridor after being eaten
- **WHEN** Pac-Man se come un fantasma asustado y los ojos aún no están alineados con su celda objetivo
- **THEN** los ojos DEBEN (SHALL) moverse un paso por frame en una dirección que reduzca la distancia Manhattan a `GHOST_STARTS[i]`, eligiendo entre direcciones válidas (sin muro)

#### Scenario: Eye cannot pass through walls
- **WHEN** la dirección elegida por los ojos lleva hacia una celda de muro
- **THEN** los ojos DEBEN (SHALL) seleccionar una alternativa válida entre las opciones disponibles no-reversas, o invertir si están en un callejón sin salida

### Requirement: Eyes move at 2x normal ghost speed
Los ojos que navegan de vuelta a su pen DEBEN (SHALL) moverse al doble de la velocidad estándar del fantasma (0.2 celdas por frame en vez de 0.1).

#### Scenario: Eye traversal is faster than normal ghost movement
- **WHEN** un ojo recorre un segmento de corredor de N tiles
- **THEN** DEBE (SHALL) tomar aproximadamente la mitad de los frames que necesitaría un fantasma a velocidad normal para la misma distancia

### Requirement: Eyes navigate directly to exact start cell inside pen zone
Cuando un ojo entra en la región del pen (y ∈ [12..15], x ∈ [11..16]), DEBE (SHALL) apuntar a sus coordenadas específicas de `GHOST_STARTS[i]` en lugar de usar la lógica genérica de "salida".

#### Scenario: Eye in pen zone heads to exact cell
- **WHEN** un ojo está dentro de la región del pen y aún no ha llegado a su posición de inicio
- **THEN** DEBE (SHALL) elegir direcciones que minimicen la distancia a sus coordenadas específicas `GHOST_STARTS[i]` (no simplemente "ir arriba")

### Requirement: Eyes respawn upon reaching origin cell
Cuando un ojo se alinea con su celda objetivo (`GHOST_STARTS[i].x`, `GHOST_STARTS[i].y`), DEBE (SHALL) transicionar inmediatamente de vuelta al estado normal del fantasma.

#### Scenario: Eye arrives at start position and respawns
- **WHEN** las coordenadas redondeadas de un ojo son iguales a sus coordenadas en `GHOST_STARTS[i]`
- **THEN** los ojos DEBEN (SHALL) establecer `eaten = false`, reiniciar dirección a `'up'`, restaurar velocidad a 0.1, y reanudar el comportamiento normal de IA en frames subsiguientes

### Requirement: Safety timeout teleports stuck eyes after 10 seconds
Si un ojo no ha alcanzado su celda objetivo dentro de los 10 segundos posteriores a ser comido, DEBE (SHALL) ser teletransportado directamente a `GHOST_STARTS[i]` como respaldo.

#### Scenario: Eye fails to arrive within timeout
- **WHEN** han transcurrido 10 segundos desde que el fantasma fue comido y los ojos aún no están alineados con su celda objetivo
- **THEN** el sistema DEBE (SHALL) establecer la posición de los ojos directamente en `GHOST_STARTS[i]`, limpiar el estado comido, y reiniciar dirección a `'up'`

### Requirement: Eyes do not collide with Pac-Man during navigation
Mientras estén en modo ojos (navegando de vuelta), un fantasma NO DEBE (SHALL NOT) provocar daño por colisión ni ser re-comido por Pac-Man.

#### Scenario: Pac-Man passes through navigating eye without effect
- **WHEN** la posición de Pac-Man se superpone con unos ojos que aún están navegando hacia su pen
- **THEN** no DEBE (SHALL) ocurrir pérdida de vida, cambio de puntuación, ni transición de estado para ninguna entidad
