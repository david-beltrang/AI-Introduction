# AI Introduction — Course Projects

Proyectos y talleres desarrollados para el curso de Introducción a la Inteligencia Artificial en Ingeniería de Sistemas, Pontificia Universidad Javeriana. El repositorio cubre cinco áreas: búsqueda informada, lógica de primer orden (programación en Prolog y motor de resolución), razonamiento probabilístico con redes bayesianas, juegos adversariales con búsqueda minimax, y un dataset de ML supervisado.

## Contenido

| Carpeta | Tema | Stack |
|---|---|---|
| [FirstProject](./FirstProject/) | Búsqueda informada — A\* para planificación de un robot | Prolog (SWI-Prolog) |
| [SecondProject](./SecondProject/) | Lógica de Primer Orden — Motor de inferencia por resolución con conversión a FNC | Python 3, Tkinter |
| [ThirdProject](./ThirdProject/) | Razonamiento probabilístico — Inferencia por enumeración en Redes Bayesianas | Python 3, Tkinter |
| [FourthProject](./FourthProject/) | Dataset de clasificación — Telco Customer Churn (ML supervisado) | CSV (IBM Watson) |
| [03-Workshop](./03-Workshop/) | Lógica declarativa — Árbol genealógico Harry Potter con razonamiento deductivo | Prolog (SWI-Prolog) |
| [04-Workshop](./04-Workshop/) | Juegos adversariales — Tic-Tac-Toe con Minimax + Poda Alpha-Beta | Java, Swing |

Cada carpeta tiene su propio README con detalles de implementación y ejecución.

## Progresión del curso

El contenido sigue una progresión desde razonamiento exacto hacia razonamiento bajo incertidumbre:

1. **Búsqueda informada** (FirstProject, 03-Workshop): el agente conoce el mundo completamente y busca una secuencia óptima de acciones. FirstProject usa A\* en un espacio de estados numérico; 03-Workshop usa inferencia deductiva en Prolog sobre un grafo de conocimiento.

2. **Lógica de Primer Orden y resolución** (SecondProject): el agente razona sobre conocimiento simbólico con cuantificadores y variables. La demostración se reduce a búsqueda de la cláusula vacía tras la conversión a FNC.

3. **Razonamiento probabilístico** (ThirdProject): el mundo es incierto. En lugar de verdad/falsedad, el agente calcula distribuciones de probabilidad condicionadas a evidencia observada.

4. **Juegos adversariales** (04-Workshop): búsqueda en árbol con dos agentes en competencia; la poda alpha-beta optimiza el recorrido del mismo árbol minimax.

5. **Machine Learning supervisado** (FourthProject): el dataset representa el siguiente paso natural, donde el conocimiento no se programa sino que se aprende de datos.

## Autor

David Beltrán Gómez — Ingeniería de Sistemas, Pontificia Universidad Javeriana
