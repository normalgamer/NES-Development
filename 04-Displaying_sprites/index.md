[Back to main page](https://normalgamer.github.io/NES-Development/)

# Displaying sprites and color palettes

## Palettes

Before displaying anything on screen, you need to set the color palette. There are two different palettes, each 16 bytes: one is used for the background, and the other for sprites. The byte in the palette corresponds to one of the 64 basic colors the NES can display ($0D shouldn't be used).
