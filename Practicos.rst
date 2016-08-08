.. -*- coding: utf-8 -*-

.. _rcs_subversion:

Prácticos - PIII 2015
=====================

**5 ejercicios por alumno**

- Daniel Martin	
- Alejandro Luna Bringas	
- Dario Fiorucci	
- Gustavo Saracho	
- Claudio Balmaceda	
- Tomás Magnin	
- Ramiro Walter Mazzoli Manfredi	
- Juan Cruz Mirgone	
- Francisco Paglia	
- Carolina Vivar	
- Rodrigo Passirani

**Ejercicio 1:**

- Programar un dsPIC33FJ32MC202 para que muestree una señal de audio que tiene una amplitud variable pero no supera los 1.5V y no disminuye de 0V. Su frecuencia no supera los 800Hz.
- Utilizar VRef+ = 1.5V 
- Controlar la frecuencia de muestreo con un generador externo de señal cuadrada (con el flanco descendente).
- Usar un DAC R-2R y colocar un osciloscopio para ver la señal analógica de entrada y la señal analógica de salida.
- El objetivo del práctico es para poder observar la señal analógica de salida a medida que modificamos la frecuencia de muestreo variando la señal cuadrada.

**Ejercicio 2:**

- Generar dentro del dsPIC33FJ32MC202 una señal senoidal de 15Hz.
- Configurar un ADC manual a una frecuencia de muestreo de 800Hz controlado por el Timer 2.
- Muestrear una señal analógica de entrada (amplitud fija entre 0V y 1V, y frecuencia fija de 400Hz).
- Modular en amplitud la señal de entrada con la señal generada de 15Hz como modulante. Con un índice de modulación igual a 1.
- Usar un DAC R-2R.

**Ejercicio 3:**

- Muestrear la voz humana con un ADC automático con el dsPIC33FJ32MC202.
- Entregar los cálculos realizados para la obtener la frecuencia de muestreo.
- Como señal de audio de entrada utilizar un archivo de audio en Proteus.
- Usar un DAC R-2R.

**Ejercicio 4:**

- Programar en un dsPIC33FJ32MC202 un filtro pasa bajos (Fs = 3kHz - Fc = 200Hz).
- Usar un DAC R-2R.
- Colocar a la entrada un generador de señal y un osciloscopio a la salida.

**Ejercicio 5:**

- Elegir alguno de los ejercicios anteriores, adaptarlo para el dsPIC30F4013 y probarlo en la placa EASYdsPIC 4.
- Entregar proyecto mikroC y mostrar su funcionamiento en la placa (pueden grabar un video o mostrarlo durante alguna de las clases).

**Nota:** Para todos los ejercicios:

- Código fuente entendible, sangría y espacios adecuados.
- Organización del código fuente en distintas funciones. Por ejemplo: void config_adc() void config_int0() void config_ports().
- Decidir las demás configuraciones que no están indicadas en el enunciado.
- Entregar proyecto mikroC y Proteus (salvo ejercicio 5).









