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
| Andrey Lechuga | Suite de 6 pruebas (5 obligatorias + 1 caza-errores) y auditoría de integración |
| Izquierdo | Visualización en mapas de calor y gráficas comparativas |
| Ramón | Documentación, justificaciones, IA check y control de Git |

## 5. Criterios de aceptación técnicos
- Código funcional, limpio, con type hints y solo `numpy` como dependencia.
- Métricas reales (costo, nodos explorados), no inventadas.
- Soporte a cambios dinámicos del terreno (`set_cell`, `set_region_risk`, `reset`) sin reconstruir el entorno.
- Heurística de A* admisible respecto a este costo (el costo mínimo por paso es la distancia base, ya que $1+\alpha R^\beta \ge 1$).

## 6. Alcance de los módulos implementados
- **Módulo 1 (Jesús):** `GridEnvironment` + función de costo (Celdas 1–3 de `modelado_terreno.ipynb`).
- **Módulo 2 (Martín):** Algoritmos `dijkstra` y `a_star`, heurística admisible `heuristic`, estructura `SearchResult` con conteo exacto de `nodes_explored` y `visited_order`, y pruebas de humo de búsqueda (Celdas 4–6 de `modelado_terreno.ipynb`). Pruebas exhaustivas y visualización son de los siguientes módulos y consumen `SearchResult`.
- **Módulo 3 (Andrey Lechuga):** Suite automatizada de los 5 escenarios obligatorios (1. Base, 2. Bloqueo dinámico, 3. Inaccesibilidad, 4. Cambio de riesgo antes/después, 5. Camino más corto vs. más conveniente) más el Escenario 6 caza-errores (relajación tardía en vado inundado $R=3$, asimetría direccional de evacuación y validaciones de frontera/diagonal en `cost(u, v)`), almacenando resultados listos para graficar en `ESCENARIOS_PRUEBA` y `HISTORIAL_METRICAS` (Celdas 7–9 de `modelado_terreno.ipynb`).
- **Módulo 4 (Izquierdo):** Visualización en mapas de calor categóricos de los escenarios de evacuación, superposición de rutas óptimas ($A^*$ y Dijkstra), visualización comparativa de la Pregunta Eje (Escenario 5: ruta más corta que cruza $R=3$ con $J=71.35$ vs ruta más conveniente que rodea por $R=0$ con $J=13.00$, reducción del 81.8% en costo), y dashboard comparativo de métricas de rendimiento (nodos explorados, costos y tiempos de cómputo) (Celdas 10–14 de `modelado_terreno.ipynb`).

