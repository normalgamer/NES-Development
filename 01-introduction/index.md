# Introduction to the NES

The NES is an 8-bit console released in 1985.

## Architecture Overview
The NES includes a custom 6502 based CPU, the 65C02, with and Audio Processing Unit (APU) and a controller handling inside one chip, and a Picture Processing Unit (PPU).
There are 2 kilobytes of RAM connected to the CPU to store variables, and 2 kilobytes of RAM connected to the PPU.

Some carts included extra CPU RAM, called Work RAM or WRAM. If a cart has to store save data in it, it will have a battery attached to make sure it isn't erased. Other 
carts may include extra PPU RAM, sound channels and others. Each cart has at least 3 chips: one for the program code (PRG), one for the character graphics (CHR), and the 
last is the lockout (not involved in programming the game).

## CPU Overview

The NES CPU is a 65c02, a modified version of the 8 bit 6502 used in other systems like the Commodore 64. It has a 16 bit address bus to access up to 64Kb of memory. 
Included in that memory space is the 2Kb of CPU RAM, ports to access the PPU/APU/controller ports, WRAM (if on the cart), and 32Kb for PRG ROM. 32Kb quickly became too 
small for bigger games, which is why memory mappers were used. These allowed for bank swapping, to use multiple PRG and CHR banks

### Memory map

$0000   | 2Kb CPU RAM

$0800   | Unused

$2000   | PPU IO Ports

$4000   | APU/Controller IO Ports

$6000   | 8Kb Cartridge RAM (WRAM)

$8000   | 32Kb Cartridge ROM

$FFFA   | NMI/RESET/IRQ vectors

## PPU Overview
