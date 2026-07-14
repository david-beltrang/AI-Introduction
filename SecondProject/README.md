# SecondProject — Motor de inferencia por resolución en LPO

## Objetivo

Demostrar si una consulta en Lógica de Primer Orden (LPO) es consecuencia lógica de una base de conocimiento, usando el método de **refutación por resolución**: se niega la consulta, se convierte todo a Forma Normal Conjuntiva (FNC) y se busca la cláusula vacía.

## Algoritmos implementados

### 1. Conversión a Forma Normal Conjuntiva (FNC) — 7 pasos en cadena

Implementado en `cnf_converter.py` como una pipeline de transformaciones aplicadas recursivamente sobre el AST de la fórmula:

1. Eliminar bicondicional (`A ↔ B` → `(A → B) ∧ (B → A)`)
2. Eliminar implicación (`A → B` → `¬A ∨ B`)
3. Forma Normal de Negación — Leyes de De Morgan + doble negación
4. Estandarización de variables (renombrado único con sufijo `_N`)
5. Skolemización (elimina cuantificadores existenciales con funciones `SK1`, `SK2`, …)
6. Eliminación de cuantificadores universales
7. Distribución de `∨` sobre `∧`

### 2. Resolución con unificación y poda heurística

Implementado en `resolution-py`:

- **Unificación de Robinson** con occurs check (evita `x = f(x)`)
- **Unit Preference**: ordena las cláusulas por longitud antes de cada iteración para resolver primero hechos y cláusulas unitarias
- Límite de seguridad: máximo 15 iteraciones y 800 cláusulas totales
- Poda: no combina pares donde ambas cláusulas tienen más de 3 literales

## Cómo ejecutar

Requiere Python 3 y `tkinter` (incluido en la distribución estándar de Python).

```bash
cd SecondProject
python frontend.py
```

Se abre una ventana con tres secciones:
1. **Base de Conocimiento**: ingresa fórmulas en LPO usando el teclado virtual (botones `∀`, `∃`, `¬`, `∧`, `∨`, `→`, `↔`)
2. **Consulta**: escribe la fórmula a demostrar y pulsa "Probar Inferencia"
3. **Resultado**: muestra el proceso de conversión, las cláusulas generadas y si se encontró la cláusula vacía

El botón "Ver Paso a Paso FNC" abre una ventana secundaria con cada transformación intermedia.

## Estructura de archivos

| Archivo | Descripción |
|---|---|
| `structures.py` | Clases del AST: `Term`, `Literal`, `Clause`, `Predicado`, `Not`, `And`, `Or`, `Implica`, `DobleImplica`, `ParaTodo`, `Existe` |
| `parser.py` | Parser recursivo descendente para fórmulas LPO con soporte de cuantificadores; respeta precedencia `↔ > → > ∨ > ∧` |
| `cnf_converter.py` | Pipeline de 7 transformaciones hacia FNC; solo registra pasos que producen un cambio visual |
| `resolution-py` | Extracción de cláusulas, unificación de Robinson con occurs check, algoritmo de resolución con Unit Preference |
| `frontend.py` | GUI en Tkinter: teclado virtual de símbolos LPO, visualización de pasos FNC y resultado de inferencia |

## Notas técnicas

- El archivo `resolution-py` carece de extensión `.py` pero es importado como módulo desde `frontend.py` usando `from resolution import ...`. En algunas versiones de Python esto puede requerir renombrarlo a `resolution.py`.
- Los predicados se parsean con la convención `NombreCapitalizado(arg1, arg2)`. Las variables se distinguen de constantes por empezar con minúscula (convención usada en la unificación).
- La Skolemización genera funciones `SK1(x)`, `SK2(x, y)`, etc., donde los argumentos son las variables universales en alcance en ese momento.
- La re-parseo del string FNC al final de `execute_inference` (`parsear(finc_f)`) es un workaround para obtener el AST de la fórmula convertida sin modificar la interfaz de `convertir_a_fnc_paso_a_paso`.
