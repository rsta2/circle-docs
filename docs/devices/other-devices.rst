Other devices
~~~~~~~~~~~~~

This section covers some device driver classes, which do not belong to other groups of devices. These classes have their own interface and are not derived from the class :cpp:class:`CDevice`.

CBcmFrameBuffer
^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/bcmframebuffer.h>

.. cpp:class:: CBcmFrameBuffer

	This class is a driver for the frame buffer device(s), provided by the firmware of the Raspberry Pi. The Raspberry Pi 4, 400 and the Compute Module 4 support multiple frame buffer devices, all other models only one. A frame buffer is basically an address range in main memory, which is continuously read by the firmware in background, to be displayed on a HDMI or composite TV display. Writing to this memory address range modifies the displayed image. The Raspberry Pi firmware supports frame buffers with different widths, heights and depths of the pixel information. If one wants to display text in a frame buffer, the characters must be formed from a character generator in the software. The firmware does not support text displays on its own.

.. note::

	To be able to use more than one frame buffer device, the option ``max_framebuffers=N`` (N > 1) is required in the file *config.txt* on the SD card.

.. cpp:function:: CBcmFrameBuffer::CBcmFrameBuffer (unsigned nWidth, unsigned nHeight, unsigned nDepth, unsigned nVirtualWidth = 0, unsigned nVirtualHeight = 0, unsigned nDisplay = 0, boolean bDoubleBuffered = FALSE)

	Constructs a frame buffer device object with ``nWidth`` * ``nHeight`` pixels. If both parameters are zero, the frame buffer is automatically created with the default size, which is normally the maximum supported size of the connected display. Each pixel has a depth of ``nDepth`` bits (4, 8, 16, 24 or 32).

	The memory range of the frame buffer may be larger than the displayed physical display size. This can be used to quickly switch the displayed image (see :cpp:func:`SetVirtualOffset()`). The optional virtual display size is ``nVirtualWidth`` * ``nVirtualHeight`` pixels. If ``bDoubleBuffered`` is ``TRUE``, the virtual display height is automatically set to twice the physical display size, if ``nVirtualWidth`` and ``nVirtualHeight`` are specified as 0.

	``nDisplay`` is the zero-based ID number of the frame buffer device, which is transferred to the firmware to select a specific display on the Raspberry Pi 4, 400 and the Compute Module 4.

.. cpp:function:: void CBcmFrameBuffer::SetPalette (u8 nIndex, u16 nRGB565)
.. cpp:function:: void CBcmFrameBuffer::SetPalette32 (u8 nIndex, u32 nRGBA)

	Set the entry ``nIndex`` of the color palette to ``nRGB565`` or ``nRGBA``. The color palette is only used in in 4-bit or 8-bit pixel depth mode. The color palette must be set before :cpp:func:`Initialize()` is called, but can be updated later.

.. c:macro:: PALETTE_ENTRIES

	The maximum number of entries in the color palette in 4-bit or 8-bit depth mode (256). ``nIndex`` must be below this.

.. cpp:function:: boolean CBcmFrameBuffer::Initialize (void)

	Initializes the frame buffer device and starts the display. Returns ``TRUE`` on success.

.. note::

	This method does succeed on Raspberry Pi 1-3 and Zero, even when there is no display connected. On the Raspberry Pi 4, 400 and Compute Module 4 this method fails in this case.

.. cpp:function:: u32 CBcmFrameBuffer::GetWidth (void) const
.. cpp:function:: u32 CBcmFrameBuffer::GetHeight (void) const
.. cpp:function:: u32 CBcmFrameBuffer::GetVirtWidth(void) const
.. cpp:function:: u32 CBcmFrameBuffer::GetVirtHeight(void) const

	Return the physical or virtual size of the frame buffer in number of pixels.

.. cpp:function:: u32 CBcmFrameBuffer::GetPitch (void) const

	Returns the size of one pixel line in memory in number of bytes and may contain padding bytes.

.. cpp:function:: u32 CBcmFrameBuffer::GetDepth (void) const

	Returns the size of one pixel in memory in number of bits.

.. cpp:function:: u32 CBcmFrameBuffer::GetBuffer (void) const
.. cpp:function:: u32 CBcmFrameBuffer::GetSize (void) const

	Return the address and total size of the frame buffer in main memory.

.. cpp:function:: boolean CBcmFrameBuffer::UpdatePalette (void)

	Updates the color palette, after modifying it using :cpp:func:`SetPalette()` or :cpp:func:`SetPalette32()`. Returns ``TRUE`` on success. This method should be used only with a pixel depth of 4 or 8 bits.

.. cpp:function:: boolean CBcmFrameBuffer::SetVirtualOffset (u32 nOffsetX, u32 nOffsetY)

	Sets the offset of the top-left corner of the physically displayed image in a larger virtual frame buffer to [``nOffsetX``, ``nOffsetY``]. Returns ``TRUE`` on success.

.. cpp:function:: boolean CBcmFrameBuffer::WaitForVerticalSync (void)

	Waits for the next vertical synchronization (VSYNC) blanking gap. Returns ``TRUE`` on success.

.. cpp:function:: boolean CBcmFrameBuffer::SetBacklightBrightness(unsigned nBrightness)

	Sets the backlight brightness level of the display to ``nBrightness``. This has been tested with the Official 7" Raspberry Pi touchscreen only. The brightness level can be about 0..180 there. Returns ``TRUE`` on success.

.. cpp:function:: static unsigned CBcmFrameBuffer::GetNumDisplays (void)

	Returns to number of available displays, which is always 1 on models other than the Raspberry Pi 4, 400 or Compute Module 4.

CBcmRandomNumberGenerator
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/bcmrandom.h>

.. cpp:class:: CBcmRandomNumberGenerator

	This class is a driver for the built-in hardware random number generator.

.. cpp:function:: u32 CBcmRandomNumberGenerator::GetNumber (void)

	Returns a 32-bit random number.

.. note::

	Generating a random number takes a short while. For generating a large number of random numbers, you should use a polynomial random number generator, and seed it using this hardware random number generator.

CBcmWatchdog
^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/bcmwatchdog.h>

.. cpp:class:: CBcmWatchdog

	This class is a driver for the built-in watchdog device. It can be used to automatically restart a Raspberry Pi computer after program failure, or to restart it immediately from a specific partition.

.. cpp:function:: void CBcmWatchdog::Start (unsigned nTimeoutSeconds = MaxTimeoutSeconds)

	Starts the watchdog, to elapse after ``nTimeoutSeconds`` seconds. The system restarts after this timeout, if the watchdog is not re-triggered before.

.. cpp:var:: const unsigned CBcmWatchdog::MaxTimeoutSeconds = 15

	Is the maximum timeout in seconds.

.. cpp:function:: void CBcmWatchdog::Stop (void)

	Stops the watchdog. It will not elapse any more.

.. cpp:function:: void CBcmWatchdog::Restart (unsigned nPartition = PartitionDefault)

	Immediately restarts the system from the SD card partition with the number ``nPartition``, with these special values:

.. cpp:var:: const unsigned CBcmWatchdog::PartitionDefault = 0
.. cpp:var:: const unsigned CBcmWatchdog::PartitionHalt = 63

	``PartitionHalt`` halts the system, instead of restarting it.

.. cpp:function:: boolean CBcmWatchdog::IsRunning (void) const

	Returns ``TRUE``, if the watchdog is currently running.

.. cpp:function:: unsigned CBcmWatchdog::GetTimeLeft (void) const

	Returns the number of seconds left, until a restart will triggered.
