# ROBÓTICA DE SERVICIOS

# Indice
* [Indice][ind]
* [Welcome][wel]
* [Localized Vacuum Cleaner][p1]



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

Esta práctica trata de limpiar la mayor superficie posible de suelo en el menor tiempo utilizando un robot de limpieza, el cual es capaz de localizarse en el mapa de la casa. Para ello trabajaremos en 3 fases diferentes: registro, planificación y ejecución.

## Registro

Primero, decidí el tamaño de cuadrículas que quería. Para ello, medí el robot y decidí que las cuadrículas debían de ser de 15x15 para que no se dejase trozos sin limpiar ni pasara muchas veces por el mismo sitio. 

![image](https://github.com/user-attachments/assets/973843da-028e-4cd1-8df4-f35962201f12)

El tamaño de estas cuadrículas cambia más adelante por 33x33 debido a que el nuevo mapa de unibotics tiene otras proporciones y era el ajuste más óptimo. 

Para el registro del mapa en píxeles he cogido varios puntos de gazebo a pixeles y mediante regla de 3 obtuve una escala. Para la matriz de transformación de 3D a 2D saqué el ángulo de rotación que es de -90º y tenemos una translación de 4.06 y 5.38 metros, quedando la matriz:

<div align="center">

| 0 | 1 | 0 | 4.06 |
|---|---|---|---|
| -1 | 0 | 0 | 5.38 |
| 0 | 0 | 1 |  0  |
| 0 | 0 | 0 |  1  |

</div>

Luego para comprobar que las posiciones tuvieran coherencia en el mapa, desplacé el robot en los cuatro cuadrantes que hay respecto al eje de gazebo y dibujé su posición en las cuadrículas. En el siguiente vídeo podemos ver como los puntos en el mapa se corresponden a las posiciones en el mundo de gazebo, comprobando así que los cálculos anteriores son correctos.


[Screencast from 2024-09-28 13-18-36.webm](https://github.com/user-attachments/assets/b67f48ed-97cc-4b55-9527-ac925d1b35a4)

Para terminar con el registro, diseñé otra matriz, donde cada celda abarca 15x15 píxeles y pintamos si esa casilla está ocupada, ha sido visitada o no. En este momento dibujé por la cuadrículas por las que iba pasando el robot.

Mapa casillas ocupación:

![image](https://github.com/user-attachments/assets/266f2d7c-a1df-4df7-b5ce-5ce99d22421b)


# Planificación

Algoritmo BSA:

[Screencast from 2024-10-08 12-39-21.webm](https://github.com/user-attachments/assets/65d9e660-811a-406a-b5b7-e3449957d706)


# Ejecución

Resultado final:

![image](https://github.com/user-attachments/assets/88b67edd-1988-4471-89b4-68a2577ad65f)

