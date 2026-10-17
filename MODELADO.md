# MODELADO — Terreno y función de costo

## 1. Terreno
Cuadrícula $G$ de `rows × cols`. Cada celda almacena un nivel de riesgo entero (matriz `numpy` `int8`).

| Nivel | Significado | $R(v)$ | Factor $1+\alpha R^\beta$ (α=2, β=1.5) |
|---|---|---|---|
| 0 | Seco | 0 | 1.00 |
| 1 | Anegación leve | 1 | 3.00 |
| 2 | Anegación moderada | 2 | 6.66 |
| 3 | Anegación severa | 3 | 11.39 |
| -1 | Bloqueo total (inundación crítica) | — | $\infty$ |

## 2. Función de costo
$$C(u,v)=\begin{cases} d(u,v)\,\bigl(1+\alpha\,R(v)^{\beta}\bigr) & \text{si } v \text{ transitable}\\ \infty & \text{si } R(v)=-1\end{cases}$$

- $d(u,v)$: distancia base; $1$ para movimiento ortogonal, $\sqrt2$ para diagonal.
- $R(v)\in\{0,1,2,3\}$: nivel de riesgo de la celda **destino**.
- $\alpha\ge 0$: aversión al riesgo (qué tanto se castiga el agua frente a la distancia).
- $\beta\ge 1$: exponente no lineal; con $\beta>1$ el riesgo alto se castiga más que proporcionalmente.

Propiedades: $C(u,v)\ge d(u,v)>0$ (sin aristas negativas → Dijkstra válido; $d$ es una cota inferior admisible para A*). Con $R=0$ el costo se reduce a distancia pura.

## 3. Decisiones de implementación
- **Base de datos matricial (`self.grid`):** Matriz `numpy.ndarray` (`int8`) manipulable por celda (`set_cell`), por región rectangular (`set_region_risk`), o cargando una matriz completa (`load_matrix`).
- Las celdas bloqueadas **no se devuelven** en `get_neighbors` (equivale a $C=\infty$ y evita ciclos o expansión de nodos imposibles). `cost()` sí devuelve `inf`.
- **Corner cutting:** con diagonales (`allow_diagonal=True`), el movimiento diagonal se impide cuando **las dos** celdas ortogonales intermedias están bloqueadas (`is_blocked(r + dr, c) and is_blocked(r, c + dc)`).
- Costo basado en la celda destino: entrar a agua profunda cuesta, salir de ella no.

## 4. Respuesta a la pregunta eje
Dos rutas al mismo refugio se comparan por costo acumulado $J(P)=\sum_{(u,v)\in P}C(u,v)$, no por longitud. Una ruta corta que cruza celdas de nivel 3 paga ~11.4× por paso; por ejemplo, 4 pasos por nivel 3 cuestan ≈ 45.6, mientras que un rodeo seco de 10 pasos cuesta 10. El sistema elige el rodeo. La ruta corta solo gana si el desvío cuesta más que el riesgo evitado; $\alpha$ y $\beta$ controlan ese equilibrio. Si la única ruta está bloqueada, $J=\infty$ y se reporta que no hay camino.

## 5. Algoritmos de búsqueda y heurística (Módulo 2)
- **Contrato de salida (`SearchResult`):** Estructura estándar con soporte de atributos y diccionario que expone `path: list[Node]`, `cost: float`, `nodes_explored: int` y `visited_order: list[Node]`.
- **Dijkstra (`dijkstra`):** Búsqueda de costo uniforme ($f(n)=g(n)$) sobre `env.get_neighbors(u)`. Garantiza optimalidad dado que $C(u,v) \ge d(u,v) \ge 1 > 0$.
- **A* (`a_star`):** Búsqueda informada ($f(n)=g(n)+h(n)$) con heurística admisible y consistente adaptada a `env.allow_diagonal`:
  - Ortogonal (`allow_diagonal=False`): **Distancia Manhattan** $h(n,g) = \Delta r + \Delta c$.
  - Diagonal (`allow_diagonal=True`): **Distancia Octile** $h(n,g) = \max(\Delta r, \Delta c) + (\sqrt{2}-1)\min(\Delta r, \Delta c)$ ($\ge$ Euclidiana y $\le$ costo real).
- **Admisibilidad:** Como $1+\alpha R(v)^\beta \ge 1$ para todo $R(v)\in\{0,1,2,3\}$, se cumple $C(u,v) \ge d(u,v)$ en cada paso y $h(n) \le h^*(n)$ siempre.
- **Manejo de bloqueos (`-1`):** Si `start` o `goal` están bloqueados (`env.is_blocked`) o el refugio está encerrado por celdas `-1`, ambos algoritmos terminan limpiamente devolviendo `path=[]` y `cost=math.inf`.

