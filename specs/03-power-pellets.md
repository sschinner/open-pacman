# SPEC 03 — Power pellets y modo asustado

> **Status:** Approved
> **Depends on:** SPEC 01, SPEC 02
> **Date:** 2026-09-21
> **Objective:** Añadir 4 power pellets que pongan a los fantasmas en modo asustado (azules, lentos, huidizos) y permitan a Pac-Man comerlos para sumar puntos.

## Scope

**In:**

- 4 power pellets en las posiciones clásicas (1,3), (26,3), (1,23) y (26,23), reemplazando el dot; nueva tile `*` = 4 en `maze.js`, simetría intacta.
- Modo asustado: timer de 360 frames (~6s), fantasmas azules a velocidad `FRIGHT_SPEED` (0.05) que huyen de Pac-Man.
- Comer un fantasma asustado: puntos en combo 200/400/800/1600 por pellet; el fantasma pasa a `eyes`, vuelve al pen (reutiliza SPEC 02) y respawnea.
- Parpadeo azul/blanco en los últimos 120 frames antes de volver a normal.
- Pellet vale 50 pts. Comer un segundo pellet con pánico activo reinicia el timer y el combo.
- Perder una vida cancela el pánico (timer 0, combo `null`, fantasmas a normal) y resetea posiciones.
- Los pellets cuentan en `dotsRemaining`: hay que comerlos para ganar.
- Debug (`?debug=true`): se muestra el modo de cada fantasma junto al `kind`.

**Out of scope (para futuros specs):**

- Movimiento aleatorio de los asustados (el hueco del arcade estricto elige al azar).
- Inversión de dirección en el túnel durante el pánico.
- Ojos más rápidos que el cuerpo (speed distinta de `EYES_SPEED` 0.2).
- Audio/efectos de sonido.
- Highscores o persistencia.
- Cambios a `MAZE` fuera de las 4 celdas de pellet.

## Data model

```js
// maze.js — nuevo tile: '*' = 4 (power pellet), reemplaza un dot
//   fila 3  y fila 23: col 1 y col 26 cambian de '.' a '*'
//   parseTile: '*' -> 4

// game.js — constantes nuevas (divisores enteros de celda)
const DOT_POINTS = 10;
const POWER_PELLET_POINTS = 50;
const FRIGHT_SPEED = 0.05; // 1/20 celda/frame -> alinea cada 20 frames
const EYES_SPEED = 0.2;    // 1/5 celda/frame  -> alinea cada 5 frames
const FRIGHT_FRAMES = 360; // ~6s a 60fps
const FLASH_FRAMES = 120;  // ~2s finales de parpadeo

// game.js — estado por fantasma y global
//   ghost: { ..., mode: 'normal' }   // 'normal' | 'frightened' | 'eyes'
//   game:  { ..., frightTimer: 0, ghostCombo: null }  // combo 200,400,800,1600
```

Reglas de diana (`targetFor`, en orden):

- `inPen( g )` → `PEN_EXIT_TARGET` (sin cambios, SPEC 02).
- `g.mode === 'eyes'` → centro del pen (`{ x: 13, y: 14 }`), y al entrar `inPen` vuelve a `normal` y sale por la puerta.
- Resto → reglas de `kind` de SPEC 01.

Regla de huida en `decideGhost`: `flee` = `g.mode === 'frightened'` || (`kind === 'shy'` y distancia ≤ 8).

## Implementation plan

1. `maze.js`: parsear `*` → 4, reemplazar los 4 dots por `*`; `game.js`: `dotsRemaining` cuenta tiles 2 y 4; `render.js`: `drawPowerPellets` (radio ~5, color del dot). Verificación: cargar, se ven 4 pellets grandes simétricos, sin errores en consola.
2. `game.js`: constantes nuevas, `mode: 'normal'` en el ghost y `frightTimer`/`ghostCombo` en la partida; comer tile 4 otorga 50 pts, reinicia timer/combo y pone los fantasmas (modo `normal`) en `frightened` con `FRIGHT_SPEED`; al llegar a 0 el timer los devuelve a `normal`. Verificación: comer un pellet azulea y lentifica a los fantasmas ~6s y vuelven a normal.
3. `game.js`: `targetFor` para `eyes` (centro del pen) y `flee` en `decideGhost` para `frightened`; en `moveGhost`, si `mode === 'eyes' && inPen( g )` → `normal` + `GHOST_SPEED`. Verificación: con `?debug=true`, en pánico la diana es la celda de Pac-Man y huyen; los ojos vuelven al pen.
4. `game.js`: colisión — si el fantasma está `frightened` lo come (suma `ghostCombo`, dobla, pasa a `eyes` + `EYES_SPEED`); si `eyes` se ignora; si `normal` pierde una vida y `resetPositions` fuerza timer 0, combo `null` y mode `normal`. Verificación: comer 4 fantasmas en un pánico suma 200/400/800/1600; morir resetea todo.
5. `render.js`: dibujar `eyes` (solo ojos, sin cuerpo), cuerpo azul `#2121ff` en pánico, parpadeo blanco (alterna cada ~8 frames) cuando `frightTimer ≤ FLASH_FRAMES`, y etiqueta de modo en el debug. Verificación: azul/ojos/parpadeo y debug correctos en todas las fases.

