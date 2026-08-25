# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Clon del arcade **Asteroids** en Canvas HTML5 puro. Sin dependencias, sin bundler, sin build step. Todo el juego vive en `game.js` (un solo archivo), renderizado sobre el `<canvas>` definido en `index.html`.

## Running

No hay proceso de build ni tests. Para probar cambios, abre `index.html` directamente en el navegador o sirve el directorio:

```bash
npx serve .
```

Luego visita `http://localhost:3000`. Cualquier cambio en `game.js` solo requiere recargar la página (no hay hot reload ni compilación).

## Architecture

Todo el estado y la lógica están en `game.js`, organizado en secciones delimitadas por comentarios `// ── Nombre ──`:

- **Input**: `keys` (estado continuo por tecla) y `justPressed` (edge-trigger de una sola vez, consumido vía `pressed(code)`) — así distingue "mantener presionado" (rotar/propulsar) de "un solo disparo por pulsación".
- **Utils**: `wrap` (envolvimiento toroidal de posición dentro de los bordes del canvas), `dist`, `rand`, `randInt`.
- **Entidades como clases** (`Bullet`, `Asteroid`, `Ship`, `Particle`): cada una expone `update(dt)` y `draw()`, y se marca a sí misma con `dead = true` cuando debe eliminarse. El filtrado de muertos ocurre centralizado en el loop de `update()` global, no dentro de cada entidad.
- **Estado del juego**: variables globales (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`) reinicializadas por `initGame()`. `state` es una máquina de estados simple: `'playing' | 'dead' | 'gameover'`.
- **Loop principal**: `requestAnimationFrame` con `dt` en segundos (clamped a 0.05s para evitar saltos grandes tras pausas del tab). Cada frame: `update(dt)` → `draw()`.
- **Colisiones**: fuerza bruta O(n·m) por distancia entre centros (`dist(a, b) < radioSuma`) — bala↔asteroide y nave↔asteroide. Aceptable dado el volumen bajo de entidades; no hay spatial partitioning.
- **Tamaños de asteroide**: 3 (grande) → 2 (mediano) → 1 (pequeño), indexando arrays paralelos `RADII`, `SPEEDS`, `POINTS`. Al morir, un asteroide de tamaño >1 se parte en 2 vía `split()`.
- **Progresión de nivel**: cuando `asteroids.length === 0`, `nextLevel()` incrementa `level` y hace spawn de `3 + level` asteroides grandes.

Todo el código y comentarios del juego están en español, consistente con el `README.md`.
