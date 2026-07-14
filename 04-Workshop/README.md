# 04-Workshop — Tic-Tac-Toe con Minimax + Poda Alpha-Beta

## Objetivo

Implementar un agente que juega Tres en Raya (Tic-Tac-Toe) de forma óptima contra un humano, garantizando que nunca pierde. El humano juega con X (turno 0) y la IA juega con O (turno 1).

## Algoritmo implementado

**Minimax con poda Alpha-Beta**

Implementado en `Minimax.java`. En cada turno de la IA:

1. `getBestMove` recorre el árbol de juego completo desde el estado actual.
2. En cada nodo, `getOpenSpotsIndexes` genera los movimientos legales disponibles.
3. Se simula cada movimiento (coloca/quita un `Marker` en la matriz) y se llama recursivamente a `minimax`.
4. La **poda alpha-beta** corta ramas cuando `alpha > beta` (turno del solicitante) o `beta < alpha` (turno del oponente), reduciendo nodos evaluados.
5. La función de evaluación `getFieldScore` devuelve `SIZE - depth` para victoria del solicitante y `-(SIZE) + depth` para derrota, favoreciendo victorias rápidas sobre victorias lentas.

A diferencia del 03-Workshop (deductivo/declarativo en Prolog), aquí el agente razona sobre un espacio de estados mediante búsqueda adversarial.

## Cómo ejecutar

Requiere JDK 8 o superior. Compilar desde la raíz del repositorio:

```bash
cd "04-Workshop/src"
javac *.java
java Main
```

Se abre una ventana de 600×600 px con el tablero. El jugador humano hace clic en una celda para colocar X. La IA responde automáticamente con O. Al terminar el juego (victoria o empate), se muestra una pantalla de fin de partida y un clic reinicia el juego.

> **Nota:** las imágenes `x.png` y `o.png` se cargan con ruta relativa `04-Workshop/assets/`, por lo que el comando `java Main` debe ejecutarse desde la raíz del repositorio (no desde dentro de `src/`).

## Estructura de archivos

| Archivo | Descripción |
|---|---|
| `Main.java` | Punto de entrada; crea el `JFrame` y el `GamePanel`. Define constantes globales: `WIDTH=600`, `HEIGHT=600`, `ROWS=3`, `MATCH=3`, `SIZE=9` |
| `Minimax.java` | Algoritmo Minimax con poda Alpha-Beta; expone `getBestMove(markers, turn)` que devuelve el índice lineal del mejor movimiento |
| `AI.java` | Adaptador entre `Minimax` y `Grid`; llama a `getBestMove` y delega en `grid.placeMarker` |
| `Checker.java` | Detecta combinaciones ganadoras en las 8 direcciones (filas, columnas, 2 diagonales) mediante `checkWin` y `checkMatch` |
| `Grid.java` | Estado del tablero (`Marker[3][3]`), gestión de turnos, detección de fin de juego y reset |
| `GamePanel.java` | Panel principal; conecta eventos de ratón del usuario con `Grid` y dispara el movimiento de la IA tras cada jugada humana |
| `Panel.java` | Game loop a 30 FPS con delta time; clase base para `GamePanel` |
| `Marker.java` | Representación visual de X/O; incluye animación de parpadeo para la combinación ganadora |
| `Placement.java` | Zona clicable de cada celda; maneja hover con fade-in/fade-out |
| `IGameObject.java` | Interfaz `update(float) / render(Graphics2D)` implementada por todos los objetos del juego |

## Notas técnicas

- El tablero es 3×3, por lo que el árbol minimax tiene como máximo 9! = 362 880 hojas. Con poda alpha-beta el número real de nodos evaluados es significativamente menor (el juego imprime "Minimax evaluó N movimientos posibles" en cada turno de la IA).
- El índice lineal de una celda se calcula como `x + y * ROWS`, donde `x` es columna e `y` es fila. `placeMarker(int index)` lo decodifica con `index % ROWS` y `index / ROWS`.
- `Main.SIZE` y `Main.ROWS` son estáticos y mutables; en teoría se podría cambiar `ROWS` a 4 para un tablero 4×4, aunque `Checker` y la heurística no están parametrizados para ello.
