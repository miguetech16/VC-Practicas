# Readme Práctica 4. Reconocimiento de matrículas

### Realizado por:
                    - Andrés Felipe Vargas Cortés
                    - Miguel Ángel Peñate Alemán

### Contenidos 

## Descripción de la Práctica 

El objetivo de esta práctica, consiste en desarrollar un prototipo que procese un ([vídeo de ejemplo proporcionado](https://alumnosulpgc-my.sharepoint.com/:v:/g/personal/mcastrillon_iusiani_ulpgc_es/EXRsnr4YuQ9CrhcekTPAD8YBMHgn16KwlunFg32iZM0xVQ?e=kzuw4l)) o varios vídeos (incluyendo vídeos de cosecha propia), en los que se recoja cierta información sobre lo que ocurre, haciendo un seguimiento fotograma a fotograma y cumpla con los siguientes objetivos:

- detecte y siga las personas y vehículos presentes
- detecte y lea las matrículas de los vehículos presentes
- cuente el total de cada clase
- vuelque a disco un vídeo que visualice los resultados
- genere un archivo csv con el resultado de la detección y seguimiento. Se sugiere un formato con al menos los siguientes campos:

```
fotograma, tipo_objeto, confianza, identificador_tracking, x1, y1, x2, y2, matrícula_en_su_caso, confianza, mx1,my1,mx2,my2, texto_matricula
```

La **entrega del cuaderno o cuadernos** se hace efectiva a través del campus virtual por medio de un **enlace github**. Además del **archivo README**, debe incluirse el resultado del vídeo proporcionado como test (o enlace al mismo), y el correspondiente archivo *csv*. En el caso de entrenarse algún detector, por ejemplo de matrículas, debe proporcionarse acceso al conjunto de datos.

## Desarrollo de la Práctica

### Preparación del Entorno

Para el procesamiento de detección de los diferentes objetivos dentro de un video se hará uso de un modelo de partida basado en YOLO proporcionado por [Ultralytics](https://github.com/ultralytics/ultralytics), este primer modelo se encargará de detectar personas, vehículos, motocicletas, etc. Por otro lado se ha entrenado un segundo modelo de YOLO para la detección de matrículas, el resultado de esta detcción será procesado por un OCR (Reconocimiento óptico de carácteres) en este caso easyOCR.

Por temas de compatibilidad se recomienda crear un nuevo *enviroment* e instalar algunas utilidades usando los siguientes comandos desde Anaconda Prompt:

```
conda create --name VC_P4 python=3.9.5
conda activate VC_P4
pip install ultralytics
pip install lapx
pip install easyocr
```

### Obtención y Clasificación del Dataset

Una vez el equipo se encuentra con disponibilidad y todas las librerias han sido instaladas, procedimos a generar nuestro propio Dataset, para ello simplemente obtuvimos diversas imágenes de matrículas de todos los tipos que tuvimos a manos, es decir de cosecha propia con un total de 103 figuras distintas. 

A continuación, utilizamos Labelme para generar un conjunto anotaciones por cada imágen, con el objetivo de señalizar a nuestro modelo donde se encuentran las matrículas dentro de cada imagen.

Para el uso de Labelme se pueden usar las siguientes instrucciones desde la consola de Anaconda:

```
conda activate labelme
pip install labelme
```
En caso de algún error es posible usar otro *enviroment*, debido a que solo se usa para la generación de anotaciones.

 Posteriormente, utilizamos Labelme2YOLO se convirtieron los datos al formato requerido para entrenar nuestro modelo, el cual necesita acceder anotaciones con un formato como:
 
 ```
<object-class-id> <x> <y> <width> <height> 
"Identificador de clase" - "Coordenada x del centro" - "Coordeanada y del centro" - "Anchura" - "Altura"
```

 El conjunto resultante formado por imágenes y anotaciones fue dividido en tres grupos: entrenamiento (train), prueba (test) y validación (val), se pueden encontrar en la carpeta YOLODataset.


### Entrenamiento mediante GPU

Dado que no es conveniente que el ordenador esté funcionando exclusivamente para este propósito durante varias horas, lo mejor es aprovechar la GPU para llevar a cabo el entrenamiento. El uso de la gráfica puede reducir drásticamente el tiempo de entrenamiento siendo más óptima para nuestro próposito. 

Será necesario instalar CUDA y asegurarnos de que la versión instalada sea compatible con nuestro entorno, para lo cual sin necesidad de realizar pasos adicionales, podemos ejecutar el siguiente comando en Anaconda Prompt:

```
conda search cudatoolkit
```

![conda_search](images/Captura.PNG)

Como podemos ver, la versión compatible más alta es la 11.8.0, una vez descargado e instalado desde [CUDA](https://developer.nvidia.com/cuda-toolkit-archive), debemos realizar el siguiente comando en el anaconda prompt para instalar pytorch dentro del environment en el que queremos utilizar CUDA:

```
conda install pytorch torchvision torchaudio pytorch-cuda=11.8 -c pytorch -c nvidia
```

Una vez instalado, podemos verificar que funciona mediante el siguiente comando de pytorch:

```
import torch
print(torch.cuda.is_available())
```

En caso de que la gráfica no haya sido reconocida se puede descargar e instalar la librería [cuDNN](https://developer.nvidia.com/rdp/cudnn-archive).

### Entrenamieno del Modelo

Se procede al entrenamiento del modelo después de tener todo instalado y en orden, simplemente bastan las siguientes lineas, se define el modelo con la versión 11, se entrena el modelo al que le adjunta un fichero donde figuran las rutas a los datos que necesita, además se definen algunos datos de interés como las épocas, la paciencia o los lotes.

```
model = YOLO("yolo11n.yaml")
results = model.train(data='C:/Users/varga/Documents/GitHub/VC-Practicas/P4/YOLODataset/dataset.yaml', 
                        epochs=300, 
                        imgsz=512, 
                        plots=True,
                        patience= 15,
                        batch=-1,
                        workers=12,
                        device=0,
                        )  
```
Una vez completado el entrenamiento, los pesos se almacenan en la carpeta trainX/weights/best.pt, junto con otros datos relacionados con el proceso de entrenamiento.

En esta tarea, hemos utilizado dos modelos de YOLO11n: el estándar y uno que hemos entrenado nosotros mismos. A continuación, se describe el proceso para entrenar nuestro modelo:

### Procesamiento de Video

Una vez hemos obtenido el modelo entrenado para capturar las matrículas de los vehículos se puede proceder al procesamiento del video, donde se pondrá a prueba el primer modelo pre-entrenado, el modelo que nosotros hemos entrenado y el OCR.

Para ello se definen los modelos, se indica el idioma español para el OCR y una función para la detección de matrículas y su lectura:

```
# Definir la función encontrar_matricula
def encontrar_matricula(car_img):
    results = customModel(car_img)
    for r in results:
        boxes = r.boxes
        for box in boxes:
            x1, y1, x2, y2 = box.xyxy[0]
            x1, y1, x2, y2 = int(x1), int(y1), int(x2), int(y2)
            cv2.rectangle(car_img, (x1, y1), (x2, y2), (0, 255, 0), 2)
            matricula_img = car_img[y1:y2, x1:x2]

            # Leer matrícula usando OCR
            matricula_resultados = reader.readtext(matricula_img, allowlist="ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789-.")
            if matricula_resultados:
                matricula_text = matricula_resultados[0][-2]
                conf_matricula = matricula_resultados[0][-1]  # Confianza de OCR
                return matricula_text, conf_matricula, x1, y1, x2, y2
    return "", 0, 0, 0, 0, 0  # Retornar valores vacíos si no se detecta matrícula
```
La función anterior es utilizada en el caso de que el primer modelo que procesará el video fotograma a fotograma, encuentre un objeto y lo relacione con alguna de las siguientes clases:

```
"car", "motorbike", "bus", "train", "truck" 
```

En todo momento se irá registrando la información recopilada del video dentro de una lista, donde se guardarán los siguientes datos:

```
resultados.append({
                    'fotograma': cap.get(cv2.CAP_PROP_POS_FRAMES),
                    'tipo_objeto': tipo_objeto,
                    'confianza': conf,
                    'identificador_tracking': track_id,
                    'x1': x1, 'y1': y1, 'x2': x2, 'y2': y2,
                    'matrícula_en_su_caso': texto_placa,
                    'confianza_matricula': conf_matricula,
                    'mx1': mx1, 'my1': my1, 'mx2': mx2, 'my2': my2,
                    'texto_matricula': texto_placa
                })
```

Estos datos serán volcados en un archivo .csv, así como el resultado visual del video procesado.

## Resultados

En este apartado se muestran los resultados obtenidos:

#### Vídeo Generado 1 (Demo cuaderno):  

 [Video_cuaderno](https://alumnosulpgc-my.sharepoint.com/:v:/g/personal/andres_vargas101_alu_ulpgc_es/EVRNr1zRNXxGp-j5GjCCnlMBVjz2_UT28lowG5pjy3pDKw?nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJPbmVEcml2ZUZvckJ1c2luZXNzIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXciLCJyZWZlcnJhbFZpZXciOiJNeUZpbGVzTGlua0NvcHkifX0&e=OeIrOn)


 ![Video_cuaderno](/P4/images/Video1.gif)


#### Vídeo Generado 2 (Video propio): 

[Video_propio](https://alumnosulpgc-my.sharepoint.com/:v:/g/personal/andres_vargas101_alu_ulpgc_es/EcoFLbqUDbJCnvPlZ6fqaSMBePY2CUVLiTN5sCk7ELET6g?nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJPbmVEcml2ZUZvckJ1c2luZXNzIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXciLCJyZWZlcnJhbFZpZXciOiJNeUZpbGVzTGlua0NvcHkifX0&e=KLwmei)

![Video_propio](/P4/images/Video2.gif)

#### Archivos CSV: 

Se genera un archivo CSV que contiene los resultados del procesamiento. 

Objetivos Requeridos Cumplidos:

- detecte y siga las personas y vehículos presentes
- detecte y lea las matrículas de los vehículos presentes
- cuente el total de cada clase
- vuelque a disco un vídeo que visualice los resultados
- genere un archivo csv con el resultado de la detección y seguimiento

Extras:

- Anonimizar a las personas y matrículas de vehículos presentes en un vídeo.
