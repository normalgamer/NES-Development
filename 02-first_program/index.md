# First program

Now we're getting serious. We're going to start programming in assembly, and we will make the screen go blue.

## Directives

Directives are commands you send to the assembler to do things like locating code in memory. They start with a . and are indented (some people use 2 spaces, and other people use 4). This directive tells the assembler to put the code at address $8000, which is inside the game ROM area:

`  .org $8000`

## Labels

Labels are aligned to the left and have a : after their name. The label is something you use to organize your code. The assembler will translate the label to an address.

```
  .org $8000
Function:
```

When the assembler runs, it translates the label with an address. So for the `Function` label, the assembler will translate it to `$8000`. For example, `JMP Function` will translate to `JMP $8000`

## Opcodes
