
## Tasks

Every system that shares and schedules resource use between multiple actions or _threads_ of execution must have a structure to represent each one. (Remember scheduling? If not, check out the previous chapter!)

In the TI RTOS these are aptly named *Tasks*.

Each task represents a program that wishes to be scheduled for time to run on the micro-controller. Each can be thought of as it's own program; unaware of the others that are also operating on the device. Ideally, we would like to keep it this way and use only the RTOS to pass information and signals back and forth. We will demonstrate how to do this in a later document (mailboxes).


To get a task running, we require, at minimum:
- A function pointer to the Task body
- A pointer to pre-allocated stack
- A parameter struct to configure the task

To accomplish these requirements, we have to add things in a few areas of our program. Lets break it down by sections.



```c
/* BIOS Header files */
#include <ti/sysbios/BIOS.h>
#include <ti/sysbios/knl/Task.h>
/* TI-RTOS Header files */
#include <ti/drivers/GPIO.h>
```
In the body of our file, we have to first include a few headers so that we can use BIOS and task functions. Once included we can now use the functions listed in the [documentation for Tasks](http://software-dl.ti.com/dsps/dsps_public_sw/sdo_sb/targetcontent/sysbios/6_32_05_54/exports/bios_6_32_05_54/docs/cdoc/ti/sysbios/knl/Task.html)

```c
#define TASKSTACKSIZE   768
Char taskStack[TASKSTACKSIZE];
```
We can now turn our sights to creating a stack for the task. (Remember! __Each__ task __must__ have it's own stack!!) For simplicity, we have defined __TASKSTACKSIZE__ to be 768 bytes
then used it to initialize our taskStack array.

How big should your stack be? Great question! When configuring the stack size for each of your tasks you should be aware of the operations of your task. Does it allocate many things onto the stack? If so, can you calculate what the maximum stack usage will be? If you can't, that's a big red flag! This means that your task is unbounded and that in some edge condition (possibly caused by the user!) an unknown amount of memory can be allocated. This is a security no no!

```c
Task_Struct taskStruct;
```
With the stack created we must define a handle for our task. This structure will be used by you and the RTOS to keep track of the task, it's state and perform further actions on it if needed.


Now we are ready to define our exciting first task!!

Lets start small, with a function that blinks __Board_LED0__ at a frequency given by an input argument.

To define a Task, we can use the function prototype: `Void taskNameFxn(UArg arg0, UArg arg1);`

(Notice that it is simply a function that takes two arguments of type `UArg` and returns `void`. A `Uarg` is simply a generic type that you will use to pass arguments, casing to and from your original type.)

```c
Void taskFxn(UArg arg0, UArg arg1)
{
  while (1) {
      Task_sleep((UInt)arg0);
      GPIO_toggle(Board_LED0);
  }
}
```
Here we have defined the function that our task will run. There are a few very interesting things to take note of here. Lets list them then dive a bit deeper into each of them.
- The function contains `while(1)` (infinite Loop)
- The function `Task_Sleeep(...)` is used.
- arg0 is casted into a `Uint`

Recall that the purpose of organizing our system into Tasks is so that we may have many of them operating in unison. If an infinite loop (`while(1)`) is part of a task, how do any others ever get a turn?! This goes back to our chapter on _scheduling_. The RTOS is periodically taking back control from the task, assessing what should be run next, then executing that. Even though the task  itself will loop forever, other tasks still get a turn.

To help the RTOS out, our task lets the RTOS know that it has nothing to do for a given (arg0) amount of time. It does this with the `Task_sleep(...)` function. Instead of simply wasting all of the time the RTOS gives our task, thus using power in the process, we instead kindly tell the RTOS that we will not be requiring any processor time for a given period. This is important for power saving, and resource management; allowing tasks that require the time to be more likely to be scheduled.

With our task's function, stack, and struct defined, lets jump into our `int main(void)` and get it constructed.

```c
int main(void) {
    Board_initGeneral();
    Board_initGPIO(); //We are using LEDs
    //...SNIP...
}
```

First thing we must do is to ask the RTOS to initialize the board and any devices we are using. (This must only be performed once, not per task.)

```c
int main(void) {
    //...SNIP...
    Task_Params taskParams;
    Task_Params_init(&taskParams);
    taskParams.arg0 = (UArg)1000;
    taskParams.stackSize = TASKSTACKSIZE;
    taskParams.stack = &taskStack;
    //...SNIP...
}
```
Now we create our task parameters in a variable of type `Task_Params`. Immediately after initialization, we use `Task_Params_init(&taskParams)` to set all of the default values. We then must configure the __stack size__ and the __stack pointer__ to what we initialized above.

__NOTE:__ Both the taskStack and the taskParams were passed by reference here. If you're not sure what that means, please chat with your professor, TA, or consult the all powerful ~~Bing~~ ~~Google~~ DuckDuckGo.


```c
int main(void) {
    //...SNIP...
    Task_construct(&taskStruct, (Task_FuncPtr)taskFxn, &taskParams, NULL);

    BIOS_start();
    //...SNIP...
}
```
Finally, we can ask the RTOS to construct and schedule our Task.

We do this with `Void 	
Task_construct(Task_Struct *structP, Task_FuncPtr fxn, const Task_Params *params, Error_Block *eb);`. I know what you're thinking... "ICK! That's a scary function prototype." Let's break it down. The first argument is simply the task structure we created in the beginning. The second, the function we want the task to execute. The third, the parameters structure we created and fourth, an error handler, which we will leave blank for now.  

Put all of this together and you have your first task! A blinking light!

Next we will create a task to talk to the outside world via UART.
