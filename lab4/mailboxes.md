## Inter-Task Communication, Mailboxes

Now that we can communicate with the outside world with [UART](uart.md) and we
have a task that [blinks an LED](tasks.md) we now need a way to have them talk
to each other!

For this, we will use a TI-RTOS contract name Mailboxes. [(Mailbox Documentation)](http://software-dl.ti.com/dsps/dsps_public_sw/sdo_sb/targetcontent/sysbios/6_32_05_54/exports/bios_6_32_05_54/docs/cdoc/)

A mailbox does exactly what you think it would do! It stores a specific type and amount of messages that can be delivered and received by anyone that knows about the mailbox.  

Diving a bit further into the technical details, a mailbox is a thread-safe construct. This means that even though the kernel is switching between tasks, a mailbox can-not be damaged by two tasks trying to access it "at the same time". It does this by using locks and copying the content of each message into a kernel buffer. If you want to learn more about how mailboxes work, chat with your instructor as it is beyond the scope of this class.

In this example, we will create one mailbox and two tasks. The first task will put messages into the mailbox. The second will take them out and read them.

To get started, we first need to tell the compiler building the RTOS that we want to include mailbox functionality.

```c
var Mailbox = xdc.useModule('ti.sysbios.knl.Mailbox');
```
To do this, we add the line above to our config file. (In my case it was `empty.cfg`). This line lets the compiler know that we want to have the mailbox systems compiled in as part of the operating system.

Next, back to the routine we know better; let's add some headers. (... see that rhyme there?...)

```c
/* BIOS Header files */
#include <ti/sysbios/knl/Mailbox.h>

#define NUMMSGS         3       // Number of messages
```
As usual, we want to include the header file for the feature we are using.
Additionally, I've added `NUMMSGS` as a pre-processor definition and will use it to specific the max number of things that can be in the mailbox at one time.

Next we must define what a piece of mail looks like!
```c
typedef struct MsgObj {
    int        val;            // Message value
} MsgObj, *Msg;
```
For this example, let's create a struct named `MsgObj`. (Not familiar with structs? Take a look [here](https://www.tutorialspoint.com/cprogramming/c_structures.htm) to refresh your memory.)

the `MsgObj` struct is simple for this example, only containing a single `Char` named `val`.


Lets define our tasks.

```c
Void senderFxn(UArg arg0, UArg arg1)
{
    Mailbox_Handle mbxHandle = (Mailbox_Handle)arg0;
    int count = 0;
    while (1) {
        MsgObj msg;
        msg.val = count;
        Mailbox_post(mbxHandle, &msg, BIOS_WAIT_FOREVER);
        count++;
    }
}
```
First lets take a look at a simple sender. We first get our mailbox handle via `arg0` (as set by `main()`). Next we simply create a message, set its value to the count, and send the message!

But wait! What is the final argument `BIOS_WAIT_FOREVER`? This is the timeout value. If the mailbox is full, we must tell the kernel what we want to do about it. In this case, we want our task to wait until there is space in the mailbox before continuing.

```c
Void reciverFxn(UArg arg0, UArg arg1)
{
    Mailbox_Handle mbxHandle = (Mailbox_Handle)arg0;
    while (1) {
      MsgObj msg;
      Mailbox_pend(mbxHandle, &msg, BIOS_WAIT_FOREVER);
      if(msg.val % 10 == 0)
        //exciting operation here for when val %10 == 0
    }
}
```
Next lets define our simple receiver task. As we did before, lets get our mailbox handle from `arg0`.

Then for all time, we will simply use the function `Mailbox_pend` to wait for something to be in the mailbox and then receive it. For the example's sake, we do an operation on `val` and switch behavior accordingly.


```c
int main(void)
{
    //Call board init functions
    Board_initGeneral();

    Mailbox_Struct mbxStruct;

    //Construct a Mailbox Instance
    Mailbox_Params mbxParams;
    Mailbox_Params_init(&mbxParams);
    Mailbox_construct(&mbxStruct,sizeof(MsgObj), 2, &mbxParams, NULL);

    Mailbox_Handle mbxHandle;
    mbxHandle = Mailbox_handle(&mbxStruct);

    //...SNIP...
    Task_Params taskParams;
    Task_Params_init(&taskParams);
    taskParams.arg0 = (UArg)mbxHandle;
    //...SNIP...

}
```
Finally, lets look at the main function we use to construct the mailbox and a snippet of configuring the task to pass the mailbox as `arg0`.
