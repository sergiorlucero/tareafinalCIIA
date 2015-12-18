/* Copyright 2014, Mariano Cerdeiro
 * Copyright 2014, Pablo Ridolfi
 * Copyright 2014, Juan Cecconi
 * Copyright 2014, Gustavo Muro
 *
 * This file is part of CIAA Firmware.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 *    this list of conditions and the following disclaimer in the documentation
 *    and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 *    contributors may be used to endorse or promote products derived from this
 *    software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
 * POSSIBILITY OF SUCH DAMAGE.
 *
 */

/** \brief teclas source file
 **
 ** This is a mini example of the CIAA Firmware.
 **
 **/

/** \addtogroup CIAA_Firmware CIAA Firmware
 ** @{ */
/** \addtogroup Examples CIAA Firmware Examples
 ** @{ */
/** \addtogroup Blinking Blinking_echo example source file
 ** @{ */

/*
 * Initials     Name
 * ---------------------------
 * PS           Pablo Solivellas
 * HP           Hernan Ponso
 * DP           Damian Primo
 *
 */

/*
 * modification history (new versions first)
 * -----------------------------------------------------------
 * 20151201 v0.0.1   PS   first functional version
 */

/*==================[inclusions]=============================================*/
#include "tareafinal.h"           /* <= own header */

/*==================[macros and definitions]=================================*/
//#define TECLADO_TOTAL_TECLAS     4

/*==================[internal data declaration]==============================*/

/*==================[internal functions declaration]=========================*/

/*==================[internal data definition]===============================*/

/** \brief File descriptor for digital input ports
 *
 * Device path /dev/dio/in/0
 */
static int32_t fd_out, fd_in;
uint8_t led, teclas, ledBlanco;
int cont, freq, contB;
static uint8_t estadoTeclas, teclasFlancoUP;

/*==================[external data definition]===============================*/

/*==================[internal functions definition]==========================*/

/*==================[external functions definition]==========================*/

int main(void)
{
   /* Starts the OS in the Application Mode 1 */
   /* This example has only one Application Mode */
   StartOS(AppMode1);

   /* StartOs shall never returns, but to avoid compiler warnings or errors
    * 0 is returned */
   return 0;
}

/** \brief Error Hook function
 *
 * This function is called from the os if an os interface (API) returns an
 * error. Is for debugging proposes. If called this function triggers a
 * ShutdownOs which ends in a while(1).
 *
 * The values:
 *    OSErrorGetServiceId
 *    OSErrorGetParam1
 *    OSErrorGetParam2
 *    OSErrorGetParam3
 *    OSErrorGetRet
 *
 * will provide you the interface, the input parameters and the returned value.
 *
 */
void ErrorHook(void){
   ciaaPOSIX_printf("ErrorHook was called\n");
   ciaaPOSIX_printf("Service: %d, P1: %d, P2: %d, P3: %d, RET: %d\n", OSErrorGetServiceId(), OSErrorGetParam1(), OSErrorGetParam2(), OSErrorGetParam3(), OSErrorGetRet());
   ShutdownOS(0);
}

void blink(uint8_t valor){
   uint8_t outputs;

   /* read selected led */
   ciaaPOSIX_read(fd_out, &outputs, 1);
   //Blink selected led!
   outputs ^= valor; //or logico
   ciaaPOSIX_write(fd_out, &outputs, 1);
}

/*Funcion que modifica el movimiento secuencial de los leds */

void procesarLeds(char mov)
{
 switch(mov){
    case 'l':
      switch (led){
        case VERDE: led = ROJO;
        break;
        case ROJO: led = AMARILLO;
        break;
        case AMARILLO: led = RGBVERDE;
        break;
        case RGBVERDE: led = RGBROJO;
        break;
        case RGBROJO: led = RGBAZUL;
        break;
        case RGBAZUL: led = VERDE;
        break; 
      }
    break;
    case 'r':
      switch (led){
        case VERDE: led = RGBAZUL;
        break;
        case RGBAZUL: led = RGBROJO;
        break;
        case RGBROJO: led = RGBVERDE;
        break;
        case RGBVERDE: led = AMARILLO;
        break;
        case AMARILLO: led = ROJO;
        break;
        case ROJO: led = VERDE;
        break; 
      }
    break;
  }
  ciaaPOSIX_write(fd_out, &led, 1);
}

