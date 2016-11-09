.. -*- coding: utf-8 -*-

.. _rcs_subversion:

Clase 11 - PIII 2016
====================

Grabación de dsPIC con Pickit 3
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- El Pickit 3 permite programar el dsPIC grabando el archivo .hex compilado con el mikroC
- Requiere el aplicativo programador. Descargarlo desde http://www.microchip.com/pagehandler/en-us/devtools/dev-tools-parts.html

.. figure:: images/clase10/pickit3_1.png

- Conectar el Pickit 3 a la PC y esperar que instale controladores (la instalación del aplicativo instala los controladores también).

- Para abrirlo ejecutamos:

.. figure:: images/clase10/im4.png

- Podemos probar conectando la Demo board que viene con el PicKit 3 ( más info en: http://ww1.microchip.com/downloads/en/DeviceDoc/41296B.pdf )

- Le damos a Check Comunication y nos detecta la Demo Board conectada:

.. figure:: images/clase10/im6.png

- Si conectamos el circuito de grabación del dsPIC30F3010, también lo detecta:

.. figure:: images/clase10/im7.png

- Se puede leer el dsPIC y grabar el firmware en un .hex y también se puede escribir nuestro .hex creado con mikroC.

**Ejercicio 1:**

- Hacer un Hola Mundo en mikroC simplemente para hacer parpadear un led. Escribir el programa en mikroC, compilar para generar el hex, grabarlo con el PicKit 3 y por último probarlo en la placa.


Amplificador operacional
^^^^^^^^^^^^^^^^^^^^^^^^

- Entrada diferencial y una ganancia muy alta (generalmente mayor que 100)
- En general se alimenta con fuentes positivas y negativas (permite excursiones tanto por arriba como por debajo tierra)
- Su nombre proviene de una de sus utilidades básicas, como lo son realizar operaciones matemáticas
- Un amplificador operacional ideal tendría ganancia infinita y ninguna corriente de entrada

**Sumador**

- Es probable que el más útil de los circuitos es el circuito amplificador sumador
- Ejemplo: sumador de tres voltajes de entrada, cada uno multiplicado por un factor de ganancia constante

.. figure:: images/clase10/sumador.png

- El voltaje de salida puede expresarse como:

.. code-block:: 

	Vo = (Rf/R1)V1 + (RF/R2)V2 + (Rf/R3)V3

**Ejercicio 2:**

- Utilizar un 741 ( datasheet en: http://www.ti.com/lit/ds/symlink/ua741.pdf )para sumar dos señales (una senoidal y ruido)
- Hacerlo en proteus
- Luego en protoboard con los generadores de señales

**Ejercicio 3:**

- Utilizar un 741 para agregar un offset a una señal de audio
- Hacerlo en Proteus con un archivo WAV
- Luego en protoboard sacando el audio del celular o laptop



