# AGENTS.md

Curso de Inteligencia Artificial (UTMACH,ITI, 10.º semestre). Documentación y código en
**español**; los comentarios y docstrings del código están en **inglés**. Mantener ese split.

## Qué es este repo

Práctica de reproducción de imágenes con algoritmo genético. El punto de entrada es
`pygad_imagen.ipynb`; `gari.py` solo aporta `img2chromosome` / `chromosome2img`. No hay build,
tests, linter ni CI: la única verificación posible es ejecutar el cuaderno.

`Informe_Reproduccion_Imagenes_PyGAD.pdf` (67 págs.) es el entregable académico y la fuente de
verdad documental. Sus cifras (parámetros del AG, tiempos, descripción de los experimentos)
deben coincidir con el cuaderno: si cambias algo del AG, actualiza también el informe/README.

## Cosas que un agente no deduciría solo

- **Los PNG numerados no los genera el cuaderno.** No hay bucle, `on_generation` callback ni
  `savefig` en el notebook. Convención real (verificada en el informe, Anexo E): para un
  experimento de *N* generaciones, `N.png` es la gráfica de aptitud y `N+1.png` es la imagen
  reconstruida, en los checkpoints {1, 100, 1000, 10000, 25000, 50000}. Para regenerarlos hay
  que escribir ese callback.
- **El cuaderno está pensado para Colab.** La primera celda es `!pip install pygad`, una
  *magick* que revienta fuera de Jupyter. En el Python del sistema (3.13.7) solo está `numpy`;
  `pygad`, `imageio` y `matplotlib` no están instalados. Para correr local: `pip install -r
  requirements.txt` y luego `jupyter lab pygad_imagen.ipynb`.
- **Los tiempos son caro real, no un detalle.** 50 000 generaciones ≈ 1 h 30 min; 10 000 ≈ 17
  min. Para un smoke test baja `num_generations` a ~10 antes de ejecutar `run()`. No lances la
  configuración por defecto para "comprobar que funciona".
- **El orden de las celdas importa.** `fitness_fun` lee la global `target_chromosome`, definida
  en una celda anterior. Reordenar o ejecutarla por partes rompe el fitness en silencio (no da
  error). Si cambias la imagen objetivo, re-ejecuta desde la celda que la carga.
- **El fitness es un error con signo invertido, a propósito.** `sum(target) - sum(abs(target -
  solution))`, porque PyGAD maximiza por defecto. No lo "corrijas" a minimización.
- **`num_genes = target_im.size` = 30 000**, porque `fruit.jpg` es 100×100 RGB → `(100, 100, 3)`.
  No hardcodear 10 000 (eso serían los píxeles sin canales).
- **La celda que redefinem `img2chromosome`/`chromosome2img` es código muerto, y eso es
  intencional:** el informe lo documenta como redundancia (§3.4 y §7.5). No la borres en
  silencio; si la quitas, actualiza el informe.
- **El PDF no es legible sin herramienta.** No hay extractor de PDF en el entorno. Para leer su
  texto: `pip install pypdf --target <dir-temporal>` y usar `PdfReader`, con `PYTHONIOENCODING`
  en UTF-8 (el texto contiene acentos y `cp1252` aborta la impresión).
- **Inconsistencias ya documentadas, no son bugs pendientes de arreglar:** `fruit.jpg` es la
  imagen de referencia de GARI, no una foto de Fruits-360 (el enunciado pedía `fruit.png`);
  `1.png` y `100.png` tienen eje horizontal 0–1; `tiempos.txt` está medido a mano (el cuaderno no
  usa `time`). Si "arreglas" alguna, cambia la sección 8.1 / 9 / 11.3 del informe.
- **El cuaderno guarda outputs con PNG en base64** (~160 KB del archivo son imágenes de las
  gráficas). Es intencional: el informe las analiza. No hacer `Clear outputs` sin avisar.

## Git

El repo remoto es `https://github.com/MarioAvilaSL/Reproducing-Images-Pygad.git`. No hacer
`git init`/commit/push sin que lo pidan explícitamente.
