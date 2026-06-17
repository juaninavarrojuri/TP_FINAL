# TP_FINAL
Primer línea agregada
# Sensor de Humedad de tierra
### Electrónica Digital II - Universidad Nacional de Córdoba

**Integrantes:**
* Delbon, Alejo
* Katz, Renata
* Navarro Juri, Juan Cruz

**Profesor:** Marcos Javier Blasco

---

## 1. Descripción General del Proyecto 

El sistema mide la conductividad eléctrica en la tierra para determinar el porcentaje de humedad presente en la misma. En base al valor medido, se puede implementar un sistema de riego automático mediante un relé o bien monitorear cómo varía la humedad en el tiempo. Este sistema va dirigido a cualquier usuario que necesite un control permanente sobre su suelo (sistemas controlados para viveros, aficionados a la botánica, cultivos, etc.).

### Alcances del Proyecto
El sistema **SÍ** es capaz de: Medir la conductividad eléctrica entre las sondas del sensor (relación inversamente proporcional entre caída de tensión y humedad en el suelo), este valor se presenta sobre un display 7 segmentos de 4 dígitos (muestra valores de 0% a 100%), activa un relé si el valor de humedad está por debajo del porcentaje umbral (60%) y transmite datos por comunicación serie por UART.

El sistema **NO** incluye: Almacenamiento local de datos (Data Logging) en tarjeta SD ni conectividad inalámbrica Wi-Fi/Bluetooth.

### Posibles Etapas Siguientes (Líneas Futuras)
Este desarrollo podría escalar a un ámbito profesional diseñando una app móvil o interfaz web para la visualización remota de las variables y modificación del valor umbral en base a la necesidad del usuario. A su vez se podría llevar a la realidad el sistema de riego, en lugar de la utilización del relé el cual simula este efecto.

Otro punto a implementar sería la migración de protoboard a PCB, para así lograr reducir el volumen del dispositivo resultando más práctico y robusto para su entorno de uso.  

Por último, se considera realizar un sensado menos frecuente, es decir tomar datos de humedad cada una hora, ya que no es un mensurando que presente cambios repentinos en un lapso de tiempo muy corto. Esto significa que en cada medición se energiza el sensor, adquiere el dato, lo envía a la interfaz y luego se desenergiza sin consumir corriente hasta la siguiente adquisición.

---

## 2. Arquitectura del Sistema: Hardware y Software 

### Hardware & Interconexión

![Diagrama de bloques del hardware](image_0.png)
*Imagen 1: Diagrama de bloques del hardware.*

![Esquemático del circuito desarrollado en Proteus Professional 8.13](image_b1689f.png)
*Imagen 2: Esquemático del circuito desarrollado en Proteus Professional 8.13*

#### Descripción del Circuito y Consideraciones de Diseño:
* **Etapa de adquisición de datos (Sensor):** El sensor resistivo YL69 mide la conductividad eléctrica del suelo. Su señal es acondicionada por el módulo YL38, utilizando la salida analógica (A0) para enviar un nivel de tensión proporcional a la humedad hacia el canal analógico AN0 (Pin RA0) del PIC16F887. Se seleccionó la salida analógica para obtener una resolución matemática precisa del porcentaje (0% a 100%) a través del ADC interno, en lugar de la salida digital por comparador.
* **Etapa de visualización (Display multiplexado):** Se implementó el multiplexado por división de tiempo de un display de 7 segmentos de 4 dígitos. Los segmentos (A-G y DP) comparten un puerto de salida (Puerto D) con sus respectivas resistencias de limitación de corriente (330 Ω). Los 4 dígitos comunes se controlan secuencialmente mediante transistores NPN (2N2222) configurados como interruptores de saturación/corte desde otro puerto (Puerto C).
* **Etapa de potencia y actuación (Relé):** Como la bobina del relé requiere una corriente elevada y genera picos inductivos, se necesita una etapa de acoplamiento. Se utiliza un transistor NPN (2N2222) en configuración de conmutación comandado desde un pin digital y un diodo de libre circulación (Flyback) en paralelo con la bobina del relé para proteger al transistor contra las sobretensiones destructivas producidas por la desactivación de la carga inductiva. Todo esto viene implementado físicamente en un módulo con el relé para su conexión directa a los pines del PIC16F8887.
* **Etapa de comunicación (UART):** Se utilizan los pines dedicados RC6/TX y RC7/RX del periférico EUSART del PIC para la transmisión de las tramas de datos hacia la PC. Dado que los niveles de tensión son TTL (0V - 5V), se prevé la interconexión directa a un puente USB-TTL para el monitoreo en la terminal serie.

