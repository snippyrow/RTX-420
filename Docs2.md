# RTX-421

These are the documents for the new version, the RTX-421. The previous documentation was in conflict, so I re-wrote them here.
These documents are the final cut, and are to be followed.

# General

The RTX-421 is a homebrew video card with hardware-accelerated sprites and drawing. It is primarily used to display text, but also hosts support for graphical modes.
It is also used as an extension of a 6502 or a 65816-based computer, used for fast graphics. The primary embeded-cpu, a W65C02S, can run at a stable 10mhz, or up to 20mhz.

Here is the full features' list:
- Rendering text to the screen:
	- Full ASCII character set, 9x16
   	- Auto scrolling to fit screen size
   	- Changing the cursor position
   	- Clearing the text UI
   	- Choose one of 32 text colors
- A range of graphical settings:
  	- Support for 320x200 @ 70hz, 640x480 @ 60hz, and 800x600 @ 60hz
  	- A color pallette of 32 (5-bit color)
- As well as software:
  	- Draw circles, rectangles, squares, lines, triangles, and more!
  	- Enable double-buffering mode, handled on hardware
  	- An 8-bit data bus
  	- A 19-bit address bus, as well as for sending commands
  	- A pin to tell the driver about whether the GPU is busy
 
  
