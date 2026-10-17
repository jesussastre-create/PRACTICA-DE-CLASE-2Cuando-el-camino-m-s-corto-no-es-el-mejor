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