## 6. Suite de escenarios de prueba y auditoría caza-errores (Módulo 3 — Andrey Lechuga)
Se implementaron y verificaron en `modelado_terreno.ipynb` (Celdas 7–9) los **5 escenarios obligatorios** más **1 escenario adicional caza-errores**, almacenando los resultados en `ESCENARIOS_PRUEBA` y `HISTORIAL_METRICAS`:

| Clave | Escenario | Costo $J(P)$ | Pasos | Nodos (Dijkstra / A*) | Resultado verificado |
|---|---|---:|---:|---:|---|
| `E1_Base` | 1. Ruta base disponible ($10\times 10$) | `20.00` | `18` | `89` / `44` (-50.6%) | Cruza por el puente principal `(4, 4)` con $R=1$. |
| `E2_Bloqueo_Dinamico` | 2. Bloqueo dinámico en ruta previa | `26.00` | `18` | `66` / `48` (-27.3%) | Se bloquea `(4, 4)` (`-1`); recalcula por el desvío oriental. |
| `E3_Inaccesible` | 3. Refugio completamente inaccesible | `inf` | `-` | `40` / `40` | Canal fila 4 cerrado por completo; retorna `path=[]` sin ciclarse. |
| `E4A_Riesgo_Leve` | 4A. Antes: corredor central con $R=1$ | `13.00` | `9` | `42` / `14` (-66.7%) | Atraviesa el corredor central en línea recta ($13.00 < 15.00$). |
| `E4B_Riesgo_Severo` | 4B. Después: corredor sube a $R=3$ | `15.00` | `15` | `42` / `24` (-42.9%) | Cambia de decisión y rodea por zona seca ($15.00 < 50.57$). |
| `E5_Corto_vs_Conveniente` | 5. Camino más corto vs. más conveniente | `13.00` | `13` | `53` / `17` (-67.9%) | Descarta la ruta recta de `9` pasos ($J=71.35$) por el rodeo seco de `13` pasos ($J=13.00$, **81.8% menor costo**). |
| `E6_Caza_Errores` | 6. Caza-errores (Relajación en vado $R=3$, asimetría y fronteras) | `14.81` | `4` | `10` / `10` | Verifica relajación tardía ortogonal `(2,1)->(2,2)` ($J=14.81$) frente al bug de visitado prematuro en diagonal `(1,1)->(2,2)` ($J=18.53$), asimetría direccional ($3.00 \neq 13.39$) y validaciones de `cost(u, v)`. |

### Correcciones de integración aplicadas en Módulos 1 y 2
1. **Validación completa en `GridEnvironment.cost(u, v)` (Módulo 1):** Se añadió validación de límites para `u` y `v` (`IndexError`), verificación de bloqueo en el nodo origen `u` (`is_blocked(*u)`), y restricción de movimientos diagonales cuando `allow_diagonal=False` (`ValueError`).
2. **Compatibilidad extendida en `SearchResult` (Módulo 2):** Se preservó intacto `to_dict()` con las 4 llaves originales de Martín (`path`, `cost`, `nodes_explored`, `visited_order`) y se añadieron propiedades/llaves de compatibilidad (`time_ms`, `found`, `nodes_expanded`, `steps`) para integración directa con los módulos de Visualización (Módulo 4) y Documentación (Módulo 5).
 
## 7. Visualización en Mapas de Calor y Análisis Comparativo (Módulo 4 — Izquierdo)
Se implementó en `modelado_terreno.ipynb` (Celdas 10–14) la suite visual completa que consume directamente `ESCENARIOS_PRUEBA` y `HISTORIAL_METRICAS`:
1. **Mapas de calor de terreno (`plot_grid_and_path`):** Cuadrícula categórica con escala visual de riesgo accesible (Bloqueo `-1`, Seco $R=0$, Leve $R=1$, Moderado $R=2$, Severo $R=3$) y superposición vectorial de las rutas calculadas, inicio $\mathbf{S}$ y refugio $\mathbf{G}$. Mosaico $2\times 3$ con los escenarios E1 a E6.
2. **Demostración gráfica de la Pregunta Eje (Escenario 5):** Superposición simultánea de la ruta corta física (línea roja discontinua, 9 pasos cruzando agua $R=3$, $J=71.35$) contra el rodeo seco óptimo (línea azul continua, 13 pasos por $R=0$, $J=13.00$). Gráfica de barras adjunta que ilustra el ahorro del **81.8%** en costo acumulado.
3. **Dashboard de métricas (Dijkstra vs $A^*$):** Gráficas de barras comparativas de Nodos Explorados (con etiquetas de reducción de hasta -67.9% en $A^*$), Costos Óptimos idénticos (demostración empírica de admisibilidad) y Tiempos de Ejecución en milisegundos.

