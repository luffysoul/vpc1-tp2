# TP2 — Visión por Computadora I (CEIA, FIUBA)

Detector de máximo enfoque sobre video mediante análisis espectral. Todo el trabajo está en el
notebook `TP2_VPC1.ipynb`, que ya tiene todas las salidas ejecutadas.

Se implementa la métrica de nitidez propuesta en:

> Kanjar De, V. Masilamani (2013). *Image Sharpness Measure for Blurred Images in Frequency Domain.*
> Procedia Engineering 64, pp. 149–158.

## Contenido

1. **Implementación de la métrica FM** — los seis pasos de la sección 3.1 del paper,
   `FM = TH / (M·N)`.
2. **Validación en tres frentes** — FM decrece monótonamente ante desenfoque gaussiano creciente
   (figura 5a del paper); la ecuación (1) se confirma contra el ejemplo numérico de *Lena* que el
   propio paper publica; y se establece un límite del método: **FM no es comparable entre regiones
   de distinto tamaño**.
3. **Experimento 1** — medición sobre el frame completo.
4. **Experimento 2** — medición sobre una ROI centrada del 5% y del 10% del área del frame.
5. **Detección automática** de la región de máximo enfoque, marcada en color distinto sobre las
   curvas, con análisis de sensibilidad al umbral.
6. **Comparación** de ambos experimentos, con un control que separa el efecto del recorte del efecto
   del tamaño, y la medición de regiones de fondo que explica el mecanismo real.
7. **Sensibilidad al preprocesamiento** — leakage espectral y ventaneo de Hann (excede la consigna).
8. ⭐ **Puntos extra** — *unsharp masking* en punto flotante, barrido 2-D de σ y α con **selección
   automática de los parámetros**, medición de halos, y escritura del video realzado a disco.

## Resultados principales

| Medición | Rango dinámico | Frame pico | Región de máximo enfoque |
|---|---|---|---|
| Frame completo | 4,20x | 109 | frames 79–115 (1,23 s) |
| ROI 10% | 8,37x | 111 | frames 100–113 (0,47 s) |
| ROI 5% | 8,50x | 111 | frames 92–113 (0,73 s) |

**El pico es robusto; el ancho de la región no lo es.** El frame de máximo enfoque no cambia para
umbrales del 80% al 99%, y coincide con el `argmax` crudo. En cambio el borde inferior de la región
está mal condicionado en el caso de la ROI 10%, porque la curva tiene un hombro donde la pendiente es
cuatro veces menor que en el flanco de bajada. Se documenta en la sección 7 en lugar de ocultarse.

**Medir sobre una ROI central es superior a medir sobre el cuadro completo, y no por el motivo
obvio.** Un control a igual número de píxeles (frame completo remuestreado a 202×114: 3,67x contra
8,37x de la ROI) muestra que la ventaja viene del recorte y no del tamaño. Y medir regiones de fondo
del mismo tamaño revela que **el fondo no aporta un piso constante: entra en foco en otro instante**
(picos en los frames 69 y 118, contra 111 del sujeto). El video es un barrido focal y la curva del
cuadro completo es la superposición de picos de distintas profundidades — por eso su meseta es más
ancha y su pico aparece corrido.

**Unsharp masking:** los parámetros no se eligen a mano — el barrido de 21 combinaciones los asigna
programáticamente. Se selecciona **σ=0,8 y α=3,0** por máximo rango dinámico (15,2x), y la región
detectada se ensancha de 0,47 s a 0,83 s. El barrido registra además los halos, lo que permite
mostrar el **frente de Pareto** entre discriminación y calidad visual: la combinación seleccionada
no es la de menos halos (4,1%), y una alternativa conservadora (σ=0,8, α=2,0) conserva el 93% del
rango dinámico con el 57% de los halos. Se prioriza el rango dinámico porque acá el realce es
instrumental para medir, no para producir un video de consumo visual. El óptimo en α se explica por
la saturación de FM contra su techo de 1, no por amplificación de ruido, y el pico se mantiene en el
frame 111 en las 21 combinaciones probadas.

**Sobre el tipo de dato:** medir el realce en `uint8` introduce un sesgo que depende de la región —
por debajo del 0,6% en la ROI, pero hasta −17,2% en el frame completo—, así que todo el realce se
calcula en `float64` y se convierte a `uint8` solo para visualizar.

## Integrante

- Jaime Pinzón

## Estructura

```
TP2/
├── TP2_VPC1.ipynb
├── video/
│   └── focus_video.mov
├── salida/
│   └── focus_video_realzado.mp4   (generado por el notebook)
├── requirements.txt
└── README.md
```

## Cómo correrlo

Se usó Python 3.10. El notebook lee el video con ruta relativa, así que hay que abrirlo desde esta
carpeta.

```bash
pip install -r requirements.txt
jupyter notebook TP2_VPC1.ipynb
# o, para re-ejecutarlo completo sin abrirlo:
jupyter nbconvert --to notebook --execute --inplace TP2_VPC1.ipynb
```
