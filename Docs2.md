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
  	- A write pin, to send a command
  	- Hardware-accelerated pixel writes
  	- Queue up multiple commands at once

 # Technical

 The most important pin on the board is called "BUSY". This pin is designed to tell the driver that the GPU is busy with a command. It is *very* important to wait
 for this to end. During a pixel write, the CPU may use the latch registers more than once. During this, the CPU is shut off from the data pins. If you never notice,
 the GPU could drop entire commands that way be vital to your system. In order to use multiple commands at once, queue them up using a special command. This is described
 later. Keep in mind queud commands do not use a passthrough, so they may take longer.


 When writing sprites or rendering a large scene in pixels, it is wise to enable overclocking. This command is also described. Overclocking allows the CPU to run at 20mhz
 or higher, allowing for faster processing. If you want to display a flat image, or a text terminal, then overclocking may not be necessary. The GPU does not need to be refreashed
 once per frame, as the display buffer is one single 1Mbit SRAM chip.
