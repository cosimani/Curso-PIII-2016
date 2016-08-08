.. -*- coding: utf-8 -*-

.. _rcs_subversion:

Clase 09 - PIII 2015
====================

**Probando filtros en Proteus y en Placa**

- Video sobre cómo utilizar el generador de señal (https://www.youtube.com/watch?v=qCRcNYbqBxs)

**Ejemplo para dsPIC33FJ32MC202 para Proteus**

.. code-block:: c

	// Device setup:
	//     Device name: P33FJ32MC202
	//     Device clock: 010.000000 MHz
	//     Sampling Frequency: 1000 Hz
	// Filter setup:
	//     Filter kind: FIR
	//     Filter type: Lowpass filter
	//     Filter order: 30
	//     Filter window: Rectangular
	//     Filter borders:
	//       Wpass:30 Hz
	const unsigned BUFFFER_SIZE  = 32;
	const unsigned FILTER_ORDER  = 30;

	const unsigned COEFF_B[FILTER_ORDER+1] = {
	    0x01AE, 0x02CE, 0x03FF, 0x053B, 0x067E, 0x07C0,
	    0x08FC, 0x0A2A, 0x0B46, 0x0C4A, 0x0D2F, 0x0DF2,
	    0x0E8E, 0x0F00, 0x0F45, 0x0F5C, 0x0F45, 0x0F00,
	    0x0E8E, 0x0DF2, 0x0D2F, 0x0C4A, 0x0B46, 0x0A2A,
	    0x08FC, 0x07C0, 0x067E, 0x053B, 0x03FF, 0x02CE,
	    0x01AE};

	unsigned inext;                       // Input buffer index
	ydata unsigned input[BUFFFER_SIZE];   // Input buffer, must be in Y data space

	void config_adc()  {
	    ADPCFG = 0xFFF7; // La entrada analogica es el AN3 (pin 5)
	    // Con cero se indica entrada analogica y con 1 sigue siendo entrada digital.

	    AD1CON1bits.ADON = 0;  // ADC apagado por ahora
	    AD1CON1bits.AD12B = 0;  // ADC de 10 bits
	    AD1CON1bits.FORM = 0b00;  // Formato de salida entero

	    // Tomar muestras en forma manual, porque lo vamos a controlar con el Timer 2
	    AD1CON1bits.SSRC = 0b000;

	    // Adquiere muestra cuando el SAMP se pone en 1. SAMP lo controlamos desde el Timer 2
	    AD1CON1bits.ASAM = 0;

	    AD1CON2bits.VCFG = 0b011;  // Referencia con fuente externa VRef+ y VRef-
	    AD1CON2bits.SMPI = 0b0000;  // Lanza interrupción luego de tomar n muestras.
	    // Con SMPI=0b0000 -> 1 muestra ; Con SMPI=0b0001 -> 2 muestras ; Con SMPI=0b0010 -> 3 muestras ; etc.

	    // AD1CON3 no se usa ya que usamos muestreo manual

	    // Muestreo la entrada analogica AN3 contra el nivel de VRef+ y VRef-
	    AD1CHS0 = 0b00011;
	}

	void config_timer2()  {
	    // Prescaler 1:1   -> TCKPS = 0b00 -> Incrementa 1 en un ciclo de instruccion
	    // Prescaler 1:8   -> TCKPS = 0b01 -> Incrementa 1 en 8 ciclos de instruccion
	    // Prescaler 1:64  -> TCKPS = 0b10 -> Incrementa 1 en 64 ciclos de instruccion
	    // Prescaler 1:256 -> TCKPS = 0b11 -> Incrementa 1 en 256 ciclos de instruccion
	    T2CONbits.TCKPS = 0b00;

	    // Empieza cuenta en 0
	    TMR2=0;

	    // Cuenta hasta 5000 ciclos y dispara interrupcion
	    PR2=5000;  // 5000 * 200 nseg = 1 mseg   ->  1 / 1mseg = 1000Hz
	}

	void config_ports()  {
	    TRISAbits.TRISA3 = 1;  // Entrada para muestrear = AN3

	    // Elegimos los puertos RB2-RB11 para la salida digital
	    TRISBbits.TRISB2 = 0;  // Menos significativo
	    TRISBbits.TRISB3 = 0;
	    TRISBbits.TRISB4 = 0;
	    TRISBbits.TRISB5 = 0;
	    TRISBbits.TRISB6 = 0;
	    TRISBbits.TRISB7 = 0;
	    TRISBbits.TRISB8 = 0;
	    TRISBbits.TRISB9 = 0;
	    TRISBbits.TRISB10 = 0;
	    TRISBbits.TRISB11 = 0;  // Mas significativo

	    TRISBbits.TRISB0 = 1;  // Para control del filtro

	    TRISBbits.TRISB13 = 0;  // Debug T2
	    TRISBbits.TRISB14 = 0;  // Debug ADC
	}

	void detect_timer2() org 0x0022  {
	    IFS0bits.T2IF=0;  // Borramos la bandera de interrupción Timer 2

	    LATBbits.LATB13 = !LATBbits.LATB13;  // Para debug de la interrupcion Timer 2

	    AD1CON1bits.DONE = 0;  // Antes de pedir una muestra ponemos en cero
	    AD1CON1bits.SAMP = 1;  // Pedimos una muestra

	    asm nop;  // Tiempo que debemos esperar para que tome una muestra

	    AD1CON1bits.SAMP = 0;  // Pedimos que retenga la muestra
	}

	void detect_adc() org 0x002e  {
	    unsigned CurrentValue;

	    IFS0bits.AD1IF = 0; // Borramos el flag de interrupciones del ADC
	    LATBbits.LATB14 = !LATBbits.LATB14;  // Para debug de la interrupcion ADC

	    if(PORTBbits.RB0 == 1)  {
	        input[inext] = ADCBUF0;                 // Fetch sample

	        CurrentValue = FIR_Radix(FILTER_ORDER+1,// Filter order
	                                 COEFF_B,      // b coefficients of the filter
	                                 BUFFFER_SIZE, // Input buffer length
	                                 input,        // Input buffer
	                                 inext);       // Current sample

	        inext = (inext+1) & (BUFFFER_SIZE-1);   // inext = (inext + 1) mod BUFFFER_SIZE;

	        LATBbits.LATB11 =   ((unsigned int)CurrentValue & 0b0000001000000000) >> 9;
	        LATBbits.LATB10 =   ((unsigned int)CurrentValue & 0b0000000100000000) >> 8;
	        LATBbits.LATB9 =  ((unsigned int)CurrentValue &  0b0000000010000000) >> 7;
	        LATBbits.LATB8 =  ((unsigned int)CurrentValue &  0b0000000001000000) >> 6;
	        LATBbits.LATB7 =  ((unsigned int)CurrentValue &  0b0000000000100000) >> 5;
	        LATBbits.LATB6 =  ((unsigned int)CurrentValue &  0b0000000000010000) >> 4;
	        LATBbits.LATB5 = ((unsigned int)CurrentValue &  0b0000000000001000) >> 3;
	        LATBbits.LATB4 = ((unsigned int)CurrentValue &  0b0000000000000100) >> 2;
	        LATBbits.LATB3 = ((unsigned int)CurrentValue &  0b0000000000000010) >> 1;
	        LATBbits.LATB2 = ((unsigned int)CurrentValue &  0b0000000000000001) >> 0;
	    }
	    else  {
	        // Almacenamos los 10 bits del ADC
	        LATBbits.LATB2 = ADCBUF0.B0;
	        LATBbits.LATB3 = ADCBUF0.B1;
	        LATBbits.LATB4 = ADCBUF0.B2;
	        LATBbits.LATB5 = ADCBUF0.B3;
	        LATBbits.LATB6 = ADCBUF0.B4;
	        LATBbits.LATB7 = ADCBUF0.B5;
	        LATBbits.LATB8 = ADCBUF0.B6;
	        LATBbits.LATB9 = ADCBUF0.B7;
	        LATBbits.LATB10 = ADCBUF0.B8;
	        LATBbits.LATB11 = ADCBUF0.B9;
	    }
	}

	int main()  {
	    config_ports();
	    config_timer2();
	    config_adc();

	    // Habilitamos interrupción del ADC y lo encendemos
	    IEC0bits.AD1IE = 1;
	    AD1CON1bits.ADON = 1;

	    // Habilita interrupción del Timer 2 y lo iniciamos para que comience a contar
	    IEC0bits.T2IE=1;
	    T2CONbits.TON=1;

	    while(1)  {  }

	    return 0;
	}

**Ejemplo para dsPIC30F4013 para Placa**

.. code-block:: c
	
	const unsigned BUFFFER_SIZE  = 32;
	const unsigned FILTER_ORDER  = 64;

	const unsigned COEFF_B[FILTER_ORDER+1] = {
	    0xFD94, 0xFDE0, 0x0000, 0x0246, 0x02C5, 0x00EF,
	    0xFE28, 0xFCBE, 0xFE01, 0x0118, 0x0386, 0x0324,
	    0x0000, 0xFC88, 0xFBB2, 0xFE85, 0x02FE, 0x056F,
	    0x036C, 0xFE10, 0xF98B, 0xFA02, 0x0000, 0x0753,
	    0x09B0, 0x0399, 0xF804, 0xEFB4, 0xF407, 0x0865,
	    0x26C0, 0x41ED, 0x4CCD, 0x41ED, 0x26C0, 0x0865,
	    0xF407, 0xEFB4, 0xF804, 0x0399, 0x09B0, 0x0753,
	    0x0000, 0xFA02, 0xF98B, 0xFE10, 0x036C, 0x056F,
	    0x02FE, 0xFE85, 0xFBB2, 0xFC88, 0x0000, 0x0324,
	    0x0386, 0x0118, 0xFE01, 0xFCBE, 0xFE28, 0x00EF,
	    0x02C5, 0x0246, 0x0000, 0xFDE0, 0xFD94};

	unsigned inext;                       // Input buffer index
	ydata unsigned input[BUFFFER_SIZE];   // Input buffer, must be in Y data space

	void  detectarIntADC()  org 0x002a  {
	    unsigned CurrentValue;

	    IFS0bits.ADIF = 0; // Borramos el flag de interrupciones del ADC
	    LATFbits.LATF1 = !LATFbits.LATF1;  // Para debug de la interrupcion ADC

	    if(PORTFbits.RF4 == 1)  {
	        LATFbits.LATF5 = 1;  // Filtro no aplicado

	        input[inext] = ADCBUF0;                  // Fetch sample

	        CurrentValue = FIR_Radix(FILTER_ORDER+1, // Filter order
	                                 COEFF_B,        // b coefficients of the filter
	                                 BUFFFER_SIZE,   // Input buffer length
	                                 input,          // Input buffer
	                                 inext);         // Current sample

	        inext = (inext+1) & (BUFFFER_SIZE-1);    // inext = (inext + 1) mod BUFFFER_SIZE;

	        LATBbits.LATB8 =   ((unsigned int)CurrentValue & 0b0000001000000000) >> 9;
	        LATBbits.LATB9 =   ((unsigned int)CurrentValue & 0b0000000100000000) >> 8;
	        LATBbits.LATB10 = ((unsigned int)CurrentValue &  0b0000000010000000) >> 7;
	        LATBbits.LATB11 = ((unsigned int)CurrentValue &  0b0000000001000000) >> 6;
	        LATBbits.LATB12 = ((unsigned int)CurrentValue &  0b0000000000100000) >> 5;
	        LATCbits.LATC13 = ((unsigned int)CurrentValue &  0b0000000000010000) >> 4;
	        LATCbits.LATC14 = ((unsigned int)CurrentValue &  0b0000000000001000) >> 3;
	        LATDbits.LATD0 =  ((unsigned int)CurrentValue &  0b0000000000000100) >> 2;
	        LATDbits.LATD1 =  ((unsigned int)CurrentValue &  0b0000000000000010) >> 1;
	        LATDbits.LATD2 =  ((unsigned int)CurrentValue &  0b0000000000000001) >> 0;
	    }
	    else  {
	        LATFbits.LATF5 = 0;  // Filtro no aplicado

	        LATBbits.LATB8 = ADCBUF0.B9;
	        LATBbits.LATB9 = ADCBUF0.B8;
	        LATBbits.LATB10 = ADCBUF0.B7;
	        LATBbits.LATB11 = ADCBUF0.B6;
	        LATBbits.LATB12 = ADCBUF0.B5;
	        LATCbits.LATC13 = ADCBUF0.B4;
	        LATCbits.LATC14 = ADCBUF0.B3;
	        LATDbits.LATD0 = ADCBUF0.B2;
	        LATDbits.LATD1 = ADCBUF0.B1;
	        LATDbits.LATD2 = ADCBUF0.B0;
	    }
	}

	void detectarIntT2() org 0x0020  {
	    IFS0bits.T2IF=0;  //borra bandera de interrupcion de TIMER2

	    LATFbits.LATF0 = !LATFbits.LATF0;

	    ADCON1bits.SAMP=1; //pedimos muestras
	    asm nop;  //ciclo instruccion sin operacion
	    ADCON1bits.SAMP=0;  //retener muestra e inicia conversion
	}

	void configADC()  {
	    ADPCFG = 0b111011;  // elegimos AN2 como entrada para muestras
	    ADCHS = 0b0010; // usamos AN2 para recibir las muestras en el ADC
	    ADCON1bits.SSRC = 0b000; // muestreo manual
	    ADCON1bits.ADON = 0;  // apagamos ADC
	    ADCON2bits.VCFG = 0b000;  // tension de referencia 0 y 3.3
	    IEC0bits.ADIE=1;  // habilitamos interrupcion del ADC
	}

	void configTIMER2()  {
	    T2CON = 0x0000;   //registro de control de TIMER2 a cero
	    T2CONbits.TCKPS = 0b00; // prescaler = 1
	    TMR2 = 0;  // desde donde va a arrancar la cuenta
	    PR2 = 1250;   // hasta donde cuenta segun calculo para disparo de TIMER2
	    IEC0bits.T2IE = 1; // habilitamos interrupciones para TIMER2
	}

	void configPuertos()  {
	    // 10 bits de salida
	    TRISBbits.TRISB8 = 0;
	    TRISBbits.TRISB9 = 0;
	    TRISBbits.TRISB10 = 0;
	    TRISBbits.TRISB11 = 0;
	    TRISBbits.TRISB12 = 0;
	    TRISCbits.TRISC13 = 0;
	    TRISCbits.TRISC14 = 0;
	    TRISDbits.TRISD0 = 0;
	    TRISDbits.TRISD1 = 0;
	    TRISDbits.TRISD2 = 0;

	    TRISBbits.TRISB2 = 1;  // AN2

	    TRISFbits.TRISF0 = 0;  // Debug T2
	    TRISFbits.TRISF1 = 0;  // Debug ADC

	    TRISFbits.TRISF4 = 1;  // Filtro y no filtro

	    TRISFbits.TRISF5 = 0;  // Led indicador de filtro aplicado
	}

	void main()  {
	    configPuertos();
	    configTIMER2();
	    configADC();

	    ADCON1bits.ADON = 1;

	    T2CONbits.TON=1;

	    while(1)  {
	    }
	}







