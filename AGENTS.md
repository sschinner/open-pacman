# AGENTS.md

Pac-Man en Vanilla JS (HTML + CSS + JS plano), sin build ni tooling. Proyecto de aprendizaje de Spec Driven Development.

## Ejecutar / verificar
- No hay build, tests ni linter. Abrir `src/index.html` en el navegador.
- Verificación manual: jugar, comer dots y comprobar HUD (SCORE/VIDAS) y overlays de ganar/perder.

## Arquitectura
- No hay ES modules. Cada archivo expone sus funciones como globals vía `window.X = ...`.
- El orden de los `<script>` en `src/index.html` es obligatorio y refleja las dependencias:
  `maze.js` → `game.js` → `render.js` → `main.js`. Nuevos archivos JS deben enlazarse ahí respetando el orden.
- Dependencias entre globals: `game.js` usa los de `maze.js` (`MAZE`, `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`); `render.js` usa `DIRS` (global de `game.js`); `main.js` consume `createGame`/`update`/`draw`.
- `MAZE` es pristino: `createGame()` lo copia a `game.grid`. Nunca mutar `MAZE`, solo `game.grid`.

## Laberinto (`maze.js`)
- 31 strings de 28 chars, parseados a números: `#`=1 pared, `.`=2 dot, `-`=3 puerta, espacio=0 transitable.
- Grid 28x31 simétrico respecto al eje vertical (entre cols 13 y 14); la fila del túnel (14) tiene los extremos abiertos. Al editar el laberinto se debe mantener esa simetría y el túnel.

## Convenciones
- Idioma del repo: español. Comentarios, README y textos de UI en español (overlays "GANASTE"/"PERDISTE").
- Estilo: espacios dentro de paréntesis y corchetes (`if ( x )`, `grid[ 0 ]`), comillas simples, `;` al final.
- Velocidades fraccionarias para alinear con la rejilla: `PACMAN_SPEED` 0.125 (=1/8, se alinea cada 8 frames) y `GHOST_SPEED` 0.1. Al cambiarlas, usar divisores enteros para no romper la alineación (`aligned()` tolera <1e-3).

## Flujo Spec Driven
- El proyecto existe para practicar desarrollo guiado por specs (ver `.agents/skills/`).
- `/spec` escribe specs en `specs/NN-slug.md` (estado inicial `Draft`) en el idioma del prompt de entrada.
- `/spec-impl` solo implementa specs cuyo estado signifique "Approved"/"Aprobado"; crea la rama `spec-NN-slug` y nunca hace commits automáticos.
- `specs/.spec-config.yml` controla `AutoCreateBranch` (por defecto `true`); `/spec` lo crea si falta.