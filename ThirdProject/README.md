# ThirdProject — Motor de inferencia por enumeración en Redes Bayesianas

## Objetivo

Construir una Red Bayesiana de manera interactiva (definiendo variables, estructura de dependencias y tablas de probabilidad condicional) y calcular distribuciones de probabilidad condicional `P(var_consulta | evidencia)` mediante inferencia exacta.

## Algoritmo implementado

**Inferencia por enumeración en Red Bayesiana**

Dado `P(X | e)`, el algoritmo:

1. Determina el orden topológico de las variables (DFS desde las raíces).
2. Identifica las variables ocultas (ni en la consulta ni en la evidencia).
3. Suma sobre todas las combinaciones de estados de las variables ocultas calculando `P(estado_completo)` como producto de las probabilidades condicionales `CPT[clave_padres][estado]` de cada variable en orden topológico.
4. Normaliza el resultado con el factor `α = 1 / Σ P(X=x | e)` para obtener la distribución completa.

A diferencia del SecondProject (que trabaja sobre lógica simbólica), este proyecto opera sobre probabilidades numéricas y razona con incertidumbre.

## Cómo ejecutar

Requiere Python 3 y `tkinter` (incluido en la distribución estándar de Python).

```bash
cd ThirdProject
python frontend.py
```

Flujo de uso en la GUI:
1. **Panel "Definición de Variables"**: define cada variable con el formato `NomVar(Padre1,Padre2) -> (Estado1,Estado2)`. Las variables raíz no tienen padres: `NomVar -> (Estado1,Estado2)`.
2. Pulsa **"Generar Plantilla de Probabilidades"** para obtener el esqueleto de la CPT con `0.0` en cada entrada.
3. Completa los valores en el panel **"Tablas de Probabilidad (CPT)"** y pulsa **"Cargar Probabilidades"**.
4. En el panel **"Consulta e Inferencia"**, escribe la variable a consultar y opcionalmente la evidencia (`B=si, C=no`) y pulsa **"Ejecutar Inferencia"**.
5. El panel de resultados muestra la distribución completa y el estado más probable.

## Estructura de archivos

| Archivo | Descripción |
|---|---|
| `classes.py` | Clases `VariableAleatoria` (nombre, estados, padres, CPT) y `RedBayesiana` (grafo, orden topológico, inferencia por enumeración, visualización de estructura) |
| `parser.py` | Parsea definiciones de variables (`NomVar(Padre1) -> (Est1,Est2)`) y bloques de CPT en texto plano (`P(Var\|Padre=val) = p1, p2`) |
| `frontend.py` | GUI en Tkinter con tres paneles: definición de variables, edición de CPTs y consulta/resultado; incluye generador de plantillas CPT y visualización de estructura de la red |

## Notas técnicas

- La inferencia por enumeración tiene complejidad exponencial en el número de variables ocultas. Es exacta pero no escala a redes grandes; para ese caso se usarían algoritmos como Belief Propagation o Variable Elimination.
- El orden topológico se recalcula en cada inferencia mediante DFS, lo que garantiza corrección incluso si las variables se agregan fuera de orden.
- La normalización con `α` asegura que la distribución sume 1.0 incluso si hay errores de punto flotante en las CPTs. Si la suma difiere de 1.0 en más de 0.001, la GUI advierte al usuario.
- Las CPTs se indexan como `cpt[(estado_padre1, estado_padre2, ...)][estado_hijo]`. Para variables sin padres, la clave es la tupla vacía `()`.
