# Readme del cuaderno de la primera práctica de Visión por Computador

# Realizado por:
                    - Andrés Felipe Vargas Cortés
                    - Miguel Ángel Peñate Alemán

# Realización de las Tareas:

# Crear una imagen con la textura de un tablero de ajedrez

Para esta tarea se utiliza un algoritmo que trabaja con las variables alto y ancho definidas anteriormente, esto se hace
debido a que el algoritmo resultará valido y recreará el mismo patrón aunque las variables se modifiquen, pero deben ser 
iguales para formar un cuadrado.

Básicamente dinuja un cuadrado blanco, salta un cuadrado negro (color de fondo) y vuelve a dibujar un cuadrado, cuando termina la línea empieza adelantado y cuando termina de dibujar la fila vuelve desde el principio. Esta dentro de un bucle 
while con la condición de que una de las variables que controla donde se dibuja el cuadrado llegue a ser igual al alto que
haya sido definido.

// Recursos Utilizados //

https://github.com/otsedom/otsedom.github.io
https://www.mclibre.org/consultar/python/lecciones/python-operaciones-matematicas.html (Para las divisiones enteras)


# Crear una imagen estilo Mondrian con las funciones de Opencv

Para esta tarea, simplemente después de ver algunos ejemplos del estilo Mondrian por la web y comprender como funcionaban las
funciones de Opencv para dibujar lineas y rectángulos realizamos un cuadro de ejemplo.

// Recursos Utiizados //

https://github.com/otsedom/otsedom.github.io
Ejemplos del estilo Mondrian en la web

# Modificar un plano de la imagen

Para esta tarea simplemente se reutilizó el código de ejemplo anterior para la captura de video que se devide en tres planos diferentes R (Para el rojo), G (Para el verde), B(Para el azul), de modo que solamente hacía falta aplicar algunos efectos a 
cada plano, para ello se buscaron todos los filtros disponibles y se aplicaron 3 de ellos a cada plano, como en el siguiente 
ejemplo:  
          r = cv2.convertScaleAbs(r, alpha=2.0, beta=0)

De forma que después cuando se utiliza la función cv2.imshow() se puedan visualizar con los efectos aplicados.

// Recursos utilizados //

https://github.com/otsedom/otsedom.github.io
https://shimat.github.io/opencvsharp_docs/html/6121915d-1174-7345-bdca-789ee1373642.htm

# - Destacar tanto el píxel con el color más claro como con el color más oscuro de una imagen

Para levar a cabo esta tarea era necesario encontrar los pixeles más brillantes y oscuros, para esto se usa una función que 
trasnforma la imagen en escala de grises y mediante el método de cv2.minMaxLoc() encontramos estos pixeles más facilmente,
a continuación se utiliza la función a tiempo real en la imagen y a partir del método cv2.circle() se dibuja un circulo que 
tiene como centro los pixeles detectados como más brillante y más oscuro.

En cuanto a la versión 8x8 pixeles se utiliza una función parecida pero en su lugar se compará con toda la imagen la media de 
brillo de los 8x8 pixeles que se detectan, para después en lugar de dibujar un circulo se deibuja un rectángulo.

// Recursos utilizados //

https://github.com/otsedom/otsedom.github.io
https://shimat.github.io/opencvsharp_docs/html/6121915d-1174-7345-bdca-789ee1373642.htm
https://openai.com/chatgpt/ (Para funciones de búsqueda)

# Hacer una propuesta pop art con la entrada de la cámara web o vídeo

En esta tarea dividimos la imagen capturada por el video en 2 partes, mientras le aplicamos diferentes efectos de forma diferente, en primer la imagen original se basa en una escala de grises con el método cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY),
a continuación se le aplica un efecto de cámara térmico que se aplica con el método cv2.applyColorMap con la opción de mapa de colores "cv2.COLORMAP_INFERNO", mientras que para la segunda mitad de la imagen se utiliza una escala de grises en el canal verde, mientras que en los canales rojo y azul se mantienen a cero, obteniendo un efecto de cámara nocturna.

// Recursos utilizados //
https://github.com/otsedom/otsedom.github.io
https://shimat.github.io/opencvsharp_docs/html/6121915d-1174-7345-bdca-789ee1373642.htm
https://shimat.github.io/opencvsharp_docs/html/0a695799-f4a9-b91e-b49b-df0459c53b9e.htm (Distintos colormaps)

