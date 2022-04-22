# ADC and UART serial communication


### Learning objectives

After completing this project i learnt :

* Understand the analog-to-digital conversion process
* How to use internal ADC unit
* Understand the UART communication
* Use functions from UART library

The purpose of the  project is to understand analog-to-digital number conversion and the use of an internal 8-channel 10-bit AD converter. Another goal is to understand serial asynchronous communication, data frame structure and communication options using an internal USART unit.



## Preparation tasks (done before the project)

1. Use schematic of the LCD keypad shield and find out the connection of five push buttons: Select, Left, Up, Down, and Right.

   &nbsp;

   &nbsp;

   &nbsp;

   &nbsp;

   &nbsp;

   &nbsp;

2. According to the connection, calculate the voltage values on pin PC0[A0] if one button is pressed at a time. In this case, the voltage on the pin is given by the [voltage divider](https://www.alprojectoutcircuits.com/tools/voltage-divider-calculator/), where resistors R3, R4, R5 and R6 are applied successively.

   ![Equation: Voltage divider](images/eq_divider1.png)

   ![Equation: Voltage divider](images/eq_divider2.png)

   &nbsp;

   ![Equation: Voltage divider](images/eq_divider3.png)

   &nbsp;

   ![Equation: Voltage divider](images/eq_divider4.png)

   &nbsp;

   ![Equation: Voltage divider](images/eq_divider5.png)

   &nbsp;

3. Calculate the voltage value if none of the push buttons is pressed.

   ![Equation: Voltage divider](images/eq_divider6.png)

   &nbsp;

4. Calculate the ADC values for these voltages according to the following equation if reference is Vref=5V and number of bits for analog to digital conversion is n=10.

   Equation: ADC conversion

   | **Push button** | **PC0[A0] voltage** | **ADC value (calculated)** | **ADC value (measured)** |
   | :-: | :-: | :-: | :-: |
   | Right  | 0&nbsp;V | 0   |  |
   | Up     | 0.495&nbsp;V | 101 |  |
   | Down   |       |     |  |
   | Left   |       |     |  |
   | Select |       |     |  |
   | none   |       |     |  |



## Part 2: Analog-to-Digital Conversion

We live in an analog world, surrounded by digital devices. Everything we see, feel or measure is analog in nature such as light, temperature, speed, pressure etc. It is obvious that we need something that could convert these analog parameters to digital value for a microcontroller or micro-processor to understand it.

An [Analog to Digital Converter](https://components101.com/articles/analog-to-digital-adc-converters) (ADC) is a circuit that converts a continuous voltage value (analog) to a binary value (digital) that can be understood by a digital device which could then be used for digital computation. These ADC circuits can be found as an individual ADC ICs by themselves or embedded into a modern microcontroller.

The internal ADC module of ATmega328P can be used in relatively slow and not extremely accurate data acquisitions. But it is a good choice in most situations, like reading sensor data or reading waveforms.

AVR ADC module has 10-bit resolution with +/-2LSB accuracy. It means it returns a 10-bit integer value, i.e. a range of 0 to 1023. It can convert data at up to 76.9kSPS, which goes down when higher resolution is used. We mentioned that there are 8 ADC channels avaiprojectle on pins, but there are also three internal channels that can be selected with the multiplexer decoder. These are temperature sensor (channel 8), bandgap reference (1.1V) and GND (0V) [[4]](https://embedds.com/adc-on-atmega328-part-1/).

The operation with the AD converter is performed through ADMUX, ADCSRA, ADCL+ADCH, ADCSRB, and DIDR0 registers. See [ATmega328P datasheet](https://www.microchip.com/wwwproducts/en/ATmega328p) (**Analog-to-Digital Converter > Register Description**) and complete the following table.

   | **Operation** | **Register(s)** | **Bit(s)** | **Description** |
   | :-- | :-: | :-: | :-- |
   | Voltage reference    | ADMUX | REFS1:0 | 00: ..., 01: AVcc voltage reference (5V), ... |
   | Input channel        | ADMUX | MUX3:0 | 0000: ADC0, 0001: ADC1, ... |
   | ADC enable           | ADCSRA |  |  |
   | Start conversion     |  |  |  |
   | ADC interrupt enable |  |  |  |
   | ADC clock prescaler  |  | ADPS2:0 | 000: Division factor 2, 001: 2, 010: 4, ...|
   | ADC 10-bit result    |  |  |  |



```

### Both versions

1. Compile the template code and download to Arduino Uno board or load `*.hex` firmware to SimulIDE circuit (create an identical connection to the LCD keypad shield).

   ![SimulIDE](images/screenshot_simulide_lcd_probe.png)

2. In `main.c` configure ADC as follows:
   * voltage reference: AVcc with external capacitor
   * input channel: ADC0
   * clock prescaler: 128
   * enable ADC module
   * enable interrupt

   Use single conversion mode and start each conversion every second (use Timer/Counter1 overflow).

   Read the voltage level when a push button is pressed and display it in decimal at LCD display position `a`. Display the same value but in hexadecimal at position `b`. Note that you can use the 16-bit ADC variable--which is declared in the AVR library--to read the value from both converter registers ADCH:L.

   LCD-keypad shield

```c
/**********************************************************************
 * Function: ADC complete interrupt
 * Purpose:  Display value on LCD and send it to UART.
 **********************************************************************/
ISR(ADC_vect)
{
    uint16_t value = 0;
    char lcd_string[4] = "0000";

    value = ADC;                  // Copy ADC result to 16-bit variable
    itoa(value, lcd_string, 10);  // Convert decimal value to string

    // WRITE YOUR CODE HERE
}
```

3. Write the values to the table from Preparation tasks section and compare them with the calculated ones.

   SimulIDE

<a name="part3"></a>

## Part 3: UART communication

The UART (Universal Asynchronous Receiver-Transmitter) is not a communication protocol like SPI and I2C, but a physical circuit in a microcontroller, or a stand-alone integrated circuit, that translates communicated data between serial and parallel forms. It is one of the simplest and easiest method for implement and understanding.

In UART communication, two UARTs communicate directly with each other. The transmitting UART converts parallel data from a CPU into serial form, transmits it in serial to the receiving UART, which then converts the serial data back into parallel data for the receiving device. Only two wires are needed to transmit data between two UARTs. Data flows from the Tx pin of the transmitting UART to the Rx pin of the receiving UART [[6]](https://www.circuitbasics.com/basics-uart-communication/).

UARTs transmit data asynchronously, which means there is no external clock signal to synchronize the output of bits from the transmitting UART. Instead, timing is agreed upon in advance between both units, and special **Start** (log. 0) and 1 or 2 **Stop** (log. 1) bits are added to each data package. These bits define the beginning and end of the data packet so the receiving UART knows when to start reading the bits. In addition to the start and stop bits, the packet/frame also contains data bits and optional parity.

The amount of **data** in each packet can be set from 5 to 9 bits. If it is not otherwise stated, data is transferred least-significant bit (LSB) first.

**Parity** is a form of very simple, low-level error checking and can be Even or Odd. To produce the parity bit, add all 5-9 data bits and extend them to an even or odd number. For example, assuming parity is set to even and was added to a data byte `0110_1010`, which has an even number of 1's (4), the parity bit would be set to 0. Conversely, if the parity mode was set to odd, the parity bit would be 1.

One of the most common UART formats is called **9600 8N1**, which means 8 data bits, no parity, 1 stop bit and a symbol rate of 9600&nbsp;Bd.

UART frame 8N1

### Example of UART communication

> 
>

1. In the project, we are using [UART library](http://www.peterfleury.epizy.com/avr-software.html) developed by Peter Fleury. Use online manual of UART library and add the input parameters and description of the functions to the following table.

   | **Function name** | **Function parameter(s)** | **Description** | **Example** |
   | :-- | :-- | :-- | :-- |
   | `uart_init` | `UART_BAUD_SELECT(9600, F_CPU)` | Initialize UART to 8N1 and set baudrate to 9600&nbsp;Bd | `uart_init(UART_BAUD_SELECT(9600, F_CPU));` |
   | `uart_getc` |  |  |
   | `uart_putc` |  |  |
   | `uart_puts` |  |  |

2. Extend the application from the previous point and send information about the results of the analog to digital conversion to the UART transmitter. Use internal UART module in 9600 8N1 mode. If needed, define the CPU clock frequency in `main.c`:

```c
#ifndef F_CPU
# define F_CPU 16000000  // CPU frequency in Hz required for UART_BAUD_SELECT
#endif
```

### Version: SimulIDE

1. In SimulIDE, right click to ATmega328 package and select **Open Serial Monitor**. In this window you can receive data from the microcontroller, but also send them back.

   ![SimulIDE](6_Images and videos/screenshot_simulide_lcd_uart.png)


   ![SimulIDE](6_Images and videos/screenshot_simulide_lcd_final.png)

