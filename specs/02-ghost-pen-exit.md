# SPEC 02 — Salida de los fantasmas desde el pen

> **Status:** Implemented
> **Depends on:** SPEC 01
> **Date:** 2026-09-20
> **Objective:** Hacer que los 4 fantasmas salgan del pen al inicio de la partida en lugar de quedar atrapados dentro de la jaula.

## Scope

**In:**

- Diana de salida (waypoint) para los fantasmas mientras están dentro del pen.
- Detección de "dentro del pen" por coordenadas del área (cols 11–16, filas 13–15), sin estado nuevo.
- Constantes del área del pen declaradas en `maze.js`, junto al resto de la geometría.

**Out of scope (para futuros specs):**

- Liberación escalonada (temporizador por fantasma).
- Prevención de reentrada al pen desde fuera.
- Regreso de fantasmas vulnerables ("ojos") al pen, que depende del modo asustado.
- Cambios al laberinto, a la puerta (value 3) o al orden de `<script>`.

## Data model

Se añaden dos constantes de geometría en `maze.js`. La puerta es el valor 3 de la fila 12, cols 13–14:

```js
// maze.js
// Área interior del pen: filas 13-15, columnas 11-16 (celdas 0 transitables).
const PEN_BOUNDS = { xMin: 11, xMax: 16, yMin: 13, yMax: 15 };
// Celda encima de la puerta (fila 12), a la que se canalizan los fantasmas encerrados.
const PEN_EXIT_TARGET = { x: 13, y: 11 };
```

No hay estado nuevo en el objeto `ghost`: la salida se decide solo por posición.

## Implementation plan

1. `maze.js`: declarar `PEN_BOUNDS` y `PEN_EXIT_TARGET` junto a `GHOST_STARTS` y exportarlas (`window.PEN_BOUNDS`, `window.PEN_EXIT_TARGET`). Verificación: cargar `index.html` sin errores de consola.
2. `game.js`: helper `inPen( g )` basado en `PEN_BOUNDS` y, al inicio de `targetFor`, devolver `PEN_EXIT_TARGET` si el fantasma está dentro del pen. Verificación: jugar y ver los 4 fantasmas salir por la puerta y moverse por el mapa.
3. Regresión manual: perder una vida (`resetPositions` devuelve los 4 al pen) y confirmar que vuelven a salir; comer dots y comprobar HUD y overlays GANASTE/PERDISTE.

## Acceptance criteria

- [ ] Al iniciar la partida, los 4 fantasmas salen del pen cruzando la puerta (fila 12, cols 13–14) y se mueven por el mapa; ninguno queda atrapado.
- [ ] Con `?debug=true`, mientras un fantasma está dentro del pen su diana dibujada es `{ x: 13, y: 11 }`.
- [ ] Fuera del pen, `targetFor` conserva las dianas de SPEC 01 (`chaser`/`ambusher`/`flanker`/`shy`).
- [ ] Tras perder una vida, los fantasmas vuelven a salir del pen.
- [ ] Sin `?debug=true` no se dibuja nada extra.
- [ ] No hay errores en consola al cargar, jugar y reiniciar.
- [ ] Comer todos los dots muestra GANASTE; perder 3 vidas muestra PERDISTE.

## Decisions

- **Sí:** detección del pen por coordenadas del área, sin flag `inPen`. Autosana reentradas y cubre el reset de vidas sin estado que pueda desincronizarse.
- **No:** flag `inPen` por fantasma. Añade estado y una fuente de desincronización que la versión por coordenadas no tiene.
- **Sí:** waypoint `{13, 11}`, la celda justo encima de la puerta. El fantasma la cruza decisivamente y pasa a diana normal al salir del área.
- **Sí:** constantes del pen en `maze.js` (junto a `TUNNEL_ROW`/`PACMAN_START`/`GHOST_STARTS`) y lógica en `targetFor`; `render.js` la refleja gratis en el debug.
- **No:** liberación escalonada. SPEC 01 ya la dejó fuera; va en su propio spec.
- **No:** evitar reentradas (puerta unidireccional). No hace falta hoy; la autosanación la resuelve.
- **No:** tocar el desempate de `decideGhost`. Con la diana de salida, `up` gana estricto; no hace falta cambiar el orden `left,right,up,down`.

## Risks

| Riesgo | Mitigación |
| --- | --- |
| El `shy` huyendo (≤8 celdas) se orienta hacia el pen | Si entra, la detección por coordenadas lo vuelve a canalizar a `{13, 11}` y sale de nuevo. |
| Solapamiento visual de fantasmas al converger en el waypoint común | Aceptable: no hay colisión cuerpo a cuerpo entre fantasmas (solo pacman vs fantasma). |
| `PEN_BOUNDS`/`PEN_EXIT_TARGET` se desalinean si se edita el laberinto | Se declaran en `maze.js` con comentario junto al gate, donde se mantiene la simetría del `MAZE`. |

## What is **not** in this spec

- Liberación escalonada del pen.
- Prevención de reentrada / puerta unidireccional.
- Regreso de fantasmas vulnerables ("ojos") al pen.
- Cambios al laberinto (`MAZE`, puerta) o al orden de `<script>`.

Cada una de esas, si llega, va en su propio spec.