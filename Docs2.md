# RTX-421

These are the documents for the new version, the RTX-421. The previous documentation was in conflict, so I re-wrote them here.
These documents are the final cut, and are to be followed.

# General

The RTX-421 is a homebrew video card with hardware-accelerated sprites and drawing. It is primarily used to display text, but also hosts support for graphical modes.
It is also used as an extension of a 6502 or a 65816-based computer, used for fast graphics. The primary embeded-cpu, a W65C02S, can run at a stable 10mhz, or up to 20mhz.

**Note to self**
Make sure that signal timings are correct in the schematic berfore building a prototype

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
  	- A 20-bit address bus, as well as for sending commands
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

# Writing pixels
Writing a single pixel is probably the quickest instruction you can give it. In order to send a pixel to the display, set the lowest five pixels to the correct 5-bit color. The
upper three bits are not used. The lower-two are used for the red, the mid-two bits are for green, and the highest bit is for blue. The VGA pixel address is located in the 20-bit
address bus. *important:* the pixel address is *not* a standard pixel address, but instead a VGA pixel location. In order to take advantage of hardware-accelerated pass-writes,
you must use the VGA location, including front and back porches for both vertical and horizontal. In other operating modes this is not necessary. In order to send a command, make
sure that the BUSY pin is set low. If so, tick the SEND pin, and the GPU will process it. This is the same for any graphical mode, as well as single/double buffer modes.

Do keep in mind that all addresses above $0xB0000 are meant for commands, as they are beyond the VGA addresses for 800x600, and therefore unused.

# Pass-Write
A pass-write is the fastest way you can write data to the display. When you write to the GPU, and it is in a graphical mode, it will initialize one of these. If it
detects a command as the high-nibble of the address, it does that. Same with text mode. When the embeded CPU detects that you want a single pixel, it will write a "PASS" bit
in the internal register PA. When this happens, a PASS latch is set. This will disconnect the CPU from the address and data busses, allowing for the input latches to read. In the
same clock cycle, the QUEUE bit is enabled. This signals to the VGA writer that there is a waiting write request. Normally a CPU needs to do some processing on a request, but in
this case the data is already there. Instead of pulling from the CPU-latch registers, it pulls from the input for data and address. Data from the input latches is copied to four
registers in the same clock cycle, and on the falling edge of the CPU clock, the CPU is reconnected with the data busses. On the CPU-side, a pass-write is triggered by writing to
$0x2004. During the entire instruction the BUSY bit is set, and no more instructions are taken. It uses a read, compare and a write instruction.

# CPU-Write
Sometimes the CPU section must draw something not passed on directly by the driver. In that case the CPU must generate some valid pixel addresses and colors. Conserving CPU clock cycles
is of utmost importance when writing individual pixels. Using a more advanced W65C816S, it has full access to a 24-bit data bus, saving time. In order to write to a pixel location, you
must write to the standard *VGA address*, only the 24-th bit must be set. This signals to the hardware that you are writing or reading from the video hardware rather than the CPU memory.
For example, to write to pixel address $0x3D090, you instead write to $0x43D090. This has the final bit set high. When sending the data, on the rising edge of the clock for writing the data,
the address is latched into three 8-bit registers, for low, mid and high bits. The data bus is also latched. The QUEUE bit is also enabled, and it waits for the graphics' section to take the
request. At the end of any writes the GPU takes, the QUEUE bit is reset, usually in the same moment that the video memory is written. If not, the hardware would write the pixel to every
location.

# Display hardware
