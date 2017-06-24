## UART Task

UART is a great way to communicate to other devices and the outside world, but can sometimes be a pain to get working in the embedded systems world. Thankfully TI's RTOS and driver libraries take almost all the pain away! [(Documentation for TI UART Driver)](http://software-dl.ti.com/dsps/dsps_public_sw/sdo_sb/targetcontent/tirtos/2_14_03_28/exports/tirtos_full_2_14_03_28/docs/doxygen/html/_u_a_r_t_8h.html)

At this point I assume you, the reader, are a master of TI-RTOS Tasks after reading the previous chapter "Tasks." We will build off of that knowledge and only focus on the new stuff here. Let's jump in.

```c
/* TI-RTOS Header files */
#include <ti/drivers/UART.h>
```
First, lets include `UART.h` from the driver library so we can make use of all the useful functions it has.

Next, let's define a simple task that creates some simple local vars, configures UART, prints a message, then repeats whatever it receives back to the sender.

```c
Void uartEchoFxn(UArg arg0, UArg arg1)
{
    //...SNIP...
    char input;
    UART_Handle uart;
    UART_Params uartParams;
    const char echoPrompt[] = "\fHello World!\r\n";
    ///...SNIP...
}
```
A few local definitions will keep track of our UART device, what the character received was, and a message for us to send once.

```c
Void uartEchoFxn(UArg arg0, UArg arg1)
{
    ///...SNIP...
    // Create a UART with data processing off.
    UART_Params_init(&uartParams);
    uartParams.writeDataMode = UART_DATA_BINARY;
    uartParams.readDataMode = UART_DATA_BINARY;
    uartParams.readReturnMode = UART_RETURN_FULL;
    uartParams.readEcho = UART_ECHO_OFF;
    uartParams.baudRate = 9600;
    uart = UART_open(Board_UART0, &uartParams);

    if (uart == NULL) {
        System_abort("Error opening the UART");
    }
    ///...SNIP...
}
```

Next we set settings for UART. You can read the [Documentation](http://software-dl.ti.com/dsps/dsps_public_sw/sdo_sb/targetcontent/tirtos/2_14_03_28/exports/tirtos_full_2_14_03_28/docs/doxygen/html/_u_a_r_t_8h.html) if you want to understand what each setting is, but the primary one to be interested in is the BaudRate.

Finally, we open our UART device with `UART_open(Board_UART0, &uartParams);`

```c
Void uartEchoFxn(UArg arg0, UArg arg1)
{
    UART_write(uart, echoPrompt, sizeof(echoPrompt));

    //Loop forever echoing
    while (1) {
        UART_read(uart, &input, 1);
        UART_write(uart, &input, 1);
    }
}
```
With the UART device open, we can now read from it and write to it. Let's quickly print our "Hello World" message with `UART_write(uart, echoPrompt, sizeof(echoPrompt));`

Then we will just simply read one character and immediately write it back.

```c
Void uartEchoFxn(UArg arg0, UArg arg1)
{
    char input;
    UART_Handle uart;
    UART_Params uartParams;
    const char echoPrompt[] = "\fHello World!\r\n";

    // Create a UART with data processing off.
    UART_Params_init(&uartParams);
    uartParams.writeDataMode = UART_DATA_BINARY;
    uartParams.readDataMode = UART_DATA_BINARY;
    uartParams.readReturnMode = UART_RETURN_FULL;
    uartParams.readEcho = UART_ECHO_OFF;
    uartParams.baudRate = 9600;
    uart = UART_open(Board_UART0, &uartParams);

    if (uart == NULL) {
        System_abort("Error opening the UART");
    }

    UART_write(uart, echoPrompt, sizeof(echoPrompt));

    // Loop forever echoing
    while (1) {
        UART_read(uart, &input, 1);
        UART_write(uart, &input, 1);
    }
}
```

Here is the full function for reference.


```c
int main(){
  //... SNIP ...
  Board_initGeneral();
  Board_initUART();
  //... SNIP ...
  //SCHEDULE YOUR TASK!
  //START THE RTOS
}

```
And as always... don't forget to create and schedule your task!

Next we will talk about inter-task communication with mailboxes!  
