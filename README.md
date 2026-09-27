# Reproducing Images con PyGAD

Reproducción de una imagen mediante un **algoritmo genético**, implementado en Python con
[PyGAD](https://pygad.readthedocs.io/) y el módulo auxiliar
[GARI](https://github.com/ahmedfgad/GARI) (*Genetic Algorithm for Reproducing Images*).

El algoritmo parte de una población de soluciones completamente aleatorias y la evolve
generación tras generación hasta aproximarse, de forma progresiva, a una imagen objetivo de
100 × 100 píxeles.

## Contexto académico

| Dato | Valor |
| --- | --- |
| Institución | Universidad Técnica de Machala |
| Facultad | Facultad de Ingeniería Civil |
| Carrera | Tecnologías de la Información |
| Asignatura | Inteligencia Artificial |
| Curso | Décimo semestre, paralelo "A" |
| Periodo lectivo | 2026 - 2 |
| Estudiante | Mario Sebastián Ávila León |
| Tutor | Ing. Sist. Rivas Asanza Wilmer Braulio, Phd. |

El informe académico completo que documenta el marco teórico, la metodología, los
experimentos, el análisis de tiempos y las conclusiones se encuentra en
[`Informe_Reproduccion_Imagenes_PyGAD.pdf`](Informe_Reproduccion_Imagenes_PyGAD.pdf) (67 páginas).

## Estructura del proyecto

```
.
├── pygad_imagen.ipynb                     # Cuaderno principal (punto de entrada)
├── gari.py                                # Módulo auxiliar: imagen <-> cromosoma
├── fruit.jpg                              # Imagen objetivo (100x100, RGB, 24 bits)
├── tiempos.txt                            # Tiempos de ejecución de los 6 experimentos
├── Informe_Reproduccion_Imagenes_PyGAD.pdf
└── *.png                                  # Resultados de los experimentos (ver convención)
```

## Requisitos

- Python 3.10 o superior
- `numpy`, `imageio`, `matplotlib`, `pygad`

El cuaderno fue desarrollado en Google Colab, donde la primera celda instala la dependencia
principal mediante `!pip install pygad`. Para ejecutarlo en un entorno local:

```bash
pip install -r requirements.txt
jupyter lab pygad_imagen.ipynb
```

## Cómo ejecutar

1. `fruit.jpg` y `gari.py` deben estar en el mismo directorio de trabajo que el cuaderno.
2. Ejecutar las celdas **en orden**. La función de aptitud `fitness_fun` closurea la variable
   global `target_chromosome`, por lo que depende de que la celda que la define se haya
   ejecutado después de la que crea esa variable.
3. Para replicar un experimento concreto, modificar `num_generations` en la celda que crea
   `ga_instance` antes de ejecutar `ga_instance.run()`.

Flujo del experimento:

1. `imageio.imread('fruit.jpg')` → arreglo NumPy de forma `(100, 100, 3)`.
2. Normalización a flotante en `[0, 1]` dividiendo entre 255.
3. `gari.img2chromosome(target_im)` → cromosoma objetivo de **30 000 genes** (100 × 100 × 3).
4. Definición de `fitness_fun`.
5. Instanciación de `pygad.GA` y ejecución con `ga_instance.run()`.
6. Recuperación de la mejor solución con `ga_instance.best_solution()`.
7. Reconstrucción con `gari.chromosome2img(solution, target_im.shape)` y visualización.

## Configuración del algoritmo genético

Parámetros especificados explícitamente en el cuaderno:

| Parámetro | Valor | Función |
| --- | --- | --- |
| `num_generations` | 50000 | Ciclos de evolución ejecutados |
| `num_parents_mating` | 10 | Padres seleccionados por generación |
| `fitness_func` | `fitness_fun` | Función de aptitud personalizada |
| `sol_per_pop` | 20 | Individuos por población |
| `num_genes` | `target_im.size` (30 000) | Genes por cromosoma |
| `init_range_low` / `init_range_high` | 0.0 / 1.0 | Rango de la población inicial |
| `mutation_percent_genes` | 0.01 | Porcentaje de genes mutados (~3 de 30 000) |
| `mutation_type` | `"random"` | Mutación aleatoria por reemplazo |
| `random_mutation_min_val` / `max_val` | 0.0 / 1.0 | Rango de los genes mutados |

Los operadores de selección y cruce no se especifican, por lo que se usan los valores por
defecto de PyGAD: selección por estado estacionario y cruce `single_point`.

### Función de aptitud

```python
def fitness_fun(ga_instance, solution, solution_idx):
    fitness = numpy.sum(numpy.abs(target_chromosome - solution))

    # Negating the fitness value to make it increasing rather than decreasing.
    fitness = numpy.sum(target_chromosome) - fitness
    return fitness
```

La suma de diferencias absolutas es una métrica de **error** (menor es mejor). Como PyGAD
maximiza por defecto, el error se transforma en una aptitud creciente restándolo de la suma
total del cromosoma objetivo.

### Módulo GARI

`gari.py` aporta las dos funciones que conectan el dominio de la imagen con el del algoritmo
genético:

- `img2chromosome(img_arr)`: remodela la imagen a un vector 1D de longitud `prod(shape)`.
- `chromosome2img(vector, shape)`: operação inversa, validando que `len(vector)` coincida con
  `prod(shape)` y lanzando `ValueError` en caso contrario.

## Experimentos y resultados

Seis ejecuciones independientes con número creciente de generaciones:

| Experimento | Generaciones | Tiempo | Evolución visual |
| --- | --- | --- | --- |
| 1 | 1 | 20 s | Ruido de color puro, sin estructura reconocible |
| 2 | 100 | 21 s | Sin cambios perceptibles respecto del experimento 1 |
| 3 | 1 000 | 2 min | Sigue dominando el ruido; la gráfica de aptitud ya es coherente |
| 4 | 10 000 | 17 min | Aparece la silueta redondeada de la fruta |
| 5 | 25 000 | 46 min | Contorno más definido; aparecen tonalidades variedas |
| 6 | 50 000 | 1 h 30 min | Mejor aproximación visual; el fondo aún conserva ruido |

- El tiempo de ejecución creció 270 veces (20 s → 5 400 s) frente a un factor de 50 000 en el
  número de generaciones: existe una sobrecarga fija de inicialización y, a partir de 1 000
  generaciones, el costo por generación se estabiliza entre 0,102 y 0,120 s.
- La transición cualitativa más relevante ocurre entre 1 000 y 10 000 generaciones. Los
  incrementos posteriores producen mejoras de menor magnitud relativa (rendimientos decrecientes).
- En el experimento de 50 000 generaciones: aptitud de la mejor solución `12707.325182785838`,
  alcanzada en la generación 49 993.

### Convención de nombres de las imágenes

| Archivo | Contenido |
| --- | --- |
| `1.png`, `100.png`, `1000.png`, `10000.png`, `25000.png`, `50000.png` | Gráfica de aptitud del experimento |
| `11.png`, `101.png`, `1001.png`, `10001.png`, `25001.png`, `50001.png` | Imagen reconstruida del experimento |
| `fruit.jpg` | Imagen original de referencia |

Es decir, para un experimento de *N* generaciones: `N.png` es la gráfica y `N+1.png` es la
imagen reconstruida.

## Notas sobre el proyecto

- **Procedencia de la imagen.** El enunciado de la práctica solicita una imagen `fruit.png`
  procedente del repositorio Fruits-360 100x100. El archivo realmente incluido es `fruit.jpg`,
  que es la imagen de referencia del propio proyecto GARI y se usa de forma idéntica en el
  tutorial "Reproducing Images" de la documentación de PyGAD. No existe evidencia en los
  archivos del proyecto de que provenga del conjunto Fruits-360. La referencia al dataset se
  mantiene en el informe por requerimiento del enunciado, dejando constancia de esta
  distinción.
- **Tiempos medidos manualmente.** `tiempos.txt` fue escrito a mano; el cuaderno no mide el
  tiempo de ejecución (no usa el módulo `time`).
- **Gráficas de los experimentos 1 y 100.** Sus ejes horizontales abarcan de 0 a 1, lo que no
  es coherente con el número de generaciones configurado. Documentado en el informe.
- **Código redundante en el cuaderno.** Una celda redefine localmente `img2chromosome` y
  `chromosome2img` sin usarlas después; las funciones efectivas son las importadas de `gari`.
  La redundancia está documentada en el informe y no altera el resultado.
- **Sin métricas de similitud.** Los experimentos no calculan métricas objetivas (PSNR,
  SSIM) ni imprimen el valor de aptitud de cada corrida, por lo que el análisis de
  convergencia del informe se apoya en inspección visual.

## Referencias

- PyGAD — documentación: <https://pygad.readthedocs.io/en/stable/pygad.html>
- PyGAD — tutorial "Reproducing images": <https://pygad.readthedocs.io/en/stable/pygad_example_reproducing_images.html>
- GARI — <https://github.com/ahmedfgad/GARI>
- Fruits-360 100x100 — <https://github.com/fruits-360/fruits-360-100x100>
