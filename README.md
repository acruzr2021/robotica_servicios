# ROBÓTICA DE SERVICIOS

# Indice
* [Indice][ind]
* [Welcome][wel]
* [Localized Vacuum Cleaner][p1]
* [Rescue People](#Rescue-people)



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


