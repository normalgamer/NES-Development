[Back to main page](https://normalgamer.github.io/NES-Development/)

# Controller Input

## Controller Ports
The controller input is read from I/O ports $4016 and $4017, for controller 1 and 2 respectively. First you have to write $01 then $00 to $4016. This tells the controllers to latch the current button positions. Then you read from $4016/$4017 to get input. Buttons are sent one at a time, in bit 0. If bit 0 is zero, the button isn't pressed. If it's 1, it's pressed.

Button status for each controller is returned in this order: A, B, Select, Start, Up, Down, Left, Right.

```
  LDA #$01
  STA $4016
  LDA #$00
  STA $4016
  
  LDA $4016 ; Player 1 - A
  LDA $4016 ; Player 1 - B
  LDA $4016 ; Player 1 - Select
  LDA $4016 ; Player 1 - Start
  LDA $4016 ; Player 1 - Up
  LDA $4016 ; Player 1 - Down
  LDA $4016 ; Player 1 - Left
  LDA $4016 ; Player 1 - Right
  
  LDA $4017 ; Player 2 - A
  LDA $4017 ; Player 2 - B
  LDA $4017 ; Player 2 - Select
  LDA $4017 ; Player 2 - Start
  LDA $4017 ; Player 2 - Up
  LDA $4017 ; Player 2 - Down
  LDA $4017 ; Player 2 - Left
  LDA $4017 ; Player 2 - Right
```

## AND Instruction

Button info is sent in bit 0, so we need to eliminate the rest of the bits. This can be done with the AND instruction. Each of the bits is ANDed with the bits from another value. If the two bits on the same position are 1, the result is one, else it's zero.

```
    01101110
AND 11101001
------------
    01101000
```

We only want bit 0, so to remove the rest:

```
    01011101  ; Controller data
AND 00000001  ; AND value
------------
    00000001  ; Result
```

So to erase all the other bits, the AND instruction should be used after each read to $4016/$4017.

## BEQ Instruction

In the past we've use BNE to jump somewhere when checking conditions if they weren't equal. Branch if EQual will be used without a compare instruction to jump somewhere when equal to zero. When a button is not pressed, the value will be zero, so the branch will be taken (Note: when comparing values, CMP returns 0 if it's equal). That will skip all the instructions that do something if the button's pressed:

```
ReadA:
  LDA $4016       ; Read input for button A
  AND #%00000001  ; Remove all bits but bit 0
  BEQ ReadADone   ; If A isn't pressed (0), skip next instructions
  
  ; Some code here to do something when button is pressed
  
ReadADone:
```

## INC and DEC Instructions

For this example we're going to move Mario left and right. To do that we'll have to increase and decrease Mario's X and Y coordinates. The INC and DEC instructions INCrement and DECrement values. To move Mario right, we'll have to increment its X position. If you remember, we copied Mario's sprite data to RAM addresses $0200-$02FF. So let's modify it:

```
  ; Sprite data:
  vert, tile, attr, horiz
  #$XX, #$XX, #$XX, #$XX
  
  INC $0203 ; If Mario's first sprite's data starts at $0200, $0203 is the horizontal position
  INC $0207 ; Add 4 for the next sprite's x position
  INC $020B
  INC $020F
```

## Putting it all together

Download NESASM3, the source code and Super Mario Bros.' CHR file from [here](https://github.com/normalgamer/NES-Development/raw/gh-pages/06-Controller_input/controller.zip). Double-click the build.bat script and you'll have a .nes file to open on an emulator. Open FCEUXD and click Config>Input to change your input.

Congratulations! You can now move Mario left and right!

*Psst, if you want to see the RAM addresses altered, click on Debug>RAM Search, scroll down to address $0203/$0207/$020B/$020F and select Hexadecimal on Data Type/Display. Now you can see how the values go up and down :D*
