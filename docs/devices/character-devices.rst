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

CSerialDevice
"""""""""""""

.. code-block:: cpp

	#include <circle/serial.h>

.. cpp:class:: CSerialDevice : public CDevice

	This class is a driver for the PL011-compatible UART(s) of the Raspberry Pi. The Raspberry Pi 4 provides five of these serial devices, the other models only one. This driver cannot be used for the Mini-UART (AUX). The GPIO mapping is as follows (SoC numbers):

	=======	=======	=======	===================
	nDevice	TXD	RXD	Support
	=======	=======	=======	===================
	0	GPIO14	GPIO15	All boards
	0	GPIO32	GPIO33	Compute Modules
	0	GPIO36	GPIO37	Compute Modules
	1			None (AUX)
	2	GPIO0	GPIO1	Raspberry Pi 4 only
	3	GPIO4	GPIO5	Raspberry Pi 4 only
	4	GPIO8	GPIO9	Raspberry Pi 4 only
	5	GPIO12	GPIO13	Raspberry Pi 4 only
	=======	=======	=======	===================

	GPIO32/33 and GPIO36/37 can be selected with system option ``SERIAL_GPIO_SELECT``. GPIO0/1 are normally reserved for the ID EEPROM. Handshake lines CTS and RTS are not supported.

.. note::

	This driver can be used in two modes: polling or interrupt driven. The mode is selected with the parameter ``pInterruptSystem`` of the constructor.

.. c:macro:: SERIAL_BUF_SIZE

	This macro defines the size of the read and write ring buffers for the interrupt driver (default 2048). If you want to increase the buffer size, you have to specify a value, which is a power of two.

.. cpp:function:: CSerialDevice::CSerialDevice (CInterruptSystem *pInterruptSystem = 0, boolean bUseFIQ = FALSE, unsigned nDevice = 0)

	Constructs a ``CSerialDevice`` object. Multiple instances are possible on the Raspberry Pi 4. ``nDevice`` selects the used serial device (see the table above). ``pInterruptSystem`` is a pointer to interrupt system object, or 0 to use the polling driver. The interrupt driver uses the IRQ by default. Set ``bUseFIQ`` to ``TRUE`` to use the FIQ instead. This is recommended for higher baud rates.

.. cpp:function:: boolean CSerialDevice::Initialize (unsigned nBaudrate = 115200)

	Initializes the serial device and sets the baud rate to ``nBaudrate`` bits per second. Returns ``TRUE`` on success.

.. cpp:function:: int CSerialDevice::Write (const void *pBuffer, size_t nCount)

	Writes ``nCount`` bytes from ``pBuffer`` to be sent out via the serial device. Returns the number of bytes, successfully sent or queued for send, or < 0 on error. The following errors are defined:

.. c:macro:: SERIAL_ERROR_BREAK
.. c:macro:: SERIAL_ERROR_OVERRUN
.. c:macro:: SERIAL_ERROR_FRAMING

	Returned from ``Write()`` and ``Read()`` as a negative value. Please note, that these defined values are positive. You have to precede them with a minus for comparison.

.. cpp:function:: int CSerialDevice::Read (void *pBuffer, size_t nCount)

	Returns a maximum of ``nCount`` bytes, which have been received via the serial device, in ``pBuffer``. The returned ``int`` value is the number of received bytes, 0 if data is not available, or < 0 on error (see ``Write()``).

.. cpp:function:: unsigned CSerialDevice::GetOptions (void) const

	Returns the current serial options mask.

.. cpp:function:: void CSerialDevice::SetOptions (unsigned nOptions)

	Sets the serial options mask to ``nOptions``. These options are defined:

.. c:macro:: SERIAL_OPTION_ONLCR

	Translate NL to NL+CR on output (default)

.. cpp:function:: void CSerialDevice::RegisterMagicReceivedHandler (const char *pMagic, TMagicReceivedHandler *pHandler)

	Registers a magic received handler ``pHandler``, which is called, when the string ``pMagic`` is found in the received data. ``pMagic`` must remain valid after return from this method. This method does only work with interrupt driver.

.. cpp:type:: void CSerialDevice::TMagicReceivedHandler (void)

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
