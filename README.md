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
  - 8-bit data bus for sending bytes to the card
  - 19-bit address bus for addressing pixels

# Technical docs
This section is a reference for my own design
Maybe do some of this in software.. if it does not have performance cuts

# General:
  Before the buffering mode or display mode can be set, a frame must first be completed. This is signaled by the 'RE' label transitioning from low to high.
  This is caused by the throughput register ending. When this happens, all the status registers are reset, and any flags/modes are updated.

  Addresses $0x2000-$0x4000 are reserved and should not be edited without meaningful pixel writes.

  Text mode is really an expansion of the graphics modes, just with hardware accelerated font rendering and management. It is always operated at 640x480 @ 60hz,
  in 70x25 for characters. Each character is 9x16 pixels wide and tall. Text mode can also work with the double bit, allowing more compatibility. The resolution
  cannot be changed.

  The status register is conposed of the following:
  0bxxxCMMBF
  x -> don't care
  C -> Cooling mode. Turn on fan (not implemented)
  MM -> operating mode. Refer to the featureset! (W)
  B -> Busy flag. The busy flag is set when the CPU section is currently performing an action, such as drawing a sprite to the screen. (R)
  F -> Frame set. Set to a 0 for a single buffer and 1 for double mode. (W)

  Writing pixels to the GPU is easy, through the 19-bit address bus and the 8-bit data bus. In text mode, the address bus is not normally used. Text is simply
  appended to the end of the text buffer, and changing the cursor position is relativly straightforward using bytes higher than 128.
  In graphical mode, pixels are written by simply inserting the required address into the address bus and writing the pixel color into the data bus (32-palette)

  For drawing a sprite, such as a square, rectangle or other shape, we need to send a command. In order to send a command, you must write to addresses $0xC0000 and above.
  Certain commands will be palced at different locations along the address space, and they will be described later.
  In order to set which buffer is selected, you must also send a command. Usually a good way to toggle which buffer is selected is by sending 0x0/0x1 to vector $0xC0000

  If you attempt to send more commands at once than the GPU can handle by not using the BUSY flag, it will slow down. Instructions will be queued as if a stack, but stacking
  instructions takes resources!

# For single-buffer graphics' modes:
  In order for the CPU section to write a pixel to video memory, it does as follows:
  First, whether it is in double mode or not, it latches the address and data busses. This is done by writing the data to $0x2000.
  The next memory write cycle will set the pixel. Write any value to $0x2000, but the lowest three bits will become the upper three bits of the 19-bit pixel address.
  When the CPU sends the address, and a clock is ticked, a label called "latch 0" is triggered. This latches everything, as well as updating the status register (1 bit).
  Next, the CPU somehow waits for the next write instruction, skipping over the fetch. This can be done by counting the clock pulses using a shift register.
  When the next write instruction moves to write, the address line is already on the data bus. Therefore we set a new label, "latch 1".
  This data is then latched in an extra register, to complete the address. The extra register is demultiplexed with the bare CPU address line, for double mode.

  The reason we can use the bare address bus for double is that we do not need to share with the VGA output, and instead can manually write to it at any time. 
  After everything has been latched successfully, we trigger a special register called the queue bit. This tells the VGA controller that it has a write request.
  After that we can go ahead and write the next bit, as the CPU section will complete write requests far slower than the VGA section can write pixels.

  When the VGA starts a pixel and sees that the queue bit is high, it starts the process of writing the data to the video memory. Regardless of a queue bit, operating mode, or whatnot,
  it will always send the currently selected data to the throughput register. This contains all the information about the pixel color, sync pulses, or to signal a reset of the frame and GPU.
  First, it updates the throughput. Things get started when the currently selected VGA signal oscillator transitions to a low level.
  This is so that no glitches happen. Next, a flipflop is activated. This will give the latched address control over the memory as well as passing the latched
  data bus to the I/O ports. Immediatly after that, another register is activated after a few nanoseconds. This will trigger the memory to be written, as well as resetting both registers.
  As this happens, the counters regain control over the address lines, and keep the memory trained there for now.

  The queue bit is also reset after the subsequent write, and the state register is fully reset immediatly after. After all this, the CPU may send another write request to the VGA section,
  or the VGA section may initiate more pixel displays. If no queue bit is set at the start of each pixel, then no action is taken and it is trained on that address.

# For double-buffer graphics' modes:
  For sending a write request, the process is a little more simple than using a single buffer. Data is latched the same as using a single buffer, except for the upper three bits of the address.
  When sending the write request, all we need to do is write anything to the address $0x2000. The low three bits become the upper three of the address, and the data is immediatly written into
  the video memory like using a normal memory chip. The pixels we are writing go into the back buffer, while the front buffer is currently being displayed. We do not need to touch the front buffer
  as it is being displayed. We can do all the work we want on the bottom without anything showing up. Finally, when we are done with the frame, we can send the command to switch the buffer,
  and the other buffer comes avalible. The VGA section automatically does this for us, so we can simply resume as normal.

# For text modes:
  Text mode functions the same as graphical 640x480 @ 60hz, but dedicated to printing text rather than pixels. The process is the same as 1/2 buffer modes, however.

