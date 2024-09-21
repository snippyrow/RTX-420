# RTX-420
This is the technical documentation for my project graphics card, the RTX 420.
The card can run at any speed the user wants, but certain untested limitations may come of it.
*This card is designed to render text to a screen, with extra features*

The card has the following features:
  - Render text to the screen
  - Autoscroll, change cursor position, clear screen, change text color, etc.
  - A color pallette of 32
  - Support for 320x200 @ 70hz, 640x480 @ 60hz, and 800x600 @ 60hz
  - Support for loading sprites to the screen, such as circles, rectangles, lines and triangles.
  - A register to show the user it is busy, AKA a flags register
  - A pin to enable or disable double buffering in hardware
  - Two pins to switch video operation modes (0 -> Text, 1 -> 320x200 , 2 -> 640x480, 3 -> 800x600

# Technical docs
This section is a reference for my own design

