# PROJECT CHARTER — "Cuando el camino más corto no es el mejor"

Fuente única de verdad del proyecto.

## 1. Declaración del problema
Evacuación de una zona inundada hacia un refugio. La ruta de menor distancia física no necesariamente es la mejor: puede atravesar zonas con mayor nivel de agua y, por tanto, mayor riesgo. El sistema debe elegir rutas minimizando un **costo** que combina distancia y riesgo.

## 2. Pregunta eje de la rúbrica
> *"Si dos rutas llegan al mismo refugio, pero una es más corta y la otra atraviesa zonas de mayor riesgo, ¿cómo determina el sistema cuál es la mejor?"*

Respuesta: compara el **costo acumulado** $\sum C(u,v)$ de cada ruta, no su longitud. Ver [MODELADO.md](MODELADO.md) §4.

## 3. Restricciones inquebrantables
- Penalizar el riesgo de inundación frente a la distancia física.
- Manejar bloqueos absolutos (`-1`, costo $\infty$) sin ciclarse (las celdas bloqueadas nunca se devuelven como vecinas).
- Todo contenido en el notebook principal `.ipynb` y la documentación de respaldo.

## 4. Mapa de responsabilidades
| Integrante | Responsabilidad |
|---|---|
| Jesús | Modelado del terreno, función de costo y base de datos matricial |
| Martín | Algoritmos de búsqueda (A* y Dijkstra) y conteo de nodos explorados |
| Lechuga | Suite de 6 pruebas (5 obligatorias + 1 caza-errores) |
| Izquierdo | Visualización en mapas de calor y gráficas comparativas |
| Ramón | Documentación, justificaciones, IA check y control de Git |

## 5. Criterios de aceptación técnicos
- Código funcional, limpio, con type hints y solo `numpy` como dependencia.
- Métricas reales (costo, nodos explorados), no inventadas.
- Soporte a cambios dinámicos del terreno (`set_cell`, `set_region_risk`, `reset`) sin reconstruir el entorno.
- Heurística de A* admisible respecto a este costo (el costo mínimo por paso es la distancia base, ya que $1+\alpha R^\beta \ge 1$).

## 6. Alcance de este módulo (Jesús)
`GridEnvironment` + función de costo. Búsqueda, pruebas y visualización son de otros integrantes y consumen `get_neighbors`.
