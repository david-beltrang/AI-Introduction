# FirstProject — Búsqueda A* para un robot planificador

## Objetivo

Resolver un problema de planificación en el que un robot debe recuperar una batería de emergencia ubicada en la posición 3 de una habitación de tamaño 10. El robot no puede alcanzarla directamente: debe empujar una caja hasta colocarla bajo la batería, subirse a la caja y luego agarrarla.

## Algoritmo implementado

**Búsqueda A\* (A-estrella) con heurística de distancia compuesta**

- `f(n) = g(n) + h(n)`
- `g(n)`: costo acumulado según las acciones ejecutadas (mover: 1, empujar: 2, subir: 1, agarrar: 1)
- `h(n)`: estimación que combina `|posición_caja - posición_batería| + |posición_robot - posición_caja| + penalización_por_no_estar_sobre_caja`

La lista abierta se mantiene ordenada por `f(n)` (min-heap manual con `insertar_ordenado/3`). La lista cerrada evita reevaluar estados ya procesados.

## Cómo ejecutar

Requiere SWI-Prolog instalado.

```bash
swipl robot-battery-box.pl
```

Dentro del intérprete:

```prolog
?- imprimir_solucion.
```

Esto imprime en consola la secuencia de acciones numeradas, por ejemplo:

```
Solución encontrada en 6 pasos:
  Paso 1: mover_derecha
  Paso 2: empujar_derecha
  ...
```

Para obtener solo el camino como lista:

```prolog
?- resolver(Camino).
```

## Estructura de archivos

| Archivo | Descripción |
|---|---|
| `robot-battery-box.pl` | Implementación completa: hechos del mundo (posición batería, límite habitación), operadores de acción, función heurística, algoritmo A\* y predicado de impresión |

## Notas técnicas

- El **estado** se representa como `estado(PosRobot, PosCaja, EncimaCaja, TieneBateria)`. El estado meta es cualquiera donde `TieneBateria = si`.
- El espacio de búsqueda es unidimensional (la habitación es una línea de 1 a 10), lo que hace la heurística exacta en varios casos.
- La función `insertar_ordenado/3` mantiene la lista abierta ordenada por `F` en cada inserción, sin usar una estructura de heap nativa de Prolog.
- `findall` se usa para generar todos los sucesores de un nodo en un solo paso.
- **Nota sobre admisibilidad de la heurística:** La heurística `h(n)` es admisible para este problema (nunca sobreestima el costo real restante), pero **no se realizó una demostración formal de admisibilidad**. El valor `h(n)` resulta ser una cota inferior del costo real porque: (a) `|caja − batería|` ≤ costo real de empujar (empujar cuesta 2 por paso, la heurística cuenta 1), (b) `|robot − caja|` ≤ costo real de mover al robot (cuesta 1 por paso), y (c) la componente constante `penalización + 1 − juntoCaja` suma a lo sumo 2, mientras que el costo real mínimo siempre incluye `subir_caja(1) + agarrar_bateria(1) = 2`. Dado que no hay verificación formal, el nombre "A*" se usa en sentido descriptivo del algoritmo y **no implica una garantía formal de optimalidad**.
