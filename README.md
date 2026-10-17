# Cuando el camino más corto no es el mejor

Simulador de rutas de evacuación ante inundaciones. Modela el terreno como una cuadrícula con distintos niveles de anegación y calcula la ruta **más conveniente** (no la más corta) hacia un refugio, comparando estrategias de búsqueda y mostrando cómo cambian sus decisiones cuando cambia el terreno.

## Estructura del repositorio

| Archivo | Contenido |
|---|---|
| `modelado_terreno.ipynb` | **Entregable principal.** Todo el código ejecutable, visualizaciones, pruebas, comparación y justificaciones. |
| `MODELADO.md` | Detalle del modelado del terreno y de la función de costo. |
| `PROJECT_CHARTER.md` | Problema, restricciones, responsables y criterios de aceptación. |
| `requirements.txt` | Dependencias de Python. |

## Secciones del notebook

1. Introducción, metodología y respuesta a la pregunta eje
2. Modelado del terreno (`GridEnvironment`) y función de costo
3. Algoritmos de búsqueda (Dijkstra y A\*) con métricas
4. Visualización (mapas de calor, rutas, gráficas comparativas)
5. Escenarios de prueba (5 obligatorios + 1 caza-errores)
6. Verificación independiente de rutas y tabla comparativa con ejecuciones reales
7. Declaración de uso de IA y reproducibilidad

## Pregunta eje

> Si dos rutas llegan al mismo refugio, pero una es más corta y la otra atraviesa zonas de mayor riesgo, ¿cómo determina el sistema cuál es la mejor?

El sistema compara el **costo acumulado** `J(P) = Σ d(u,v)·(1 + α·R(v)^β)` de cada ruta y elige el menor. Las celdas con agua multiplican el costo de entrar en ellas, así que un rodeo seco más largo puede ser más barato que un atajo inundado. Las celdas bloqueadas (`-1`) tienen costo infinito y nunca forman parte de una ruta. Justificación completa en la sección 1.3 del notebook; demostración empírica en el Escenario 5.

## Cómo reproducir

Requiere Python ≥ 3.10.

```bash
git clone <URL-del-repo>
cd PRACTICA-DE-CLASE-2Cuando-el-camino-m-s-corto-no-es-el-mejor
python -m venv .venv
# Windows: .venv\Scripts\activate    |  Linux/macOS: source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook modelado_terreno.ipynb
```

En Jupyter: **Kernel → Restart & Run All**. Todas las métricas de la tabla comparativa se generan en esa ejecución; ninguna cifra está escrita a mano.

## Equipo

| Integrante | Responsabilidad |
|---|---|
| Jesús | Modelado del terreno y función de costo |
| Martín | Algoritmos de búsqueda y métricas |
| Lechuga | Escenarios de prueba |
| Izquierdo | Visualización |
| Ramón | Documentación, verificación, declaración de IA y control del repositorio |

## Flujo de trabajo con Git

- Una rama por integrante/sección: `feat/modelado`, `feat/busqueda`, `feat/visualizacion`, `feat/escenarios`, `docs/documentacion`.
- Integración a `main` mediante Pull Request revisado por otro integrante.
- Para evitar conflictos en el `.ipynb`, cada quien edita **sólo las celdas de su sección**; antes de abrir el PR: `git pull origin main`, *Restart & Run All*, y luego commit.
- Mensajes de commit descriptivos con prefijo: `feat:`, `fix:`, `test:`, `docs:`, `refactor:`.
  - Ej.: `feat(busqueda): A* con heurística Manhattan y contador de nodos expandidos`