## Acceptance criteria

- [ ] Hay 4 power pellets en (1,3), (26,3), (1,23), (26,23), reemplazando dots; el `MAZE` sigue simétrico.
- [ ] Comer un pellet otorga 50 pts y al instante todos los fantasmas activos se vuelven azules, lentos y huyen de Pac-Man durante ~6s.
- [ ] Con `?debug=true`, en pánico la diana de cada fantasma es la celda redondeada de Pac-Man (huida por distancia máxima).
- [ ] Comer un fantasma asustado suma 200, y cada fantasma del mismo pellet 400, 800, 1600 (combo se duplica).
- [ ] El fantasma comido queda reducido a ojos, cruza la puerta al pen y respawnea con su color normal.
- [ ] Comer un segundo pellet con pánico activo reinicia el timer a 360 frames y el combo a 200.
- [ ] En los últimos 120 frames del pánico los fantasmas parpadean azul/blanco y al terminar vuelven a normal.
- [ ] Perder una vida cancela el pánico: timer 0, combo `null`, fantasmas a `normal` y posiciones del pen (SPEC 02).
- [ ] Comer todos los dots y pellets muestra GANASTE; perder 3 vidas muestra PERDISTE.
- [ ] Con `?debug=true` se ve el modo (`normal`/`frightened`/`eyes`) junto al `kind`; sin el parámetro, nada extra.
- [ ] No hay errores en consola al cargar, jugar y reiniciar.

## Decisions

- **Sí:** 4 pellets en las posiciones clásicas (1,3),(26,3),(1,23),(26,23). Encajan en dots existentes y respetan la simetría.
- **Sí:** huida por distancia Manhattan en pánico. Reutiliza la lógica del `shy`; **no** movimiento aleatorio, que exigiría un mecanismo aparte en `decideGhost`.
- **Sí:** parpadeo final de 120 frames. Elegido por fidelidad al arcade (se descartó la opción sin aviso).
- **Sí:** `FRIGHT_SPEED` 0.05 y `EYES_SPEED` 0.2, ambos divisores enteros (regla del proyecto). Ojos más rápidos que el cuerpo, como el arcade.
- **Sí:** timer de 360 frames, coherente con el bucle `requestAnimationFrame` del proyecto.
- **Sí:** combo 200/400/800/1600 reseteado por pellet; **no** puntos fijos por fantasma.
- **Sí:** ojos → pen → respawn reutilizando la detección por coordenadas de SPEC 02, sin temporizador de respawn. **No** respawn instantáneo ni ojos esperando al siguiente pellet.
- **Sí:** morir cancela el pánico (`resetPositions` centraliza el reset).
- **Sí:** `mode` por fantasma (permite distinguir `eyes`) más `frightTimer`/`ghostCombo` globales en `game`; **no** flag booleano global.
- **No:** cambios al orden de `<script>` ni archivos JS nuevos.

## Risks

| Riesgo | Mitigación |
| --- | --- |
| Los asustados huyendo se agrupan o quedan en callejones (empate de distancia) | Desempate determinista por orden de `DIRS`; efecto arcade aceptable, igual que SPEC 01. |
| Comer 2 fantasmas en el mismo frame rompe el combo | Trazar colisión por fantasma; el ya convertido en `eyes` se ignora, el combo se aplica una vez. |
| Timer por frames varía con el FPS del navegador | Igual que el resto del juego (velocidades por frame); aceptable para el proyecto. |
| Fantasma se queda `frightened` tras morir Pac-Man | `resetPositions` fuerza timer 0, combo `null` y `mode` `normal` en todos. |

## What is **not** in this spec

- Movimiento aleatorio de los asustados (arcade estricto).
- Inversión de dirección en el túnel durante el pánico.
- Ojos a velocidad distinta de `EYES_SPEED` 0.2.
- Audio, highscores o persistencia.
- Cambios a `MAZE` fuera de las 4 celdas de pellet.
- Archivos JS nuevos o cambios al orden de `<script>`.

Cada una de esas, si llega, va en su propio spec.