# 03-Workshop — Árbol genealógico en Prolog (familia Potter/Black)

## Objetivo

Modelar el árbol genealógico del universo Harry Potter como una base de conocimiento en Prolog y derivar automáticamente relaciones familiares de cualquier grado (hasta tatarabuelo, primo cuarto, tío bisabuelo, etc.) y la clasificación de "pureza de sangre" del canon.

## Algoritmos implementados

**Razonamiento lógico deductivo con Prolog (resolución SLD)**

No hay un algoritmo de búsqueda externo: el motor de inferencia de Prolog resuelve las consultas mediante resolución SLD con backtracking. Los elementos técnicos relevantes son:

- **Recursión con caso base y recursivo** para `es_descendiente/2` y `es_ancestro/2`, permitiendo recorrer la genealogía a profundidad arbitraria.
- **Corte (`!`)** en `es_sangre_pura/2` y `nivel_sangre/3` para evitar backtracking innecesario una vez hallada la primera solución válida.
- **Negación por falla (`\+`)** para verificar que una persona no pertenece a familia de muggles.
- Reglas **compuestas en cadena** para relaciones extendidas: `es_primo_cuarto(A,B)` requiere 5 llamadas encadenadas a `progenitor/2`.

A diferencia del FirstProject (que también usa Prolog), aquí no hay búsqueda heurística ni planificación: el problema es puramente declarativo/deductivo.

## Cómo ejecutar

Requiere SWI-Prolog instalado.

```bash
swipl Harry_potter_family_tree.pl
```

Ejemplos de consultas:

```prolog
% ¿Es Harry Potter descendiente de Phineas Nigellus Black?
?- es_descendiente(harry_potter, phineas_nigellus_black).

% ¿Cuál es el nivel de sangre de Draco Malfoy?
?- nivel_sangre(draco_malfoy, Nivel).

% ¿Quiénes son los primos de Harry Potter?
?- es_primo(harry_potter, X).

% ¿Son Ron Weasley y Harry Potter parientes?
?- es_pariente(ronald_weasley, harry_potter).

% ¿Cuál es el nivel de sangre de Nymphadora Tonks?
?- nivel_sangre(nymphadora_tonks, Nivel).
```

## Estructura de archivos

| Archivo | Descripción |
|---|---|
| `Harry_potter_family_tree.pl` | Base de conocimiento completa: hechos (`es_padre/2`, `es_madre/2`, `es_pareja/2`, `sangre_pura/1`, `familia_de_muggles/1`) y reglas de 9 niveles de parentesco más clasificación de sangre |

## Notas técnicas

- La base de conocimiento cubre tres generaciones de la familia Black, la familia Potter, los Weasley y líneas relacionadas (Malfoy, Tonks, Lupin, Granger, Dursley).
- `es_hermano/2` tiene dos definiciones: una como hecho explícito (para Lily/Petunia) y otra como regla derivada (mismo padre y madre). Esto puede generar respuestas duplicadas en algunas consultas; usar `setof/3` para evitarlo.
- La regla `nivel_sangre/3` distingue tres casos: `pura` (ambos padres puros, sin vínculo muggle), `muggle` (en la lista `familia_de_muggles/1`) y `mestiza` (tiene al menos un ancestro mágico puro pero no cumple la condición de pureza completa).
- `es_pariente/2` usa `es_ancestro/2` de forma cruzada: dos personas son parientes si comparten al menos un ancestro común. Puede ser lento en consultas muy amplias por el espacio de búsqueda recursivo.