/* Funcion que modifica la frecuencia del blinking del led */
void modiffreq(char pulso){
  switch(pulso){
    case 'd':
      if (freq >= (MINFREQ + INCFREQ)) 
        freq = freq - INCFREQ;
      else{ 
        freq = MINFREQ;
        blink(RGBBLANCO); 
        ledBlanco=1;     
      }
    break;
    case 'u':
      if (freq < (MAXFREQ - INCFREQ)) 
        freq = freq + INCFREQ;
      else{ 
        freq = MAXFREQ;
        blink(RGBBLANCO);
	ledBlanco=1;
      }
    break;
  }
}

void procesarTeclas(uint8_t teclas){
   GetResource(BLOCK);

   // tecla 1 y 2 cambia el led, el RGB pasa por todos, menos el blanco
   if (TECLA1 & teclas){
      procesarLeds('l');
   }

   if (TECLA2 & teclas){
      procesarLeds('r');
   }

   // tecla 3 y 4 cambia la frecuecia, si llega al minimo o maximo, prende blanco
   if (TECLA3 & teclas){
      modiffreq('d');
   }

   // Led blinking frecuency increment
   if (TECLA4 & teclas){
      modiffreq('u');
   }

   ReleaseResource(BLOCK);
}

void teclado_task(void){
   uint8_t inputs;

   /* lee el estado de las entradas */
   ciaaPOSIX_read(fd_in, &inputs, 1);

   /* detecta flanco ascendente */
   teclasFlancoUP |= (inputs ^ estadoTeclas) & inputs;

   /* guarda el nuevo estado de las teclas */
   estadoTeclas = inputs;
}

uint8_t teclado_getFlancos(void)
{
   uint8_t ret = teclasFlancoUP;

   teclasFlancoUP = 0;

   return ret;
}

/* tratamiento leds */
void tratar_Leds(void){

   /* tratamiento leds seleccionados */
   if (cont >= freq)
   {  
      //Blink selected led!
      blink(led);
      cont = 0;
   }
   else
   {
      cont += 10;
   }
  
   /* tratamiento led Blanco */  
   if (ledBlanco==1) 
   {
    if (contB >= 100)
    {  
      //Blink led Blanco!
      blink(RGBBLANCO);
      contB = 0;
      ledBlanco=0;
    }
    else
    {
      contB += 10;
    }
   }
}

/** \brief Initial task
 *
 * This task is started automatically in the application mode 1.
 */
TASK(InitTask){
   uint8_t outputs;
   /* init CIAA kernel and devices */
   ciaak_start();

   /* print message (only on x86) */
   ciaaPOSIX_printf("Init Task...\n");

   /* open CIAA digital outputs */
   fd_out = ciaaPOSIX_open("/dev/dio/out/0", ciaaPOSIX_O_RDWR);
   fd_in = ciaaPOSIX_open("/dev/dio/in/0", ciaaPOSIX_O_RDWR);

   //led = 0B100000;
   led = VERDE;
   freq = 200;   
   cont = 0;

   //Write the led register
   ciaaPOSIX_write(fd_out, &led, 1);

   //no hay tareas ya que arrancan solas
  
   /* terminate task */
   TerminateTask();
}


TASK(LedsTask){
   //
   uint8_t outputs; 

   tratar_Leds();

   TerminateTask();
}


TASK(TecladoTask){

   teclado_task();

   // lee los flancos de las teclas
   teclas = teclado_getFlancos();

   procesarTeclas(teclas);

   TerminateTask();
}

/** @} doxygen end group definition */
/** @} doxygen end group definition */
/** @} doxygen end group definition */
/*==================[end of file]============================================*/

