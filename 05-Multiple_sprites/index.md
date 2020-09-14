[Back to main page](https://normalgamer.github.io/NES-Development/)

# Multiple sprites

Last time we loaded only one sprite, but multiple sprites is the real deal. But using multiple LDAs/STAs takes a lot of processing time. To optimize this we will store sprite data the same way we did with palettes, and we will create a new loop to load that data.

First let's set up the sprite data using .db:

```
Sprites:
    ; vert, tile, attr, horiz
  .db $80, $32, $00, $80  ; Sprite 0
  .db $80, $33, $00, $88  ; Sprite 0
  .db $88, $34, $00, $80  ; Sprite 0
  .db $88, $35, $00, $88  ; Sprite 0
  
  ; The tile numbers correspond to mario.chr file's smol Mario. If you want to dislpay other sprites, open mario.chr with YY-CHR and hover the mouse over the tiles.
  ; The tile number appears on the bottom
```

This is only the starting data, we need to copy it into RAM. Once it's on the RAM, you can easily alter the position, tile number, etc. We still need a loop to copy it:

```
LoadSprites:
  LDX #$00    ; Reset X
LoadSpritesLoop:
  LDA Sprites, x      ; Load data from address Sprites + x
  STA $0200, x        ; Store data in address $0200 + x
  INX                 ; Increment X
  CPX #$10            ; Compare X to hex $10, decimal 16
  BNE LoadSpritesLoop ; If compare wasn't equal jump to LoadSpritesLoop
```

If you want to add more sprites, add more lines using .db in the Sprites section and increase the CPX value. Now you can edit the sprite data by editing the RAM addresses $0200-$02FF.

And that's it!

## Putting it all together
Download NESASM3, the source code and Super Mario Bros.' CHR file from [here](). Edit the build.bat file and change the .asm file's name to your file's name. Double click on it and you'll have a .nes file to run on an emulator.

And that's it! It was short, but in the next lesson we will read input from the controller to move Mario around the screen.
