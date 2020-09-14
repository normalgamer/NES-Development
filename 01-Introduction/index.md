[Back to main page](https://normalgamer.github.io/NES-Development/)

# Introduction to the NES

## Basic Terminology

- Kb: Memory size is listed in kilobytes
- ROM: Read Only Memory, holds unchangeable data. This is where the game's code and graphics are stored
- RAM: Random Access Memory, holds data that can be changed. When power is removed, the data is erased
- WRAM: Working RAM, this RAM is backed by a battery. It is used in carts to store save data
- PRG: PRoGram memory, this is the game's code
- CHR: CHaRacter memory, the graphics data
- CPU: Central Processing Unit, the processor chip
- PPU: Picture Processing Unit, the graphics chip
- APU: Audio Processing Unit, the sound chip


## Architecture Overview
The NES includes a custom 6502 based CPU, the 65C02, with and Audio Processing Unit (APU) and a controller handling inside one chip, and a Picture Processing Unit (PPU). There are 2 kilobytes of RAM connected to the CPU to store variables, and 2 kilobytes of RAM connected to the PPU.

Some carts included extra CPU RAM, called Work RAM or WRAM. If a cart has to store save data in it, it will have a battery attached to make sure it isn't erased. Other carts may include extra PPU RAM, sound channels and others. Each cart has at least 3 chips: one for the program code (PRG), one for the character graphics (CHR), and the last is the lockout (not involved in programming the game).

## CPU Overview

The NES CPU is a 65c02, a modified version of the 8 bit 6502 used in other systems like the Commodore 64. It has a 16 bit address bus to access up to 64Kb of memory. Included in that memory space is the 2Kb of CPU RAM, ports to access the PPU/APU/controller ports, WRAM (if on the cart), and 32Kb for PRG ROM. 32Kb quickly became too small for bigger games, which is why memory mappers were used. These allowed for bank swapping, to use multiple PRG and CHR banks

### Memory map

Address range | Device
--------------|-----------------------
$0000-$07FF   | 2Kb internal RAM
$0800-$0FFF   | Mirror of $0000-$7FFF
$1000-$17FF   | Mirror of $0000-$7FFF
$1800-$1FFF   | Mirror of $0000-$7FFF
$2000-$2007   | [NES PPU](http://wiki.nesdev.com/w/index.php/PPU_registers) registers
$2008-$3FFF   | Mirrors of $2000-$2007 (repeats every 8 bytes)
$4000-$4017   | [NES APU](http://wiki.nesdev.com/w/index.php/APU) and [I/O registers](http://wiki.nesdev.com/w/index.php/2A03)
$4018-$401F   | APU and I/O functionality that is normally disabled (see [CPU Test Mode](http://wiki.nesdev.com/w/index.php/CPU_Test_Mode))
$4020-$FFFF   | Cartridge space: PRG ROM, PRG RAM, and [mapper](http://wiki.nesdev.com/w/index.php/Mapper) registers

Note: Most common boards and iNES mappers address ROM and Save/Work RAM in this format:

- $6000-$7FFF: Battery backed Save/Work RAM (WRAM)
- $8000-$FFFF: Usual ROM, commonly with mapper registers

The CPU expects interrupt vectors in a fixed place at the end of the cartridge space:

- $FFFA-$FFFB: NMI vector
- $FFFC-$FFFD: RESET vector
- $FFFE-$FFFF: IRQ/BRK vector

## PPU Overview

The NES PPU is a custom chip that does all the graphics display. It includes internal RAM for sprites and the color palettes. There is RAM on the NES board that holds background, and the graphics are fetched from the cart CHR memory. The PPU processes one TV scanline at a time. If there are more than 8 sprites on the scanline the rest are ignored. This is why sometimes sprites flicker on screen when too much is happening at once. When all the scanlines are done there is a period when no graphics are sent out. This is called VBlank and is the only time graphics can be updated. Both the NTSC and PAL systems have a resolution of 256x240 pixels, but the top and bottom 8 rows are typically cut off by the NTSC TV resulting in 256x224, and some TV borders may cut from 0 to 8 additional rows. NTSC runs at 60Hz, while PAL runs at 50Hz.

You can't access the PPU's memory like you would normally do. In lesson 4, you'll learn how to use the PPU I/O ports to communicate with the PPU's internal RAM.

### Memory map

$0000   | Cartridge RAM/ROM, Pattern table 0, 256 tiles

$1000   | Cartridge RAM/ROM, Pattern table 1, 256 tiles

$2000   | Name table 0, 32x30 tiles

$23C0   | Attribute table 0

$2400   | Name table 0, 32x30 tiles

$27C0   | Attribute table 1

$2800   | Name table 2, 32x30 tiles

$2BC0   | Attribute table 2

$2C00   | Name table 3, 32x30 tiles

$2FC0   | Attribute table 3

$3000   | Unused

$3F00   | Background palette

$3F10   | Sprite palette

$3F20   | Unused

$3FFF   | End

Address range | Device
--------------|-------
$0000-$0FFF   | Cartridge RAM/ROM, Pattern table 0 (first 256 8x8 tiles)
$1000-$1FFF   | Cartridge RAM/ROM, Pattern table 1 (first 256 8x8 tiles)
$2000-$23FF   | Nametable 0
$2400-$27FF   | Nametable 1
$2800-$2BFF   | Nametable 2
$2C00-$2FFF   | Nametable 3
$3000-$3EFF   | Mirrors of $2000-$2EFF
$3F00-$3F1F   | Palette RAM Indexes
$3F20-$3FFF   | Mirrors of $3F00-$3F1F

PPU Registers | Name      | Description  
--------------|-----------|------------
$2000         | PPUCTRL   | Enable NMI, sprite/background table address
$2001         | PPUMASK   | Controls the rendering of sprites and background
$2002         | PPUSTATUS | This register reflects the state of various functions inside the PPU (VBlank, sprite overflow...)
$2003         | OAMADDR   | Write the address of [OAM](http://wiki.nesdev.com/w/index.php/PPU_OAM) you want to access, to manipulate sprite position and color
$2004         | OAMDATA   | Write OAM data here. 
$2005         | PPUSCROLL | Changes the scroll position, used to scroll the screen
$2006         | PPUADDR   | The CPU can't access directly the PPU memory, so it uses two registers to write to Video RAM. First loads an address to PPUADDR...
$2007         | PPUDATA   | ...and then reads/writes data.
$4014         | OAMDMA    | Writing $XX will upload 256 bytes of data from CPU page $XX00-$XXFF to the internal PPU OAM. This page is usually located in internal RAM $0200-$02FF, but cartridge ROM/RAM can be used too.

For more info, visit http://wiki.nesdev.com/w/index.php/PPU_memory_map.

## Graphics System Overview

### Tiles

Tiles are made of 8x8 pixels. Larger characters like Mario are made from multiple tiles. Backgrounds are also made of tiles.

### Sprites

The PPU has enough memory for 64 sprites or things that move around on screen. Only 8 sprites per scanline are allowed, that's why you may see sprite flickering when too much is happening at once.

### Background

This is the landscape graphics, which scrolls at once. The sprites can be either be displayed in front or behind the background. The screen is big enough for 32x30 background tiles, and there is enough internal RAM to hold 2 screens. When games scroll the background is updated off screen before scrolling on screen.

### Pattern Tables

This is where the tile data is actually stored. It can be either ROM or RAM. Each pattern table holds 256 tiles. One table is used for sprites, and the other one for backgrounds.

### Attribute Tables

These tables set the color information in 2x2 tile sections. This means that a 16x16 pixel area can only have 4 different colors from the palette.

### Palettes

These two areas hold the color information, one for the background and one for sprites. Each palette has 16 colors. To display a tile on screen, the pixel color index is taken from the Pattern Table and the Attribute Table. That index is then looked up in the Palette to get the actual color.
