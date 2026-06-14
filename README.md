# README - Notebooks del TFM

Este proyecto contiene tres notebooks principales para preparar los datos, entrenar modelos de segmentación y exportar predicciones a QuPath.

---

## 1. Generación de patches desde OME-TIFF + GeoJSON

Este notebook realiza el preprocesado de los casos. No entrena modelos.

A partir de las imágenes histológicas y sus anotaciones de QuPath, genera patches de imagen y sus máscaras semánticas correspondientes.

**Input necesario:**

```text
TFM/
├── imagenes/   → imágenes .ome.tif, .ome.tiff, .tif o .tiff
└── geojsons/   → anotaciones .geojson exportadas desde QuPath
```

**Qué hace:**

* Empareja cada imagen con su GeoJSON.
* Lee las anotaciones de QuPath.
* Genera máscaras con las clases:

  * 0: background
  * 1: no tumor
  * 2: tumor
  * 255: ignore/no anotado
* Extrae ROIs y patches.
* Divide los casos en train, validation y test.
* Guarda los patches en carpetas separadas.

**Output generado:**

```text
TFM_UNI_PATCHES_224/
├── train/
├── val/
├── test/
├── metadata.csv
└── split_case_ids.json
```

Si se usa `PATCH_SIZE = 256`, la salida será:

```text
TFM_UNI_PATCHES_256/
```

---

## 2. Selección de mejor modelo desde patches

Este notebook entrena y compara diferentes modelos de segmentación usando los patches generados previamente.

**Input necesario:**

```text
TFM_UNI_PATCHES_224/
TFM_UNI_PATCHES_256/
```

**Qué hace:**

* Carga los patches de train, validation y test.
* Entrena varias arquitecturas con el mismo split.
* Compara modelos como U-Net, DeepLabV3 y UNI + decoder.
* Guarda el mejor checkpoint según la métrica principal:

```text
0.5 * Dice Tumor + 0.5 * Dice no tumor
```

Esta métrica se usa para seleccionar el mejor modelo porque da prioridad a las dos clases relevantes del problema: tumor y no tumor.

**Output generado:**

```text
resultados_tfm_model_selection/
├── checkpoints/
├── csv/
└── figures/
```

El resultado principal es el checkpoint del mejor modelo, que después se usa para inferencia y exportación.

---

## 3. Pipeline final DeepLabV3-ResNet50

Este notebook utiliza el modelo final seleccionado, en este caso DeepLabV3 con encoder ResNet50 preentrenado en ImageNet.

Permite tres modos de ejecución:

```python
RUN_MODE = "train"
RUN_MODE = "export"
RUN_MODE = "predict_full"
```

### Modo `train`

Sirve para reentrenar el modelo final o añadir nuevos casos al entrenamiento.

**Input necesario:**

```text
TFM/imagenes/
TFM/geojsons/
```

**Output:**

```text
resultados_deeplab_final/checkpoints/
```

---

### Modo `export`

Sirve para cargar un checkpoint, evaluar un caso anotado y exportar la predicción a GeoJSON compatible con QuPath.

**Input necesario:**

```text
checkpoint entrenado
imagen original
GeoJSON de anotación
```

**Output:**

```text
resultados_deeplab_final/geojson_qupath/
```

---

### Modo `predict_full`

Sirve para aplicar el modelo a una imagen completa nueva, aunque no tenga anotación.

**Input necesario:**

```text
checkpoint entrenado
imagen completa .ome.tif/.tif
```

**Output:**

```text
GeoJSON compatible con QuPath
figuras de comprobación visual
predicción sobre imagen completa
```

---

## Orden recomendado

```text
1. Generar patches
2. Seleccionar el mejor modelo
3. Usar el modelo final para entrenar, exportar o predecir imagen completa
```

En resumen:

```text
OME-TIFF + GeoJSON
        ↓
Patches + máscaras
        ↓
Entrenamiento y comparación de modelos
        ↓
Modelo final
        ↓
Predicción y exportación a QuPath
```
