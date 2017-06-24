```
#define NUMMSGS         3       /* Number of messages */

typedef struct MsgObj {
    Int         id;             /* Writer task id */
    Char        val;            /* Message value */
} MsgObj, *Msg;
```

```
int main(void)
{
    /* Call board init functions */
    Board_initGeneral();

    Mailbox_Struct mbxStruct;

    /* Construct a Mailbox Instance */
    Mailbox_Params mbxParams;
    Mailbox_Params_init(&mbxParams);
    Mailbox_construct(&mbxStruct,sizeof(MsgObj), 2, &mbxParams, NULL);

    Mailbox_Handle mbxHandle;
    mbxHandle = Mailbox_handle(&mbxStruct);
}
```
