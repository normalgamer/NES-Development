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
