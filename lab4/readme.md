# Lab 4: In a galaxy far far away... RTOS!

## Overview

Hello Team ECEN 3360 and welcome to Lab 4!

In our previous lab we learned to write recursive programs in ARM assembly,
interface C with assembly, operate LEDs, and use them to output meaningful
data to the end user.

In this lab we get to elevate ourselves up the stack to using a Real Time
Operating System (RTOS). This allows us to define multiple tasks
for our system to preform and not worry about the resource allocation ourselves.
Most professional embedded systems development will take place on an RTOS.

## Deliverable Requirements

- The system _MUST_ have at least two Tasks. One for light control, one for UART.
- The system _MUST_ communicate between the tasks via mailboxes.
- The system _MUST_ toggle the _red_ LED when the string "red" is received
- The system _MUST_ toggle the _blue_ LED when the string "blue" is received
- The system _MUST_ toggle the _green_ LED when the string "green" is received
- The system _MUST_ turn on all LEDs when the string "on" is received
- The system _MUST_ turn off all LEDS when the string "off" is received
- Tasks _MUST NOT_ communicate with each other using global variables or shared memory.


## Material

- [What is an RTOS? (external link)](https://en.wikipedia.org/wiki/Real-time_operating_system)
- [Scheduling (external link)](https://en.wikipedia.org/wiki/Scheduling_(computing))
- [Tasks](tasks.md)
- [Using UART in a Task](uart.md)
- [Inter-Task Communication with Mailboxes](mailboxes.md)
