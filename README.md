# Práctica de Clase 2 — *Cuando el camino más corto no es el mejor*

Simulador de rutas de evacuación ante inundaciones. Modela terrenos como una cuadrícula con distintos niveles de riesgo y anegamiento ($R \in \{0,1,2,3\}$ y bloqueos absolutos `-1`) para calcular la ruta **más conveniente** (no necesariamente la más corta) hacia un refugio, comparando **Dijkstra** y **A\*** y adaptando las decisiones del agente ante cambios dinámicos en el terreno.

---

## Estructura del repositorio

| Archivo | Contenido |
|---|---|
| `modelado_terreno.ipynb` | **Entregable principal.** Todo el código ejecutable, los 6 escenarios verificados, mapas de calor, gráficas comparativas, verificador independiente y justificaciones teóricas (Celdas 1–18). |
| `MODELADO.md` | Documentación técnica detallada de los 5 módulos, función de costo, heurística y resultados. |
| `PROJECT_CHARTER.md` | Declaración del problema, pregunta eje, restricciones, equipo y alcance por módulo. |
| `requirements.txt` | Dependencias de Python (`numpy`, `matplotlib`, `jupyter`). |

---

## Organización del Notebook (`modelado_terreno.ipynb`)

| Módulo | Integrante | Celdas | Contenido |
|---|---|---|---|
| **Módulo 1** | Jesús | Celdas 1–3 | `GridEnvironment`, función de costo no lineal $C(u,v)=d(u,v)(1+\alpha R(v)^\beta)$ y pruebas unitarias base. |
| **Módulo 2** | Martín | Celdas 4–6 | Algoritmos `dijkstra` y `a_star`, heurística admisible (Manhattan / Octile), `SearchResult` y pruebas de humo. |
| **Módulo 3** | Andrey Lechuga | Celdas 7–9 | Suite automatizada de los **5 escenarios obligatorios** + **Escenario 6 caza-errores**, invariantes y resumen tabular. |
| **Módulo 4** | Izquierdo | Celdas 10–14 | Mapas de calor categóricos (`plot_grid_and_path`), contraste visual de la Pregunta Eje (E5) y dashboard comparativo. |
| **Módulo 5** | Juan Ramón Gómez Carreón | Celdas 15–18 | Metodología, respuesta formal/empírica a la pregunta eje, verificador independiente (`verificar_ruta_independiente`), análisis crítico y declaración de IA. |

---

## Respuesta a la Pregunta Eje

> *Si dos rutas llegan al mismo refugio, pero una es más corta y la otra atraviesa zonas de mayor riesgo, ¿cómo determina el sistema cuál es la mejor?*

El sistema no compara longitudes en pasos, sino el **costo acumulado** $J(P) = \sum_{(u,v)\in P} d(u,v)\,\bigl(1 + \alpha\,R(v)^\beta\bigr)$ (con $\alpha=2.0, \beta=1.5$), eligiendo la ruta que minimiza $J(P)$:

- **Demostración en el Escenario 5 (`E5_Corto_vs_Conveniente`):** La ruta física más corta tiene **9 pasos** en línea recta pero atraviesa 6 celdas de anegación severa ($R=3$, factor $11.39\times$), acumulando **$J(P) = 71.35$**. El sistema la descarta y elige el rodeo seco de **13 pasos** ($R=0$, factor $1.00\times$) con **$J(P) = 13.00$** (**81.8% menor costo acumulado**).
- **Equilibrio dinámico distancia vs. riesgo (`E4A` vs. `E4B`):** Cuando el corredor central de 9 pasos tiene riesgo leve ($R=1$, $J=13.00$), el sistema lo prefiere frente al rodeo seco de 15 pasos ($J=15.00$). Cuando ese mismo corredor sube a riesgo severo ($R=3$, costo directo $50.57$), el sistema cambia automáticamente al rodeo seco de 15 pasos ($J=15.00$).

---

## Resumen de Resultados Reales Verificados

| Escenario | Costo Óptimo $J(P)$ | Pasos | Nodos (Dijkstra) | Nodos ($A^*$) | Reducción con $A^*$ | Verificación |
|---|---:|---:|---:|---:|---:|---:|
| **E1:** Ruta base disponible ($10\times 10$) | `20.00` | `18` | `89` | `44` | **-50.6%** | OK |
| **E2:** Bloqueo dinámico en `(4,4)` | `26.00` | `18` | `66` | `48` | **-27.3%** | OK |
| **E3:** Refugio inaccesible (canal cerrado) | `inf` | `—` | `40` | `40` | **0.0%** | OK |
| **E4A:** Corredor central con riesgo leve ($R=1$) | `13.00` | `9` | `42` | `14` | **-66.7%** | OK |
| **E4B:** Corredor central con riesgo severo ($R=3$) | `15.00` | `15` | `42` | `24` | **-42.9%** | OK |
| **E5:** Corto ($9$ pasos, $J=71.35$) vs. Conveniente | `13.00` | `13` | `53` | `17` | **-67.9%** | OK |
| **E6:** Caza-errores (vado $R=3$, asimetría y fronteras) | `14.81` | `4` | `10` | `10` | **0.0%** | OK |
| **Total consolidado** | **100% idénticos** | — | **`342`** | **`197`** | **-42.4%** | **14/14 OK** |

---

## Cómo reproducir la solución

Requiere **Python ≥ 3.10**.

```bash
git clone https://github.com/jesussastre-create/PRACTICA-DE-CLASE-2Cuando-el-camino-m-s-corto-no-es-el-mejor.git
cd PRACTICA-DE-CLASE-2Cuando-el-camino-m-s-corto-no-es-el-mejor
python -m venv .venv
# Windows: .venv\Scripts\activate   |   Linux/macOS: source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook modelado_terreno.ipynb
```

Dentro de Jupyter ejecute **`Kernel → Restart & Run All`**. Todas las tablas, gráficas y auditorías se calculan en tiempo real a partir de las ejecuciones del código.

---

## Flujo de trabajo colaborativo en Git

- Desarrollo por ramas dedicadas para cada módulo (`feat/algoritmos-martin`, `feat/escenarios-andrey-lechuga`, `feat/visualizacion-izquierdo`, `docs/documentacion`) e integración mediante Pull Requests hacia `main`.
- Commits descriptivos con convención semántica (`feat(...)`, `fix(...)`, `docs(...)`).