Como consideraciones importantes de diseño, se destaca la gran cantidad de variables que pueden modificar la lectura entregada por el sensor. Al ser un sensor resistivo, cualquier cambio en la conductividad eléctrica de la tierra va a ser percibido como un cambio en la humedad. Por ello se debe estabilizar (o estandarizar):
* **Electrolitos presentes en tierra:** dependen de la tierra y del agua de riego.
* **Temperatura:** a mayor temperatura, la conductividad eléctrica será mayor.
* **Compactación de la tierra:** si la tierra está suelta, las burbujas de aire funcionan como aislante eléctrico.
* **Corrosión de las pistas:** alimentar con CC de forma constante va a producir la corrosión de los electrodos por electrólisis.
* **Posición del sensor:** debe posicionarse en una zona representativa.

### Arquitectura de Software (Firmware)

![Diagrama de flujo del Firmware](image_1.png)

---

## 3. Especificaciones Eléctricas, Alimentación y Entorno 

#### Parámetros de Alimentación y Consumo 
* **Tensión de operación del sistema:** 5V
* **Método de alimentación:** Alimentación por USB UART
* **Consumo estimado o medido:** 105-120 mA (relé, pines del PIC, sensor, display).
* **En modo bajo consumo:** 30 mA (pines del PIC, sensor, display).

#### Entorno y Herramientas
* **Herramientas de Software:** MPLAB X IDE [v5.35] y compilador XC8 [v5.87].
* **Hardware de Programación/Depuración:** EUSART.
* **Configuración de Bits (Fuses Críticos):** `__CONFIG _CONFIG1, _FOSC_XT & _WDTE_OFF & _PWRTE_OFF & _MCLRE_ON & _CP_OFF & _CPD_OFF & _BOREN_ON & _IESO_ON & _FCMEN_ON & _LVP_OFF`,  `__CONFIG _CONFIG2, _BOR4V_BOR40V & _WRT_OFF`
* **Oscilador:** Cristal externo de 4MHz.
* **Watchdog Timer (WDT):** OFF.
* **Master Clear (MCLRE):** ON (Botón externo).
* **Periféricos Internos Utilizados:** Timer0, ADC, EUSART.

La prioridad en interrupciones se dio a la interrupción externa (en RB0) que activa o desactiva el envío de datos a la PC, su prioridad se da por evaluar su flag antes que las demás. La otra interrupción que se implementó fue por Timer0 que permite el multiplexado de los displays, siendo esta la segunda en prioridad.

---

## 4. Proceso de Integración y Desarrollo 

* **Etapa 1:** Escritura de código en base a el objetivo planteado para el circuito, obtención de señales, envío de datos, etc.
* **Etapa 2:** Armado de hardware sin el sensor, simulando el mismo con un potenciómetro para poder plantear distintos valores de humedad frente al circuito, evaluar su funcionamiento y respuesta.
* **Etapa 3:** Una vez que se logró el comportamiento deseado en las simulaciones, se aplica el sensor real, con un valor umbral establecido según las necesidades de la tierra y las características particulares de la misma.
* **Etapa 4:** Corrección de errores y puesta en marcha final.

---

## 5. Ensayos, Pruebas y Resultados

Inicialmente se incluyó un potenciómetro como señal controlada de ingreso en lugar del sensor de humedad para poder simular todos los valores posibles de detección. Esto permitió observar el correcto funcionamiento del circuito activando el relé en valores de humedad debajo del umbral establecido. Al haber solucionado los errores presentes y comprobado el funcionamiento correcto, se implementó el sensor y se utilizó en tierra con diferentes cantidades de humedad presente.

Se verificó que el sensor expuesto al aire lee una humedad del 0% (sin contacto con la tierra, igual que en tierra muy seca) y al introducirlo en tierra saturada de agua lee una humedad del 100%, habiendo obtenido así datos empíricos del sistema que aseguran el comportamiento esperado.

![Hardware final armado en funcionamiento](image_2.png)
*Imagen 3: Hardware final armado en funcionamiento.*

![Captura del terminal serie con lecturas en tiempo real](image_3.png)
*Imagen 4: Captura del terminal serie con lecturas en tiempo real*
