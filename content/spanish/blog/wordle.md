---
title: "Cómo construir un Wordle con Flutter"
meta_title: "Cómo construir un Wordle con Flutter"
description: "Cómo construir un Wordle con Flutter"
date: 2022-01-14T05:00:00Z
image: "/images/blog/wordle/wordle1.png"
categories: ["POC"]
author: "Fernando Souto"
tags: ["wordle", "Flutter", "Game", "Challenge"]
draft: false
---

Vi el fin de semana pasado a algunas personas compartiendo cuadros aleatorios en Twitter y estuve pensando…
>  ¿Qué es esto?

![Mi amigo Camilo](/images/blog/wordle/camilo.png)

After researching a bit, I found a post explaining that this is a game called **Wordle** and it is a super awesome game.
espués de investigar un poco, encontré una publicación que explicaba que esto es un juego llamado **Wordle** y es un juego súper genial.

**Kudos al desarrollador.**

Tienes que adivinar una palabra por día.

The idea of Having to enter one word per day is like the old times in the internet. The developer behind **Wordle** is not trying to push our limits to play like addicts and thats **super nice**.
La idea de tener que ingresar una palabra por día es como en los viejos tiempos en internet. El desarrollador detrás de **Wordle** no está tratando de llevarnos al límite para jugar como adictos y eso es **súper agradable.**
>  Este enfoque de “una vez al día” huele a internet antes de los algoritmos y las redes sociales.

Y después de jugar un par de días me preguntaba:
>  Puedo crear algo similar en unas pocas horas con Flutter?

Me reté a mí mismo para crear el clon de **Wordle ** en menos de 4 horas con las increíbles herramientas de desarrollo de  **Flutter. **
I push my limits to create the clon of **Wordle** in less than 4 hours with the awesome **Flutter** dev tools.

![Wordle con Flutter](/images/blog/wordle/wordle1.png)

## El juego

Tengo una matriz de 5x6 para los colores y los caracteres, y cada vez que el usuario ingresa un carácter con el teclado virtual, coloco el carácter en el lugar correcto.

![La matriz](/images/blog/wordle/wordle2.png)

Cuando el usuario llega al final de la fila, puede ELIMINAR un carácter o hacer clic en ENTER para verificar la palabra actual. Si el usuario decide presionar ENTER, entonces ejecuto el algoritmo para verificar la entrada contra la solución con el siguiente código (quizás no sea el mejor, pero estaba apurado para completar la tarea):

![Algoritmo del juego para verificar la solución](/images/blog/wordle/wordle3.png)

El resto del código es para construir la interfaz de usuario.

En el siguiente código estamos construyendo la parte superior de la interfaz con los cuadros en los que el usuario necesita ingresar los caracteres.

![Input del usuario](/images/blog/wordle/wordle4.png)

Para *el teclado*, reconstruí un poco una biblioteca llamada *virtual_keyboard_multi_language*.

Es una columna simple con 3 filas, con algo de decoración personalizada para mostrar la apariencia del teclado de **Wordle**, algunos arreglos con los valores para cada posición y ¡eso es todo!

![Teclado virtual](/images/blog/wordle/wordle5.png)

![Valores de cada elemento por fila](/images/blog/wordle/wordle6.png)

## Comentarios finales

Si aún no has probado el juego original, por favor hazlo:
[**Wordle - A daily word game**
*Adivina la palabra oculta en 6 intentos. Un nuevo rompecabezas está disponible cada día.* www.powerlanguage.co.uk](https://www.powerlanguage.co.uk/wordle/)

Este es el código fuente de mi pequeña copia del juego:
[**GitHub - ficiverson/wordle-flutter**
*You can't perform that action at this time. You signed in with another tab or window. You signed out in another tab or…*github.com](https://github.com/ficiverson/wordle-flutter)

No quiero *copiar la idea ni tener una versión comercial del juego*, pero vi a un tipo que publicó el juego **en Appstore sin el permiso** de los desarrolladores originales y **eso no está bien**

![Copy guy](/images/blog/wordle/copy.png)

Todo el crédito de la idea para **los autores originales**. ¡Gracias! :)
