# ROBÓTICA DE SERVICIOS

# Indice
* [Indice][ind]
* [Welcome][wel]
* [Localized Vacuum Cleaner][p1]
* [Rescue People](#Rescue-people)
* [Autoparking](#Autoparking)
* [Amazon Warehouse](#Amazon-warehouse)
* [Marker Visual Localization](#Marker-visual-localization)



[ind]: https://github.com/acruzr2021/robotica_servicios/blob/main/README.md#indice
[wel]: https://github.com/acruzr2021/robotica_servicios/blob/main/README.md#welcome
[p1]:  https://github.com/acruzr2021/robotica_servicios/blob/main/README.md#localized-vacuum-cleaner

---

# Welcome

Greetings, esteemed readers, and a warm welcome to my blog. My name is Alba Cruz Rodríguez and this is my website for the service robotics subject. These space will be dedicated to my practices, their development, the differents ideas and these results until we find the solution.

Robotics is a discipline we seek to solve some problems to the humanity joining sensors, actuators and software on a machine. In this doctrine studies the systems that make a connection intelligent between the perceptual and acting system. We will focus on the "brain" of the machine (or the *software*), devising the most effective and efficient algorithm for each project.

Join me to discover the exciting world of robotics and the discipline of *Service Robotics*! 

---

# Localized Vacuum Cleaner

Este proyecto trata sobre la creación de un sistema de limpieza autónomo con un robot, cuyo objetivo es limpiar la mayor cantidad de superficie en el menor tiempo posible, utilizando un robot que puede localizarse en el mapa de la casa. La solución se estructura en tres fases: registro, planificación y ejecución.

## Registro

### Determinación tamaño de las celdillas

Para comenzar, decidí cuál sería el tamaño óptimo de las cuadrículas en las que el robot operaría. Después de medir el robot, se concluyó que el tamaño adecuado sería 15x15 píxeles. Esto garantizaría que el robot cubriera toda la superficie sin dejar zonas sin limpiar ni pasar muchas veces por el mismo lugar.

![image](https://github.com/user-attachments/assets/973843da-028e-4cd1-8df4-f35962201f12)

> **Nota:** Posteriormente, este tamaño fue ajustado a **33x33 píxeles** debido a la diferencia de proporciones en el nuevo mapa proporcionado por **Unibotics**, el cual requería este nuevo ajuste para un rendimiento óptimo.

### Cálculo de la escala y transformaciones 3D a 2D

Para alinear las posiciones del robot en el entorno simulado con las coordenadas del mapa, tomé varios puntos de referencia en Gazebo y los convertí a píxeles mediante una regla de tres. A partir de estos puntos, obtuve la escala de transformación. La matriz de transformación entre el espacio 3D (Gazebo) y 2D (el mapa) tiene una rotación de -90º y una traslación de 4.06 metros en el eje X y 5.38 metros en el eje Y, quedando la matriz de la siguiente forma:

<div align="center">

| 0 | 1 | 0 | 4.06 |
|---|---|---|---|
| -1 | 0 | 0 | 5.38 |
| 0 | 0 | 1 |  0  |
| 0 | 0 | 0 |  1  |

</div>

### Verificación de posiciones

Para asegurar que las coordenadas del robot en el mapa coincidieran correctamente con las posiciones reales en Gazebo, desplazamos el robot por los cuatro cuadrantes respecto al eje de Gazebo, y dibujamos las posiciones en las cuadrículas del mapa. Esto permitió verificar que los cálculos de transformación eran correctos. En el siguiente vídeo se puede observar cómo los puntos en el mapa coinciden con las posiciones del robot en el mundo de Gazebo

[Screencast from 2024-09-28 13-18-36.webm](https://github.com/user-attachments/assets/b67f48ed-97cc-4b55-9527-ac925d1b35a4)

### Matriz de ocupación

Como parte del registro, se creó una matriz de ocupación, donde cada celda abarca 15x15 píxeles, y se marca si una casilla está ocupada, ha sido visitada o permanece libre. Mientras el robot se desplaza por el entorno, estas celdas se pintan, registrando el progreso del robot en la limpieza del área.

Aquí se muestra el mapa con las casillas:

![image](https://github.com/user-attachments/assets/266f2d7c-a1df-4df7-b5ce-5ce99d22421b)


## Planificación

Para empezar, registramos todas las casillas blancas como puntos de retorno, evitando así que nos podamos dejar algún sitio sin visitar.
Para la planificación usamos el algoritmo BSA, basándonos en las condiciones expuestas en el paper expuesto en clase: 

![image](https://github.com/user-attachments/assets/1c2b3628-dab6-48bc-aac9-98e4547f72ca)

Fijamos la posición inicial en WEST y vamos rotando en dirección a las agujas del reloj:

![image](https://github.com/user-attachments/assets/90ad7dca-f319-4863-b33a-a716910bc8c6)

Primero miramos si está rodeado de casillas ocupadasd, si es así, buscamos un punto de retorno mediante una heurística que evalua el mejor punto al que ir desde nuestra posición teniendo en cuenta la distancia al punto y los puntos intermedios encontrados mediante BFS.
Si no está encerrado, miramos si en la dirección principal hay algún obstáculo, si no es así seguimos hacia delante, si es así, rotamos a la siguiente dirección en el orden anterior de preferencia. 
Por último, si no estamos en ninguno de los tres casos, seguimos hacia delante.

Durante este algoritmo, cada punto que necesita un cambio de dirección, sitios donde se queda encerrado y caminos para ir a puntos de retorno son guardados en una lista de posiciones a las que navegar para facilitar el trabajo en la fase de navegación.

Aquí dejo un vídeo de como planifica la trayectoria en el mapa de celdillas 15x15

[Screencast from 2024-10-08 12-39-21.webm](https://github.com/user-attachments/assets/65d9e660-811a-406a-b5b7-e3449957d706)

## Navegación

Para la navegación he implementado dos controladores proporcionales, uno para el movimiento lineal y otro para el angular. He mirado el error lineal mediante la fórmula de la distancia euclídea y la angular mediante la arcotangente de la diferencia de la posición x y la posición de y. Luego he normalizado el ángulo en [-pi, pi]

Si el error lineal era menor a un umbral establecido, se considera que ya está en la celdilla y pasa al siguiente punto, si el error angular es muy grande, se anula el movimiento angular y mediante el controlador angular se orienta hacia el ángulo target. En cualquier otro caso, los dos controladores comandan velocidades.


## Ejecución

 Aquí dejo una foto del resultado de la ejecución:

![image](https://github.com/user-attachments/assets/88b67edd-1988-4471-89b4-68a2577ad65f)

Y un vídeo de la ejecución completa:

[![image](https://github.com/user-attachments/assets/7d48b5fe-d7f3-47e7-8b7e-525f7a901a2a)](https://youtu.be/X2KAd5Se6oM?si=4t_PIoZ0DBObWS2y)


## Observaciones

Al principio intenté fusionar la planificación y la navegación, pero es mucho más complicado de depurar y, una vez implementado las dos partes por separado, considero que esta última solución es más robusta.



# Rescue people

Esta práctica se enfoca en la localización de personas en el mar para llevar a cabo rescates, utilizando drones como herramienta principal. Disponemos de las coordenadas en GPS del barco y de la zona donde están las víctimas y mediante el uso de una cámara, debemos detectar sus caras.

## Cambios de coordenadas

Se nos proporcionan la posición del barco y la de las personas en coordenadas GPS. Para poder ubicarnos en el mapa de gazebo, pasamos estas coordenadas al sistema UTM, el cual nos permite, mediante la diferencia de la posición target a nuestra posición de inicio, hallar la posición en el mundo de gazebo, ya que nuestra posición inicio es el origen del eje de coordinadas del mundo.

Pasamos las coordenadas a UTM mediante la siguiente página obteniendo:

* Posición origen:
  
![image](https://github.com/user-attachments/assets/577261ab-04cd-48dd-9749-7bc9d2225fc8)

* Posición de las víctimas:

![image](https://github.com/user-attachments/assets/0d8de4bc-ab33-4a65-b33e-da584c4667b7)

Restando obtenemos que nuestras coordenadas target en el mundo gazebo son (40, -30).

Para imprimir la posición de cada persona, he usado la posición en utm del punto origen o base y le he sumado la posición en el mundo simulado, y mediante la función utm.to_latlon(), he conseguido la latitud y longitud de la posición en la que la ha encontrado.

## Algoritmo de barrido

Para optimizar la búsqueda de personas en áreas de rescate, he implementado un algoritmo de barrido en espiral que permite cubrir de manera eficiente el área alrededor de una posición inicial conocida. El objetivo es realizar un patrón de exploración que permita maximizar la cobertura del terreno, asegurando que se inspeccionen todos los puntos cercanos.

El algoritmo comienza en una posición inicial y luego se mueve secuencialmente en un patrón en espiral alrededor de ese punto. Para lograr esto:

* Iniciamos un valor de desplazamiento (number) en 0. Este valor se incrementa de forma progresiva para generar el crecimiento de la espiral.

* El valor de la posición se alterna entre desplazamientos en las coordenadas x e y, aumentando o disminuyendo según la dirección en la que se mueve el sistema. Esto asegura que el área alrededor de la posición inicial se cubra de manera sistemática.

* Cada cuatro movimientos (un ciclo completo de la espiral en las direcciones x+, y+, x-, y-), se incrementa el tamaño del paso en 2.5 unidades, ampliando así el radio de la espiral y permitiendo la exploración de áreas más lejanas.

## Detección de Caras

Para la detección de caras decidí analizar la imagen únicamente si alguno de los píxeles fuera muy diferente al resto. He analizado la imagen usando la función de opencv2 cv.CascadeClassifier. Para la correcta detección tuve que ajustar los parámetros scaleFactor, minSize y maxSize.

Estos son mis valores para una altura de 1.5:

```python
faces = face_cascade.detectMultiScale(
        frame_gray,
        scaleFactor=1.1,      
        minSize=(35,35),
        maxSize=(80,80), 
    )

```

Uno de los problemas que enfrenté fue la limitada capacidad del detector para reconocer rostros que no estén perfectamente alineados (es decir, en posición vertical u horizontal). Para solucionar esto, implementé un proceso de rotación incremental: cada vez que un frame no sea predominantemente "agua" (es decir, que contenga algo más significativo), lo roto en incrementos de 10 grados, hasta un total de 35 veces, buscando detectar rostros desde distintos ángulos.

Si se detecta una cara, compruebo si hay alguna en la lista de posiciones donde he encontrado cara, de estar vacía, añado la posición donde se capturó el frame. Si no, recorro la lista de posiciones y compruebo cual es la distancia euclídea mínima, de ser mayor a un umbral impuesto, considero que es una persona diferente. Si la distancia es menor al umbral y el tiempo de la última detencción es mayor a una constante, considero que estoy volviendo a visualizar a la persona más cercana y modifico la posición guardada. 

## Batería baja

Una vez pasados 8 minutos del despegue, consideramos que hemos agotado la batería y nos queda lo justo para volver a base, por lo cual, comando al drone a ir a la posición (0,0,1), y una vez llega a esa posición, este comienza el aterrizaje.


## Ejecución

Aquí dejo un vídeo del resultado final:

[![image](https://github.com/user-attachments/assets/bbe26884-b7a4-4321-884e-ce01a3edfc0a)](https://www.youtube.com/watch?v=JgU8OZh0u5E)


# AutoParking

En esta práctica, desarrollamos un sistema autónomo de estacionamiento para un robot móvil. El objetivo principal es que el robot sea capaz de identificar un espacio disponible utilizando un sensor láser, alinearse correctamente con él y ejecutar maniobras para estacionarse de forma eficiente.

---

## Objetivos
El propósito de esta práctica es dotar al robot de la capacidad de estacionarse de forma autónoma. Esto implica tres tareas: detectar un hueco adecuado, alinear el robot con los límites del espacio y realizar las maniobras necesarias para estacionarse sin colisionar. El éxito del proyecto depende de la correcta integración de sensores y algoritmos de control.

---

## Desarrollo

### Percepción del Entorno
Para detectar el hueco donde se estacionará, el robot utiliza tres sensores láser, frontal, lateral y trasero, que proporcionan distancias en un rango de 180 grados. Estos datos se procesan para determinar si un punto del entorno está dentro de los límites predefinidos del espacio de estacionamiento. Los límites del hueco se establecen mediante coordenadas cartesianas y se comprueban con una función que evalúa si un punto pertenece al área.

```python
rectangle_limits = {
    "x_min": 0.5,
    "x_max": 3.5 + GAP_WIDTH,
    "y_min": -5 - GAP_LENGTH,
    "y_max": -5
}

def point_inside_rectangle(x, y, limits):
    return limits["x_min"] <= x <= limits["x_max"] and limits["y_min"] <= y <= limits["y_max"]
```

Si se detectara que hay un solo pixel ocupado en ese hueco, no se consideraría apto para aparcar.

### Alineación con el hueco

Para garantizar que el robot pueda maniobrar correctamente, es necesario que esté alineado con los bordes del espacio detectado. Esto se logra mediante el cálculo de la pendiente entre los puntos obtenidos por el sensor láser. Si el robot no está alineado, ajusta su orientación mediante rotaciones controladas y pequeños movimientos hacia adelante. Este proceso de alineación es crucial para preparar al robot antes de iniciar la maniobra de estacionamiento.

### Maniobra de estacionamiento

Una vez alineado y posicionado cerca del hueco, el robot inicia la maniobra de estacionamiento. Este proceso incluye un retroceso inicial hacia el hueco con un giro controlado, seguido de un ajuste de orientación y un último movimiento para centrar al robot dentro del espacio. Las maniobras se controlan en función de un temporizador y un contador que definen cada etapa del estacionamiento.

## Resultado final

Caso típico (entre dos coches alineado):

[video](https://github.com/user-attachments/assets/dba1e4f1-158d-45e6-a02f-e1420dea105d)

Caso entre dos coches inclinado:

[video](https://github.com/user-attachments/assets/19ce6d29-d4db-4fee-99a0-123787a40e08)

Caso aparcamiento detrás de coche de referencia:

[video](https://github.com/user-attachments/assets/e5094ff9-247c-4841-af5a-492144784617)

Caso aparcamiento delante de coche:

[video](https://github.com/user-attachments/assets/23eeb53b-c093-4e72-bfad-8a36e1d43f66)


---

# Amazon Warehouse

El propósito de esta práctica es desarrollar la lógica de un robot logístico que permita realizar tareas esenciales dentro de un almacén automatizado. El robot deberá desplazarse hasta un estante específico, levantarlo, y transportarlo de manera eficiente hacia un punto objetivo en el mapa. Para lograrlo, cuenta con un mapa del entorno y es capaz de autolocalizarse con precisión en él. Este proyecto se divide en tres etapas principales: crear el plan y navegar.

## Plan

### Transformación de coordenadas del simulador al mapa

La primera etapa del proyecto consiste en planificar la trayectoria que llevará al robot desde su posición inicial hasta el punto donde se encuentra el estante. Esta planificación requiere una comprensión precisa de cómo transformar las coordenadas del simulador, que utiliza un sistema tridimensional, al sistema de coordenadas bidimensional del mapa.

En el simulador, el origen de coordenadas se encuentra en el centro del entorno, con el eje X apuntando hacia el frente del robot, el eje Y hacia su izquierda y el eje Z hacia arriba. Este sistema está desplazado respecto al mapa, que tiene el origen en la esquina superior izquierda, con el eje X apuntando hacia abajo y el eje Y hacia la derecha. Además, el simulador tiene dimensiones de 20.62 metros en el eje X y 13.6 metros en el eje Y, mientras que el mapa está compuesto por 415 píxeles de ancho y 279 píxeles de alto.

Para realizar la transformación, es necesario tener en cuenta varios factores:

1. **Escala**: La escala de conversión entre el simulador y el mapa se basa en las dimensiones de ambos sistemas. En el eje X, la escala es de 20.12 píxeles por metro, y en el eje Y es de 20.5 píxeles por metro. Estas escalas se utilizan para convertir las distancias en metros del simulador a unidades en píxeles en el mapa.

2. **Rotación de 180°**: Hay una rotación de 180° entre ambos sistemas. El eje X del simulador se invierte en el mapa, mientras que el eje Y permanece en la misma dirección pero se ajusta a la escala de píxeles.

3. **Translación**: El simulador tiene un desplazamiento respecto al mapa. El origen del simulador está ubicado a 6.964 metros en el eje X y 10.386 metros en el eje Y respecto al mapa.

### Fórmulas de transformación

Para transformar un punto del simulador \( (x_{\text{sim}}, y_{\text{sim}}) \) a las coordenadas del mapa \( (x_{\text{map}}, y_{\text{map}}) \), aplicamos las siguientes fórmulas:

- Para la coordenada \( x_{\text{map}} \) en el mapa:

```
x_map = ((- (x_sim - 6.964) / 20.5) + 6.964) * 20.12
```

- Para la coordenada \( y_{\text{map}} \) en el mapa:

```
y_map = (10.386 - (y_sim - 13.6)) * 20.12
```


## Transformación del mapa al simulador

La primera etapa del proyecto consiste en planificar la trayectoria que llevará al robot desde su posición inicial hasta el punto donde se encuentra el estante. Esta planificación requiere una comprensión precisa de cómo transformar las coordenadas del mapa, que se encuentran en un sistema bidimensional, al sistema de coordenadas tridimensional del simulador.

En el simulador, el origen de coordenadas está en el centro del entorno, con el eje X apuntando hacia el frente del robot, el eje Y hacia su izquierda y el eje Z hacia arriba. Sin embargo, este sistema está desplazado respecto al mapa, que tiene el origen en la esquina superior izquierda. El mapa utiliza un sistema de coordenadas bidimensional donde el eje X apunta hacia abajo y el eje Y hacia la derecha. Además, las dimensiones del simulador son 20.62 metros en el eje X y 13.6 metros en el eje Y, mientras que el mapa tiene una resolución de 415 píxeles de ancho y 279 píxeles de alto.

Para transformar las coordenadas del mapa a las del simulador, se debe tener en cuenta la rotación, la translación y la escala entre ambos sistemas. Las coordenadas \( (x, y) \) en el simulador se calculan a partir de las coordenadas x_map, y_map del mapa usando las siguientes fórmulas de transformación:

- Para la coordenada \( x \) en el simulador:

```
x_sim = -((x_map - 207.5) / 20.12) + 6.964
```

- Para la coordenada \( y \) en el simulador:

```
y_sim = (139.5 - y_map) * 20.5 + 10.386
```

### OMPL

OMPL (Open Motion Planning Library) es una biblioteca de código abierto diseñada para la planificación de trayectorias de robots y sistemas autónomos. Esta biblioteca proporciona una variedad de algoritmos que permiten generar trayectorias eficientes y libres de colisiones en espacios de alta dimensión. Entre estos algoritmos, destacan técnicas como RRT (Rapidly-exploring Random Trees) y sus variantes, las cuales se utilizan comúnmente en planificación de movimientos.

Para la implementación en este proyecto, se ha optado por utilizar el planificador **A\*** (A-star). Este algoritmo de búsqueda es ampliamente utilizado para encontrar el camino más corto entre un punto de inicio y un objetivo en un espacio de búsqueda, lo que lo convierte en una opción adecuada para planificar trayectorias de robots en entornos complejos. A* se basa en una función de evaluación:


```
f(n) = g(n) + h(n)
```

Donde:

- **g(n)** es el costo del camino desde el nodo inicial hasta el nodo `n`.
- **h(n)** es una estimación (heurística) del costo para llegar desde el nodo `n` hasta el objetivo.


Donde:

- **g(n)** es el costo del camino desde el nodo inicial hasta el nodo `n`.
- **h(n)** es la heurística, una estimación del costo para llegar desde el nodo `n` hasta el objetivo.


En este proyecto, la implementación de OMPL se centra en la planificación de trayectorias para un robot en un mapa. Para ello, se define un **punto de origen** (con coordenadas \(x, y, \text{yaw}\)) y un **punto objetivo** con la misma estructura. Los diferentes estados evaluados por el planificador se verifican a través de la función **`isStateValid`**, que asegura que el estado del robot sea válido y libre de colisiones. Luego, si se genera un plan, la función **`plan`** devuelve un array de puntos a seguir. Este plan generado no siempre llega al target, por lo cual, se calcula la distancia entre el punto final del path y el punto target, si la distancia es menor a 1 pixel, se da por bueno, si no, se replanifica.

#### Implementación con el Robot como un Punto

En la **primera etapa**, modelamos al robot como un **punto** sin dimensiones, representando solo su ubicación en el espacio. Para evitar colisiones con los obstáculos, **engordamos el mapa** añadiendo un **grosor del tamaño del radio del robot**, más un pequeño **offset** adicional para crear un margen de seguridad.

En la función **`isStateValid`**, verificamos que el **píxel central** del robot (en las coordenadas \(x, y\)) sea un píxel libre de obstáculos (blanco). Esto garantiza que el centro del robot esté en un área libre de colisiones y, por lo tanto, el estado es válido.

#### Implementación con el Robot como una Geometría Cuadrada

En la **segunda etapa**, tratamos al robot como una **geometría cuadrada**, con un tamaño que representa las dimensiones físicas del robot. Para evitar que el robot roce con los obstáculos, se incrementa ligeramente el tamaño de la geometría del robot, creando un margen de seguridad adicional.

En esta implementación, la función **`isStateValid`** comprueba una **matriz de píxeles** centrada en la ubicación del robot, con un tamaño igual al del robot más el margen de seguridad. Se valida que todos los píxeles dentro de esta matriz sean blancos, lo que asegura que el área total ocupada por el robot esté libre de obstáculos. Si algún píxel dentro de esta área está ocupado por un obstáculo, el estado se considera inválido.

#### Implementación del robot con el estante

En esta última etapa, se incorpora la rotación del robot en la planificación de su trayectoria. A diferencia de las etapas anteriores, donde se consideraba al robot como un punto o un cuadrado, en este caso el robot se modela como un objeto rectangular, similar a un estante. Este cambio tiene en cuenta las dimensiones reales del robot y su orientación en el espacio, lo cual es importante para evitar colisiones.

Para implementar esta solución, se utiliza el ángulo de yaw proporcionado por el planificador A* para calcular la orientación del robot. A partir de este ángulo, se genera una matriz de rotación que transforma las posiciones de las esquinas del rectángulo que representa al robot. Estas esquinas se desplazan según las coordenadas xx e yy determinadas por el planificador, lo que posiciona el robot en el mapa.

Con las nuevas posiciones de las esquinas del robot, se genera un polígono que representa el área ocupada por el robot. Este polígono se dibuja en el mapa y se verifica si se superpone con obstáculos. Para hacer esto, se utiliza una máscara que cubre el área del robot y se compara con el mapa de obstáculos. Si alguna parte del polígono se encuentra en un área ocupada por obstáculos, el estado del robot se considera inválido.

De esta manera, se asegura que no solo se eviten colisiones en el centro del robot, sino también en su contorno, considerando tanto el tamaño como la orientación del robot en el entorno. Esto permite una planificación más precisa y realista, ya que el robot se representa con sus dimensiones completas y su orientación exacta en el espacio.


## Navigation

En la fase de navegación, el robot sigue los puntos generados previamente durante la planificación de la trayectoria. Estos puntos, que fueron calculados en el espacio de trabajo del mapa, se transforman a las coordenadas del simulador, permitiendo que el robot se desplace dentro del entorno simulado. La transición entre estos puntos se realiza mediante un ciclo iterativo, donde el robot se dirige de uno a otro, ajustando su trayectoria según su posición actual.

Para asegurarse de que el robot está lo suficientemente cerca de un punto de destino, se calcula la distancia entre su posición actual y el objetivo utilizando la fórmula de distancia euclidiana. Si esta distancia es menor que un umbral predefinido, el robot considera que ha llegado al punto y procede al siguiente. Si la distancia es mayor que el umbral, el robot sigue ajustando su trayectoria hasta acercarse lo suficiente al objetivo.

Para controlar el movimiento del robot, se implementan dos controladores proporcionales: uno para la velocidad lineal y otro para la velocidad angular. El controlador de velocidad lineal ajusta la rapidez con la que el robot se mueve hacia el objetivo, basándose en el error de distancia entre la posición actual y el destino. El controlador de velocidad angular se encarga de alinear el robot con la dirección correcta, tomando en cuenta la diferencia entre el ángulo de orientación del robot y el ángulo hacia el objetivo. Ambos controladores ajustan las velocidades en función de los errores, permitiendo que el robot navegue de manera fluida y precisa hacia su destino.

Este enfoque permite que el robot siga una ruta de manera efectiva, ajustándose dinámicamente a las variaciones de su entorno mientras minimiza el error de posición y orientación a lo largo del camino.

## Resultado final

[Screencast from 2024-12-01 20-49-21.webm](https://github.com/user-attachments/assets/176a6a44-1081-46ac-94e2-b5ade2fc6485)

[Screencast from 2024-12-01 22-52-26.webm](https://github.com/user-attachments/assets/b6dd4b51-fef7-4c0e-b180-a12b5c01a3ae)

---

# Marker Visual Localization

Esta práctica final consiste en programar un robot doméstico para que, mediante el uso de su cámara integrada, pueda autolocalizarse detectando diversas balizas distribuidas en el entorno. El robot debe localizarse mientras navega por la casa.

## Autolocalización

Para la autolocalización, utilizamos balizas de la librería **AprilTags**, que son códigos en blanco y negro con un patrón único de cuadrados y tamaño fijo. Estas balizas están registradas con un identificador (ID) y coordenadas propias, las cuales utilizaremos para calcular diferentes matrices de transformación y estimar la posición del robot en el mundo simulado de **Gazebo**.

Para obtener la posición absoluta del robot, necesitamos la matriz `world2cam`, que se calcula multiplicando las matrices de transformación `world2bal` y `bal2cam`.

### Matriz world2bal

La matriz de transformación `world2bal` se calcula utilizando el ID de la baliza detectada para acceder a su posición y orientación asociadas. Inicializamos una matriz \( 4 \times 4 \), donde las primeras tres filas y columnas corresponden a una matriz de rotación respecto al eje \( z \), basada en el ángulo asociado a la baliza. La última columna incluye las posiciones \( x \), \( y \), y \( z \), siendo \( z \) un valor constante de **0.8**.

### Matriz bal2cam

Para calcular esta matriz, primero detectamos las esquinas de la baliza en la imagen capturada. Usamos la función `solvePnP` de OpenCV con el flag `SOLVEPNP_IPPE_SQUARE` para obtener los vectores de rotación (\( rvec \)) y traslación (\( tvec \)). Es fundamental usar los puntos de referencia según la documentación de OpenCV. A continuación, creamos una matriz \( 4 \times 4 \) llamada `cam2bal`. Convertimos el vector de rotación (\( rvec \)) en una matriz \( 3 \times 3 \) usando la función `Rodrigues` de OpenCV. Esta matriz corresponde a las primeras tres filas y columnas de `cam2bal`, mientras que el vector de traslación (\( tvec \)) se ubica en las primeras filas de la última columna.

Como necesitamos la matriz `bal2cam`, invertimos `cam2bal` y aplicamos rotaciones de \(-90^\circ\) en \( z \) y \(-90^\circ\) en \( x \), en ese orden. Esto se debe a que los sistemas de coordenadas de OpenCV y Gazebo son diferentes. En OpenCV, el eje \( x \) apunta a la derecha y el eje \( y \) hacia abajo, mientras que en Gazebo, el eje \( x \) apunta a la derecha, el eje \( y \) hacia adelante, y el eje \( z \) hacia arriba. Tras estos ajustes, obtenemos la matriz `bal2cam`.

### Matriz world2cam y posición estimada

La matriz `world2cam` se obtiene al multiplicar las matrices `world2bal` y `bal2cam`. Para determinar la posición estimada del robot, extraemos las coordenadas \( x \) y \( y \) de las dos primeras componentes de la última columna de `world2cam`. El ángulo de orientación se calcula a partir de la matriz de rotación, que corresponde a las primeras \( 3 \times 3 \) componentes de `world2cam`.

Para calcular el ángulo de orientación, utilizamos las siguientes fórmulas para el **theta** y el **yaw**:

$$
\theta = \arctan2\left(-R_{2,0}, \sqrt{R_{0,0}^2 + R_{1,0}^2}\right)
$$

$$
yaw = \arctan2\left(\frac{R_{1,0}}{\cos(\theta)}, \frac{R_{0,0}}{\cos(\theta)}\right) + \frac{\pi}{2}
$$

En estas fórmulas, \( R \) representa la matriz de rotación obtenida de `world2cam`, y \( R_{i,j} \) corresponde a los elementos individuales de esta matriz. Es necesario sumar \( +\pi/2 \) al valor de **yaw** debido a un desfase entre el ángulo calculado y el ángulo real.

Con estas transformaciones y cálculos, logramos determinar la posición y orientación estimada del robot en el mundo simulado.

Cuando la baliza no es visible, recurro a la posición dada por la odometría para la estimación del robot.

## Navegación

La navegación del robot se implementa mediante una máquina de estados que opera de forma pseudoaleatoria. El sistema consta de tres estados principales: **Forward**, **Back**, y **Turn**.

Dado que el robot no dispone de sensores de proximidad como bumpers o láseres, se asume que ha colisionado con un obstáculo si, tras un tiempo determinado, la distancia recorrida no supera un umbral definido. En este caso, el robot pasa del estado **Forward** al estado **Back**.

En el estado **Back**, el robot retrocede durante 2 segundos para alejarse del posible obstáculo. A continuación, entra en el estado **Turn**, donde realiza un giro a una velocidad angular aleatoria seleccionada entre 0.45 y 1. Este giro se mantiene durante 5 segundos, después de lo cual el robot vuelve al estado **Forward** y reanuda su movimiento hacia adelante.

Esta lógica permite al robot navegar de forma reactiva y adaptarse dinámicamente a su entorno, aunque no disponga de sensores de proximidad tradicionales.

## Resultado

[![image](https://github.com/user-attachments/assets/b1017c72-da55-4a47-afae-185ea37795bf)](https://youtu.be/FuojGCd9sYs)

[![image](https://github.com/user-attachments/assets/ef9cadb8-33d8-4536-87a0-485decc0990a)](https://youtu.be/rSkwMMfka-A)

En este último video podemos observar que este método de localización, cuando la baliza está a más de 4 metros de distancia, empieza a tener errores en las estimaciones.




