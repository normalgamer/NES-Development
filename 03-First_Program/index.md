[Back to main page](https://normalgamer.github.io/NES-Development/)

# Let's start coding

## iNES header

The 16 byte iNES header gives the emulator all the information about the cart, including PRG/CHR bank sizes, mappers, mirroring... You can include all this inside your .asm's first lines:

```
  .inesprg 1  ; 1x 16Kb bank of PRG code
  .ineschr 1  ; 1x 8Kb bank of CHR data
  .inesmap 0  ; mapper 0 = NROM, no bank swapping
  .inesmir 1  ; Background mirroring (ignore for now)
```

For more info about NESASM3's iNES header, click [here](https://github.com/thentenaar/nesasm/blob/master/documentation/neshdr20.txt)

## Banking

As we discussed earlier, the cart stores info in multiple banks. NESASM3 organizes everything in 8Kb code and 8Kb graphics banks. To fill the 16Kb PRG space 2 banks are needed. For each bank you'll have to use the `.org` directive to tell the assembler where to locate the code.

```
  .bank 0
  .org $C000
  ; Code here
  
  .bank 1
  .org $E000
  ; More code here
  
  .bank 2
  .org $0000
  ; Graphics here
```

**Important note:** we have to start the PRG ROM at $C000 to later write the interrupt vectors, since those have to be placed at the end of the memory map. The PRG ROM has 16Kb of memory, so we place it at $C000 to end at $FFFF. If you don't do it, you'll have to place the interrupt vectors at the end of every bank.

## Including binary files

Additional data files are frequently used for graphics data or level data. The incbin directive can be used to include that data in your .nes file. This data isn't used right now, but it's needed to make the .nes file size match the iNES header.

```
  .bank 2
  .org $0000
  
  .incbin "mario.chr" ; Includes Super Mario Bros.' graphics
```

## Interrupt Vectors

There are three times when the NES processor will interrupt your code and jump to a new location. These vectors, held in the PRG ROM tell the processor where to go when that happens. Only the first two will be used in this tutorial.

### NMI Vector

This happens once per video frame, when enabled. The PPU tells the processor it is starting the VBlank time and is available for graphics updates.

### RESET Vector

This happens every time the NES starts up or is reset.

### IRQ Vector

This is triggered from some mapper chips or audio interrupts and will not be covered.

These three must appear in your assembly file in the right order in the last bytes of the memory map. The .dw directive is used to define a Data Word (1 word = 2 bytes, the two bytes make up an address).
```
  .bank 1
  .org $FFFA
  .dw NMI     ; When an NMI happens (once per frame if enabled) the CPU will jump to the label NMI
  .dw RESET   ; When the CPU turns on or is reset it will jump to the label RESET
  .dw 0       ; IRQ interrupt isn't used in this tutorial
```

## RESET code

The reset vector was set to the label RESET. Two things are turned off: the IRQ interrupts and decimal mode. More things have to be set to run on a real NES, but it will run on the FCEUXD emulator.

```
  .bank 0
  .org $C000
  
RESET:
  SEI ; Disable IRQ
  CLD ; Disable decimal mode
```

## Finishing up

We are going to turn the screen blue. To do this we are going to write some settings to the PPU. This is done to address $2001. The 76543210 is the bit number, from 7 to 0. Those bits form the byte you will write to $2001.

- 7: Intensify blues (and darken other colors)
- 6: Intensify greens (and darken other colors)
- 5: Intensify reds (and darken other colors)
- 4: Enable sprite rendering
- 3: Enable background rendering
- 2: Disable sprite clipping in leftmost 8 pixels of the screen
- 1: Disable background clipping in leftmost 8 pixels of the screen
- 0: Grayscale

So, to make the screen blue, we have to write the following:

```
  LDA #%10000000  ; Intensify blue
  STA $2001
  
Forever:
  JMP Forever ; Jump to forever, infinite loop
```

## Putting it all together

Download NESASM3, the source code and Super Mario Bros.' CHR file from [here](https://github.com/normalgamer/NES-Development/raw/gh-pages/03-First_Program/background.zip). Edit the build.bat file and change the .asm file's name to your file's name. Double click on it and you'll have a .nes file to run on an emulator.

Congratulations! You made your first NES program!
