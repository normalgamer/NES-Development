[Back to main page](https://normalgamer.github.io/NES-Development/)

# Displaying sprites and color palettes

## Palettes

Before displaying anything on screen, you need to set the color palette. There are two different palettes, each 16 bytes: one is used for the background, and the other for sprites. The byte in the palette corresponds to one of the 64 basic colors the NES can display ($0D shouldn't be used).

![NES palette](https://github.com/normalgamer/NES-Development/blob/gh-pages/04-Displaying_sprites/palette.png)

The palettes start at PPU address $3F00 and $3F10. To set this address, PPU address port $2006 is used. This port must be written twice, one for the high byte then for the low byte:

```
  LDA $2002 ; read PPU status to reset the high/low latch to high
  LDA #$3F
  STA $2006 ; Write the high byte of $3F10 address
  LDA #$10
  STA 2006  ; Write the low byte of $3F10 address
```

That code tells the PPU to set its address to $3F10. Now the PPU data port at $2007 is ready to accept data. The first write will go to the address you set ($3F10), then the PPU will automatically increment the address after each read or write. This sets the first 4 color in the palette:

```
  LDA #$32   ; Code for light blueish
  STA $2007  ; Write to PPU $3F10
  LDA #$14   ; Code for pinkish
  STA $2007  ; Write to PPU $3F11
  LDA #$2A   ; Code for greenish
  STA $2007  ; Write to PPU $3F12
  LDA #$16   ; Code for redish
  STA $2007  ; Write to PPU $3F13
```

You could continue to do writes to fill out the rest of the palette. OR you can use a much quicker method. First use the .db directive to store data bytes:

```
  .bank 1
  .org $E000  ; Let's append the palette data on the second half of the PRG bank
PaletteData:
  .db $0F,$31,$32,$33,$0F,$35,$36,$37,$0F,$39,$3A,$3B,$0F,$3D,$3E,$0F ; Background palette data
  .db $0F,$1C,$15,$14,$0F,$02,$38,$3C,$0F,$1C,$15,$14,$0F,$02,$38,$3C ; Sprite palette data
  ; Note: if you want to see how the colors look, open YY-CHR and tweak the colors
```

Then a loop is used to copy those bytes to the palette in the PPU. The X register is used as an index into the palette, and used to count how many times the loop has repeated. To copy both palettes (a total of 32 bytes) the loop starts at 0 and counts up to 32:

```
LoadPalettes:
	LDA $2002	  ; Read PPU status to reset the high/low latch
	LDA #$3F
	STA $2006	  ; Write the high byte of $3F00 address
	LDA #$00
	STA $2006	  ; Write the low byte of $3F00 address
	LDX #$00
LoadPalettesLoop:
	LDA PaletteData, x		; Load palette byte
	STA $2007             ; Write to PPU, it will be saved at address $3F00 + x (the one we set up before)
	INX                   ; Set index to next byte
	CPX #$20            
	BNE LoadPalettesLoop  ; If x = $20, 32 bytes copied, all done. If compare wasn't equal to 32, jump to LoadPalettesLoop

```

Once the loop finishes, the palette is fully copied.

## Sprites

Anything that moves separately from the background is made of sprites. A sprite is an 8x8 pixel tile that the PPU renders on screen. Generally objects are made from multiple sprited next to each other (for example Mario). The PPU has enough memory for 64 sprites. This memory is separate from all other video memory.

If you wanna inspect the mario.chr file, download [YY-CHR](https://www.smwcentral.net/?p=section&a=details&id=22338) and open your mario.chr file.

### Sprite DMA

In order to copy sprites to the sprite memory, Direct Memory Access (DMA) is used. This allows to copy a block of RAM from CPU memory to the PPU sprite memory. The onboard RAM space from $0200 to $02FF is usually used. To start the transfer, two bytes need to be written to the PPU ports:

```
NMI:  ; Remember the NMI label? We add the following code to start sprite transfer every frame

  LDA #$00
  STA $2003 ; Set the low byte (00) of the RAM address
  LDA #$02
  STA $4014 ; Set the high byte (02) of the RAM address, start the transfer
            ; Now all sprite data located from $0200 address onwards will be copied to the PPU sprite memory
```

Later on the lesson we will copy sprite data to the RAM addresses $0200-$02FF.

### Sprite Data

Each sprite needs 4 bytes of data for its position and tile information in this order:

- 1: Y Position - vertical position of the sprite. $00 is the top of the screen. Anything above $EF is off the bottom of the screen.

- 2: Tile Number - this is the tile number ($00 to $FF) for the graphic to be taken from a Pattern Table. In YY-CHR, hover over the sprite and look at the bottom left corner to get the tile number.

- 3: Attributes - this holds color and displaying information:
  ```
  76543210
  |||   ||
  |||   ++- Color Palette of sprite.  Choose which set of 4 from the 16 colors to use
  |||
  ||+------ Priority (0: in front of background; 1: behind background)
  |+------- Flip sprite horizontally
  +-------- Flip sprite vertically
  ```

- 4: X Position - horizontal position of the screen. $00 is the left side, anything avobe $F9 is off screen.

Those 4 bytes repeat 64 times (one set per sprite) to fill the 256 bytes of sprite memory. If you want to edit sprite 0, you can change $0200-0203. Sprite 1 is $0204-0207, etc.

### Turning NMI/Sprites On

The PPU port $2001 is used again to enable sprites (the same one we used to intensify the blues). Setting bit 4 to 1 will make them appear. NMI also needs to be turned on, so the sprite DMA will run and the sprites will be copied every frame. This is done with the PPU port $2000. The pattern table 0 is also selected to choose sprites from. Background will come from pattern table 1 when that is added later.

PPUCTRL ($2000)
- 7: Generate an NMI at the start of the vertical blanking interval VBlank (0: off, 1: on)
- 6: PPU master/slave select
- 5: Sprite size (0: 8x8, 1: 8x16)
- 4: Background pattern table address (0: $0000, 1: $1000)
- 3: Sprite pattern table address for 8x8 sprites (0: $0000, 1: $1000)
- 2: VRAM address increment per CPU read/write of PPUDATA (0: increment by 1, going across, 1: increment by 32, going down)
- 1 & 0: Base nametable address (0 = $2000; 1 = $2400; 2 = $2800; 3 = $2C00)

CPU RAM address $0200-$02FF is usually used to store the sprite data, which is then copied to the PPU. And the new code to set up the sprite data:

```
  ; Sprite data stored at RAM addresses $0200-$0203. Once NMI is called it will copy data from here
  LDA #$80
  STA $0200       ; Put sprite 0 in center ($80) of screen vertically
  STA $0203       ; Put sprite 0 in center ($80) of screen horizontally
  LDA #$00
  STA $0201       ; Tile number = 0
  STA $0202       ; Color palette = 0, no flipping
  
  LDA #%10000000  ; Enable NMI, sprites from Pattern Table 0
  STA $2000
  
  LDA #%00010000  ; No intensify (black background), enable sprites
  STA $2001

Forever:
  JMP Forever ; Infinite loop
```

## At last

Expanded the reset code. You don't need to know what it exactly does, it just resets the NES, clears the memory, and waits for the first VBlank.

## Putting it all together

Download NESASM3 and Super Mario Bros.' CHR file from [here](). Edit the build.bat file and change the .asm file's name to your file's name. Double click on it and you'll have a .nes file to run on an emulator.

Congratulations! You can now see Mario's head's back on screen!
