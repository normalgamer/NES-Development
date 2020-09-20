[Back to main page](https://normalgamer.github.io/NES-Development/)
# First program

Now we're getting serious. We're going to start programming in assembly, and we will make the screen go blue.

## Directives

Directives are commands you send to the assembler to do things like locating code in memory. They start with a . and are indented (some people use 2 spaces, and other people use 4). This directive tells the assembler to put the code at address $8000, which is inside the game ROM area:

```
  .org $8000
```

## Labels

Labels are aligned to the left and have a : after their name. The label is something you use to organize your code. The assembler will translate the label to an address.

```
  .org $8000
Function:
```

When the assembler runs, it translates the label with an address. So for the `Function` label, the assembler will translate it to `$8000`. For example, `JMP Function` will translate to `JMP $8000`

## Opcodes

The opcode is the instruction that the processor will run, and it's indented like directives. For example:
```
  .org $8000
Function:
  JMP Function
```

## Operands

Operands are additional information for opcodes. An opcode can have between 1 and 3 operands. In this example `#$00` is the operand

```
  .org $8000
Function:
  LDA #$00
  JMP Function
```

## Comments

Comments are to help you understand in English what the code is doing. The assembler ignores these comments. They start with a ;. For example:

```
  .org $8000
Function:
  LDA #$00      ; Load #$00 in the accumulator
  JMP Function
```

## Loading values

`LDA #$00` loads 00 into the accumulator

`LDA $0005` loads the value stored in address $0005 into the accumulator

The $ tells the CPU to load the value stored in the address.

The # loads an direct value:

- The # loads a decimal value.
  
- The #$ loads a hex value.
  
- The #% tells the CPU to load a binary value.

## Registers

A register is a small variable a CPU has to store values. The 65c02 has three 8 bit general purpose registers: the accumulator (A), X and Y. There are additional registers, but those won't be used now.

### Accumulator

The Accumulator (A) is the main 8 bit register for loading, storing, comparing and doing math on data. The most frequent operators are:

```
  LDA #$A0    ; Load value A0 into the accumulator
  STA #$0010  ; Store the value in the accumulator into address 0010
```

### X and Y

The X and Y registers are Index Registers, usually used for counting or memory access. They are also used in loops to keep track of how many times the loop has run, while using A to process data.

```
  LDX #$FF
  STX $0010
  
  LDY #$10
  STY $0011
```

## Status Register

The Status Register holds flags with information about the last instruction. For example when doing a subtract you can check if the result was a zero.

## Common Load/Store Opcodes

```
  LDA #$0A   ; LoaD the value 0A into the accumulator A
             ; the number part of the opcode can be a value or an address
             ; if the value is zero, the zero flag will be set.

  LDX $0000  ; LoaD the value at address $0000 into the index register X
             ; if the value is zero, the zero flag will be set.

  LDY #$FF   ; LoaD the value $FF into the index register Y
             ; if the value is zero, the zero flag will be set.

  STA $2000  ; STore the value from accumulator A into the address $2000
             ; the number part must be an address

  STX $4016  ; STore value in X into $4016
             ; the number part must be an address

  STY $0101  ; STore Y into $0101
             ; the number part must be an address

  TAX        ; Transfer the value from A into X
             ; if the value is zero, the zero flag will be set

  TAY        ; Transfer A into Y
             ; if the value is zero, the zero flag will be set

  TXA        ; Transfer X into A
             ; if the value is zero, the zero flag will be set

  TYA        ; Transfer Y into A
             ; if the value is zero, the zero flag will be set
```

## Common Math Opcodes

```
  ADC #$01   ; ADd with Carry
             ; A = A + $01 + carry
             ; if the result is zero, the zero flag will be set

  SBC #$80   ; SuBtract with Carry
             ; A = A - $80 - (1 - carry)
             ; if the result is zero, the zero flag will be set

  CLC        ; CLear Carry flag in status register
             ; usually this should be done before ADC

  SEC        ; SEt Carry flag in status register
             ; usually this should be done before SBC

  INC $0100  ; INCrement value at address $0100
             ; if the result is zero, the zero flag will be set

  DEC $0001  ; DECrement $0001
             ; if the result is zero, the zero flag will be set

  INY        ; INcrement Y register
             ; if the result is zero, the zero flag will be set

  INX        ; INcrement X register
             ; if the result is zero, the zero flag will be set

  DEY        ; DEcrement Y
             ; if the result is zero, the zero flag will be set

  DEX        ; DEcrement X
             ; if the result is zero, the zero flag will be set

  ASL A      ; Arithmetic Shift Left
             ; shift all bits one position to the left
             ; this is a multiply by 2
             ; if the result is zero, the zero flag will be set

  LSR $6000  ; Logical Shift Right
             ; shift all bits one position to the right
             ; this is a divide by 2
             ; if the result is zero, the zero flag will be set
```

## Common Comparison Opcodes

```
  CMP #$01   ; CoMPare A to the value $01
             ; this actually does a subtract, but does not keep the result
             ; instead you check the status register to check for equal,
             ; less than, or greater than

  CPX $0050  ; ComPare X to the value at address $0050

  CPY #$FF   ; ComPare Y to the value $FF
```

## Common Flow Control Opcodes

```
  JMP $8000  ; JuMP to $8000, continue running code there

  BEQ $FF00  ; Branch if EQual, contnue running code there
             ; first you would do a CMP, which clears or sets the zero flag
             ; then the BEQ will check the zero flag
             ; if zero is set (values were equal) the code jumps to $FF00 and runs there
             ; if zero is clear (values not equal) there is no jump, runs next instruction

  BNE $FF00  ; Branch if Not Equal - opposite above, jump is made when zero flag is clear
```
