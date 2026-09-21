# SPEC 01 — 4 fantasmas con personalidades clásicas

> **Status:** Implemented
> **Date:** 2026-09-20
> **Objective:** Sustituir los 2 fantasmas genéricos por 4 con personalidades clásicas distintas, una de ellas persiguiendo agresivamente a Pac-Man.

## Scope

**In:**

- 4 fantasmas con 4 reglas de decisión propias: `chaser`, `ambusher`, `flanker`, `shy`.
- El `chaser` persigue agresivamente la celda de Pac-Man (ya existe como `hunter`).
- Mapeo `kind` → color clásico en el render (rojo, rosa, cian, naranja).
- Los 4 activos desde el inicio de la partida, en 4 celdas del pen.
- Modo debug con `?debug=true`: dibuja la celda-diana y el `kind` de cada fantasma.

**Out of scope (para futuros specs):**

- Modo asustado (power pellets, fantasmas azules vulnerables, puntos al comerlos).
- Ciclo chase/scatter del arcade (fases temporizadas con esquinas como diana).
- Liberación escalonada desde el pen.
- Cualquier cambio a `MAZE` (dots, power pellets, geometría).

## Data model

No hay estructuras nuevas; se extienden las existentes.

```js
// maze.js — de 2 a 4 posiciones, cada una con su kind
const GHOST_STARTS = [
  { x: 13, y: 14, kind: 'chaser' },   // rojo    — perseguidor agresivo
  { x: 14, y: 14, kind: 'ambusher' }, // rosa    — emboscada delante de Pac-Man
  { x: 13, y: 13, kind: 'flanker' },  // cian    — flanco en función del chaser
  { x: 14, y: 15, kind: 'shy' },      // naranja — tímido
];

// game.js — dianas según kind (celdas, no se exige que sean transitables)
//   chaser  → celda redondeada de Pac-Man
//   ambusher→ celda de Pac-Man + 4 · DIRS[dir de Pac-Man]
//   flanker → A = celda de Pac-Man + 2 · DIRS[dir]; diana = chaser + 2·(A − chaser)
//   shy     → si dist. Manhattan a Pac-Man > 8 → diana = celda de Pac-Man
//             si ≤ 8                             → huir (elegir opción que más aleje)

// render.js — color por kind (reemplaza el mapeo por índice)
const GHOST_COLORS = {
  chaser: '#ff0000', ambusher: '#ffb8ff',
  flanker: '#00ffff', shy: '#ffb852',
};
```

`decideGhost` unifica la elección: para todo `kind`, elegir entre las opciones transitables (sin reversa) la que minimiza distancia Manhattan a su diana; para `shy` en rango de huida, la que la maximiza. Se mantiene el fallback de giro de 180° sin salida.

## Implementation plan

1. `maze.js`: ampliar `GHOST_STARTS` a las 4 entradas anteriores. Verificación: cargar `index.html`, salen 4 fantasmas de la pen.
2. `game.js`: en `decideGhost`, reemplazar el `if ( g.kind === 'hunter' ) / else` por la resolución de diana por `kind` (helper `targetFor( game, g )`) más la elección min/max Manhattan común. `createGame` ya propaga `kind` desde `GHOST_STARTS`. Verificación: jugar, cada fantasma se mueve según su regla.
3. `render.js`: mapear `GHOST_COLORS` por `kind` en lugar de por índice (`drawGhost` recibe el color del dict). Verificación: colores clásicos correctos.
4. `main.js` + `render.js`: parsear `?debug=true` en `main.js` → global `window.DEBUG`; si está activo, `render.js` dibuja la celda-diana (marcador en el color del fantasma) y una etiqueta con el `kind`. Verificación: recargar con `?debug=true` y sin él.

## Acceptance criteria

- [ ] Se ven 4 fantasmas activos desde el inicio, en rojo, rosa, cian y naranja.
- [ ] La diana del `chaser` (debug) coincide con la celda redondeada de Pac-Man.
- [ ] La diana del `ambusher` (debug) está 4 celdas delante de Pac-Man según su dirección.
- [ ] La diana del `flanker` (debug) está simétrica al chaser respecto a la celda 2 delante de Pac-Man.
- [ ] El `shy` persigue cuando está a >8 celdas de Pac-Man y se aleja cuando está a ≤8.
- [ ] Con `?debug=true` se dibujan diana e `kind`; sin el parámetro no se dibuja nada extra.
- [ ] No hay errores en la consola al cargar, jugar y reiniciar.
- [ ] Comer todos los dots muestra GANASTE; perder 3 vidas muestra PERDISTE.

## Decisions

- **Sí:** personalidades clásicas (`chaser`, `ambusher`, `flanker`, `shy`). El requisito "perseguir agresivamente" lo cumple `chaser`.
- **No:** mantener `hunter`/`random`. Son los dos únicos estados previos y ambos desaparecen.
- **Sí:** solo chase, sin ciclo scatter. Dianas constantes, sin reloj ni fases; el tímido ya se diferencia acercándose/alejándose.
- **No:** modo asustado con power pellets. Feature grande, va en su propio spec.
- **No:** liberación escalonada del pen. Todos activos al inicio, se evita temporizador de liberación.
- **Sí:** `kind` descriptivos en inglés, coherente con los valores actuales de `game.js`.
- **Sí:** debug vía `?debug=true`. Permite verificar cada personalidad sin adivinar.
- **No:** archivo `ghosts.js` nuevo. La lógica cabe en `decideGhost` y se evita tocar el orden de `<script>`.

## Risks

| Riesgo | Mitigación |
| --- | --- |
| Varias opciones con igual distancia en un cruce (dianas ambiguas) | Desempate por orden fijo de `DIRS` (`left,right,up,down`), determinista y reproducible. |
| 4 perseguidores hacen la partida frustrante/muy rápida | Prueba manual de dificultad; en su defecto, ajustar `GHOST_SPEED` a un divisor entero (regla del proyecto). |

## What is **not** in this spec

- Power pellets y modo asustado.
- Ciclo chase/scatter.
- Liberación escalonada.
- Archivo `ghosts.js` nuevo y cambios al orden de `<script>`.
- Modificaciones al laberinto (`MAZE`).

Cada una de esas, si llega, va en su propio spec.