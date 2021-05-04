Character devices
~~~~~~~~~~~~~~~~~

Character devices usually accept and/or deliver a stream of characters via ``Write()`` and ``Read()`` calls. In Circle some character devices use a register-able callback handler instead of the ``Read()`` method, to deliver the received data. This makes time-consuming polling operations superfluous for these devices.

CScreenDevice
"""""""""""""

.. code-block:: cpp

	#include <circle/screen.h>

.. cpp:class:: CScreenDevice : public CDevice

	This class can be used to write characters to the (usually HDMI) screen, which is connected to the Raspberry Pi computer. The screen is treated like a terminal and provides a number of control sequences (see ``Write()``). This device has the name ``"ttyN"`` (N >= 1) in the device name service.

.. cpp:function:: CScreenDevice::CScreenDevice (unsigned nWidth, unsigned nHeight, boolean bVirtual = FALSE, unsigned nDisplay = 0)

	Constructs an instance of ``CScreenDevice``. ``nWidth`` is the screen width and ``nHeight`` the screen height in number of pixels. Set both parameters to 0 to auto-detect the default resolution of the screen, which is usually the maximum resolution of the used monitor. ``bVirtual`` should be set to ``FALSE`` in any case. The Raspberry Pi 4 supports more than one display. ``nDisplay`` is the zero-based display number here. Multiple instances of ``CScreenDevice`` are possible here.

.. note::

	You have to set ``max_framebuffers=2`` in `config.txt` to use two displays on the Raspberry Pi 4.

.. cpp:function:: boolean CScreenDevice::Initialize (void)

	Initializes the instance of ``CScreenDevice`` and clears the screen. Returns ``TRUE`` on success.

.. cpp:function:: unsigned CScreenDevice::GetWidth (void) const

	Returns the screen width in number of pixels.

.. cpp:function:: unsigned CScreenDevice::GetHeight (void) const

	Returns the screen height in number of pixels.

.. cpp:function:: unsigned CScreenDevice::GetColumns (void) const

	Returns the screen width in number of character columns.

.. cpp:function:: unsigned CScreenDevice::GetRows (void) const

	Returns the screen height in number of character rows.

.. cpp:function:: int CScreenDevice::Write (const void *pBuffer, size_t nCount)

	Writes ``nCount`` characters from ``pBuffer`` to the screen. Returns the number of written characters. This method supports several escape sequences:

	==============	======================================	=============
	Sequence	Description				Remarks
	==============	======================================	=============
	\\E[B		Cursor down one line
	\\E[H		Cursor home
	\\E[A		Cursor up one line
	\\E[%d;%dH	Cursor move to row %1 and column %2	starting at 1
	^H		Cursor left one character
	\\E[D		Cursor left one character
	\\E[C		Cursor right one character
	^M		Carriage return
	\\E[J		Clear to end of screen
	\\E[K		Clear to end of line
	\\E[%dX		Erase %1 characters starting at cursor
	^J		Carriage return/linefeed
	\\E[0m		End of bold or half bright mode
	\\E[1m		Start bold mode
	\\E[2m		Start half bright mode
	^I		Move to next hardware tab
	\\E[?25h	Normal cursor visible
	\\E[?25l	Cursor invisible
	\\E[%d;%dr	Set scroll region from row %1 to %2	starting at 1
	==============	======================================	=============

	^X = Control character, \\E = Escape (\\x1b), %d = Numerical parameter (ASCII)

.. cpp:function:: void CScreenDevice::SetPixel (unsigned nPosX, unsigned nPosY, TScreenColor Color)

	Sets the pixel at position ``nPosX, nPosY`` (based on ``0, 0``) to ``Color``. The color value depends on the macro value ``DEPTH``, which can be defined as 8, 16 (default) or 32 in `include/circle/screen.h` or `Config.mk`. Circle defines the following standard color values:

	* BLACK_COLOR (black)
	* NORMAL_COLOR (white)
	* HIGH_COLOR (red)
	* HALF_COLOR (dark blue)

.. c:macro:: COLOR16(r, g, b)

	Defines a color value for ``DEPTH == 16``. r/g/b can be 0-31.

.. c:macro:: COLOR32(r, g, b, alpha)

	Defines a color value for ``DEPTH == 32``. r/g/b can be 0-255. alpha is usually 255.

.. cpp:function:: TScreenColor CScreenDevice::GetPixel (unsigned nPosX, unsigned nPosY)

	Returns the pixel color value at position ``nPosX, nPosY`` (based on ``0, 0``).

.. cpp:function:: void CScreenDevice::Rotor (unsigned nIndex, unsigned nCount)

	Displays a rotating symbol in the upper right corner of the screen. ``nIndex`` is the index of the rotor to be displayed (0..3). ``nCount`` is the phase (angle) of the current rotor symbol (0..3).

.. cpp:function:: CBcmFrameBuffer *CScreenDevice::GetFrameBuffer (void)

	Returns a pointer to the member of the type ``CBcmFrameBuffer``, which can be used to directly manipulate the frame buffer.

.. CSerialDevice
.. CUSBKeyboardDevice
.. CMouseDevice
.. CUSBGamePadDevice
.. CUSBSerialDevice
.. CUSBPrinterDevice
.. CTouchScreenDevice
.. CConsole
.. CHD44780Device

CNullDevice
"""""""""""

.. code-block:: cpp

	#include <circle/nulldevice.h>

.. cpp:class:: CNullDevice : public CDevice

	This class implements the null device, which accepts all written characters and returns 0 (EOF) on read. It can be used instead of other character device classes, for instance as target for the :ref:`System log`. This device has the name ``"null"`` in the device name service.

.. cpp:function:: int CNullDevice::Read (void *pBuffer, size_t nCount)

	Returns always 0 (EOF).

.. cpp:function:: int CNullDevice::Write (const void *pBuffer, size_t nCount)

	Returns the number of written bytes, but ignores them.
