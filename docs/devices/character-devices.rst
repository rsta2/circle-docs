Character devices
~~~~~~~~~~~~~~~~~

Character devices usually accept and/or deliver a stream of characters via ``Write()`` and ``Read()`` calls. In Circle some character devices use a register-able callback handler instead of the ``Read()`` method, to deliver the received data. This makes time-consuming polling operations superfluous for these devices.

CScreenDevice
^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/screen.h>

.. cpp:class:: CScreenDevice : public CDevice

	This class can be used to write characters to the (usually HDMI) screen, which is connected to the Raspberry Pi computer. The screen is treated like a terminal and provides a number of control sequences (see :cpp:func:`CTerminalDevice::Write()`). This device has the name ``"ttyN"`` (N >= 1) in the device name service.

.. cpp:function:: CScreenDevice::CScreenDevice (unsigned nWidth, unsigned nHeight, const TFont &rFont = DEFAULT_FONT, CCharGenerator::TFontFlags FontFlags = CCharGenerator::FontFlagsNone, unsigned nDisplay = 0)

	Constructs an instance of ``CScreenDevice``. ``nWidth`` is the screen width and ``nHeight`` the screen height in number of pixels. Set both parameters to 0 to auto-detect the default resolution of the screen, which is usually the maximum resolution of the used monitor. ``rFont`` is the display font to be used with the flags ``FontFlags``. The Raspberry Pi 4 supports more than one display. ``nDisplay`` is the zero-based display number here. Multiple instances of ``CScreenDevice`` are possible here.

.. note::

	You have to set ``max_framebuffers=2`` in `config.txt` to use two displays on the Raspberry Pi 4.

.. cpp:function:: boolean CScreenDevice::Initialize (void)

	Initializes the instance of ``CScreenDevice`` and clears the screen. Returns ``TRUE`` on success.

.. cpp:function:: boolean CScreenDevice::Resize (unsigned nWidth, unsigned nHeight)

	Initializes the instance of ``CScreenDevice`` with a new size again and clears the screen. ``nWidth`` is the new screen width and ``nHeight`` the new screen height in number of pixels. Returns ``TRUE`` on success. When ``FALSE`` is returned, the width and/or height are not supported. The object is in an uninitialized state then and must not be used, but ``Resize()`` can be called again with other parameters.

.. cpp:function:: unsigned CScreenDevice::GetWidth (void) const

	Returns the screen width in number of pixels.

.. cpp:function:: unsigned CScreenDevice::GetHeight (void) const

	Returns the screen height in number of pixels.

.. cpp:function:: unsigned CScreenDevice::GetColumns (void) const

	Returns the screen width in number of character columns.

.. cpp:function:: unsigned CScreenDevice::GetRows (void) const

	Returns the screen height in number of character rows.

.. cpp:function:: int CScreenDevice::Write (const void *pBuffer, size_t nCount)

	Writes ``nCount`` characters from ``pBuffer`` to the screen. Returns the number of written characters. This method supports several escape sequences (see :cpp:func:`CTerminalDevice::Write()`).

.. cpp:function:: void CScreenDevice::SetPixel (unsigned nPosX, unsigned nPosY, TScreenColor Color)

	Sets the pixel at position ``nPosX, nPosY`` (based on ``0, 0``) to ``Color``. The color value depends on the macro value ``DEPTH``, which can be defined as 8, 16 (default) or 32 in `include/circle/screen.h` or `Config.mk`. Circle defines the following standard color values:

	* BLACK_COLOR (black)
	* NORMAL_COLOR (white)
	* HIGH_COLOR (red)
	* HALF_COLOR (dark blue)

	The following specific color values are defined:

	* RED_COLOR
	* GREEN_COLOR
	* YELLOW_COLOR
	* BLUE_COLOR
	* MAGENTA_COLOR
	* CYAN_COLOR
	* WHITE_COLOR
	* BRIGHT_BLACK_COLOR
	* BRIGHT_RED_COLOR
	* BRIGHT_GREEN_COLOR
	* BRIGHT_YELLOW_COLOR
	* BRIGHT_BLUE_COLOR
	* BRIGHT_MAGENTA_COLOR
	* BRIGHT_CYAN_COLOR
	* BRIGHT_WHITE_COLOR

.. c:macro:: COLOR16(r, g, b)

	Defines a color value for ``DEPTH == 16``. r/g/b can be 0-31.

.. c:macro:: COLOR32(r, g, b, alpha)

	Defines a color value for ``DEPTH == 32``. r/g/b can be 0-255. alpha is usually 255.

.. cpp:function:: TScreenColor CScreenDevice::GetPixel (unsigned nPosX, unsigned nPosY)

	Returns the pixel color value at position ``nPosX, nPosY`` (based on ``0, 0``).

.. cpp:function:: void CScreenDevice::Rotor (unsigned nIndex, unsigned nCount)

	Displays some rotating pixels in the upper right corner of the screen. ``nIndex`` is the index of the rotor to be displayed (0..3). ``nCount`` is the phase (angle) of the current rotor symbol (0..3).

.. cpp:function:: void CScreenDevice::SetCursorBlock (boolean bCursorBlock)

	Enable block cursor (``TRUE``) instead of the default underline cursor (``FALSE``).

.. cpp:function:: void CScreenDevice::Update (unsigned nMillis = 0)

	Updates the frame buffer from the internal buffer. Updates only, when ``nMillis`` ms were passed since the previous update (0 to disable). Once this method has been called, the frame buffer is only updated, when this method is called and only in the given time interval.

.. cpp:function:: CBcmFrameBuffer *CScreenDevice::GetFrameBuffer (void)

	Returns a pointer to the member of the type :cpp:class:`CBcmFrameBuffer`, which can be used to directly manipulate the frame buffer.

CTerminalDevice
^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/terminal.h>

.. cpp:class:: CTerminalDevice : public CDevice

	This class manages a scrolling terminal display on a dot-matrix display. It provides a number of control sequences (see :cpp:func:`CTerminalDevice::Write()`). This device has the name ``"ttyN"`` (N >= 1) in the device name service.

.. cpp:function:: CTerminalDevice::CTerminalDevice (CDisplay *pDisplay, unsigned nDeviceIndex = 0, const TFont &rFont = DEFAULT_FONT, CCharGenerator::TFontFlags FontFlags = CCharGenerator::FontFlagsNone)

	Constructs a terminal device on the dot-matrix display ``pDisplay`` with the device name ``"ttyN"`` (N = ``nDeviceIndex`` + 1). It uses the display font ``rFont`` with the flags ``FontFlags``.

.. cpp:function:: boolean CTerminalDevice::Initialize (void)

	Initializes the instance of ``CTerminalDevice`` and clears the display. Returns ``TRUE`` on success.

.. cpp:function:: unsigned CTerminalDevice::GetWidth (void) const

	Returns the terminal width in pixels.

.. cpp:function:: unsigned CTerminalDevice::GetHeight (void) const

	Returns the terminal height in pixels.

.. cpp:function:: unsigned CTerminalDevice::GetColumns (void) const

	Returns the terminal width in characters.

.. cpp:function:: unsigned CTerminalDevice::GetRows (void) const

	Returns the terminal height in characters.

.. cpp:function:: int CTerminalDevice::Write (const void *pBuffer, size_t nCount)

	Writes ``nCount`` characters from ``pBuffer`` to the terminal. Returns the number of written characters. This method supports several escape sequences:

	==============	======================================	=====================
	Sequence	Description				Remarks
	==============	======================================	=====================
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
	\\E[0m		End of bold, half bright, reverse mode
	\\E[1m		Start bold mode
	\\E[2m		Start half bright mode
	\\E[7m		Start reverse video mode
	\\E[27m		Same as \\E[0m
	\\E[%dm		Set foreground color			%d = 30-37 or 90-97
	\\E[%dm		Set background color			%d = 40-47 or 100-107
	^I		Move to next hardware tab
	\\E[?25h	Normal cursor visible
	\\E[?25l	Cursor invisible
	\\E[%d;%dr	Set scroll region from row %1 to %2	starting at 1
	\\Ed+		Start autopage mode
	\\Ed*		End autopage mode
	==============	======================================	=====================

	^X = Control character, \\E = Escape (\\x1b), %d = Numerical parameter (ASCII)

.. cpp:function:: void CTerminalDevice::SetCursorBlock (boolean bCursorBlock)

	Enables a block cursor instead of the default underline cursor, when called with ``bCursorBlock`` set to ``TRUE``.

.. cpp:function:: void CTerminalDevice::Update (unsigned nMillis = 0)

	Updates the display from the internal display buffer. Updates only, when ``nMillis`` ms were passed since the previous update (0 to disable). Once this method has been called, the display is only updated, when this method is called and only in the given time interval.

.. cpp:function:: void CTerminalDevice::SetPixel (unsigned nPosX, unsigned nPosY, TTerminalColor Color)

	Sets a pixel to a logical color. ``nPosX`` is the X-Position of the pixel (based on 0). ``nPosY`` is the Y-Position of the pixel (based on 0). ``Color`` is the logical color to be set, which has this type:

.. cpp:type:: TTerminalColor

	Same as :cpp:type:`CDisplay::TColor`.

.. c:macro:: TERMINAL_COLOR(red, green, blue)

	Same as :c:macro:`DISPLAY_COLOR`.

.. cpp:function:: TTerminalColor CTerminalDevice::GetPixel (unsigned nPosX, unsigned nPosY)

	Returns the logical color value of the pixel at the X-Position ``nPosX`` (based on 0) and the Y-Position ``nPosY`` (based on 0). Returns ``CDisplay::Black``, if the read physical color value cannot be converted into a logical color.

.. cpp:function:: void CTerminalDevice::SetPixel (unsigned nPosX, unsigned nPosY, CDisplay::TRawColor nColor)

	Sets a pixel to the physical (raw) color ``nColor``. ``nPosX`` is the X-Position of the pixel (based on 0). ``nPosY`` is the Y-Position of the pixel (based on 0).

.. cpp:function:: CDisplay *CTerminalDevice::GetDisplay (void)

	Returns a pointer to the display, we are working on.

CSerialDevice
^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/serial.h>

.. cpp:class:: CSerialDevice : public CDevice

	This class is a driver for the PL011-compatible UART(s) of the Raspberry Pi. The Raspberry Pi 4 provides five of these serial devices, the other models only one. This driver cannot be used for the Mini-UART (AUX). The GPIO mapping for Raspberry Pi 1-4 is as follows (SoC numbers):

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

	For Raspberry Pi 5 there is this different mapping:

	=======	=======	=======	===================
	nDevice	TXD	RXD	Support
	=======	=======	=======	===================
	0	GPIO14	GPIO15	Raspberry Pi 5 only
	1	GPIO0	GPIO1	Raspberry Pi 5 only
	2	GPIO4	GPIO5	Raspberry Pi 5 only
	3	GPIO8	GPIO9	Raspberry Pi 5 only
	4	GPIO12	GPIO13	Raspberry Pi 5 only
	5	GPIO36	GPIO37	None
	6			None
	7			None
	8			None
	9			None
	10	UART	UART	Raspberry Pi 5 only
	=======	=======	=======	===================

	UART is the dedicated 3-pin JST UART connector.

	This device has the name ``"ttySN"`` (N >= 1) in the device name service, where ``N = nDevice+1``.

.. note::

	This driver can be used in two modes: polling or interrupt driven. The mode is selected with the parameter ``pInterruptSystem`` of the constructor.

.. c:macro:: SERIAL_BUF_SIZE

	This macro defines the size of the read and write ring buffers for the interrupt driver (default 2048). If you want to increase the buffer size, you have to specify a value, which is a power of two.

.. c:macro:: SERIAL_DEVICE_DEFAULT

	This macro defines the default serial device, if the ``nDevice`` parameter is not specified in the constructor. By default it has the value 0 on Raspberry Pi 1-4 and 10 on Raspberry Pi 5. You can re-define this macro in the file *Config.mk*, if you want to change this.

.. cpp:function:: CSerialDevice::CSerialDevice (CInterruptSystem *pInterruptSystem = 0, boolean bUseFIQ = FALSE, unsigned nDevice = SERIAL_DEVICE_DEFAULT)

	Constructs a ``CSerialDevice`` object. Multiple instances are possible on the Raspberry Pi 4. ``nDevice`` selects the used serial device (see the table above). ``pInterruptSystem`` is a pointer to interrupt system object, or 0 to use the polling driver. The interrupt driver uses the IRQ by default. Set ``bUseFIQ`` to ``TRUE`` to use the FIQ instead. This is recommended for higher baud rates. The parameter ``bUseFIQ`` is ignored on the Raspberry Pi 5.

.. cpp:function:: boolean CSerialDevice::Initialize (unsigned nBaudrate = 115200, unsigned nDataBits = 8, unsigned nStopBits = 1, TParity Parity = ParityNone)

	Initializes the serial device and sets the baud rate to ``nBaudrate`` bits per second. ``nDataBits`` selects the number of data bits (5..8, default 8) and ``nStopBits`` the number of stop bits (1..2, default 1). ``Parity`` can be ``CSerialDevice::ParityNone`` (default), ``CSerialDevice::ParityOdd``, ``CSerialDevice::ParityEven``, ``CSerialDevice::ParitySpace`` or ``CSerialDevice::ParityMark``. Returns ``TRUE`` on success.

.. cpp:function:: int CSerialDevice::Write (const void *pBuffer, size_t nCount)

	Writes ``nCount`` bytes from ``pBuffer`` to be sent out via the serial device. Returns the number of bytes, successfully sent or queued for send, or < 0 on error. The following errors are defined:

.. c:macro:: SERIAL_ERROR_BREAK
.. c:macro:: SERIAL_ERROR_OVERRUN
.. c:macro:: SERIAL_ERROR_FRAMING
.. c:macro:: SERIAL_ERROR_PARITY

	Returned from ``Write()`` and ``Read()`` as a negative value. Please note, that these defined values are positive. You have to precede them with a minus for comparison.

.. cpp:function:: int CSerialDevice::Read (void *pBuffer, size_t nCount)

	Returns a maximum of ``nCount`` bytes, which have been received via the serial device, in ``pBuffer``. The returned ``int`` value is the number of received bytes, 0 if data is not available, or < 0 on error (see ``Write()``).

.. cpp:function:: unsigned CSerialDevice::GetOptions (void) const

	Returns the current serial options mask.

.. cpp:function:: void CSerialDevice::SetOptions (unsigned nOptions)

	Sets the serial options mask to ``nOptions``. These options are defined:

.. c:macro:: SERIAL_OPTION_ONLCR

	Translate NL to NL+CR on output (default)

.. cpp:function:: void CSerialDevice::SetParity (TParity Parity)

	Modifies the parity setting to ``Parity``, which can be ``CSerialDevice::ParityNone``, ``CSerialDevice::ParityOdd``, ``CSerialDevice::ParityEven``, ``CSerialDevice::ParitySpace`` (0) or ``CSerialDevice::ParityMark`` (1). This will disable the UART for a small time.

.. cpp:function:: boolean CSerialDevice::IsTransmitting (void) const

	Returns ``TRUE``, if the transmitter is busy, transmitting characters.

.. cpp:function:: void CSerialDevice::RegisterCharReceivedHandler (TCharReceivedHandler *pHandler, void *pParam)

	Registers a character received handler ``pHandler``, which is called, when a character has been received. This does only work with interrupt driver, and :cpp:func:`Read()` does not work, when this handler is registered. ``pParam`` is a user parameter, which is handed over to the handler.

.. cpp:type:: void CSerialDevice::TCharReceivedHandler (u8 uchChar, int nStatus, void *pParam)

	``uchChar`` is the character code of the received character. ``nStatus`` is an error code (see :cpp:func:`Write()`) as a negative value, or 0 (no error). ``pParam`` is the user parameter, which was handed over to :cpp:func:`RegisterCharReceivedHandler()`.

.. cpp:function:: void CSerialDevice::RegisterMagicReceivedHandler (const char *pMagic, TMagicReceivedHandler *pHandler)

	Registers a magic received handler ``pHandler``, which is called, when the string ``pMagic`` is found in the received data. ``pMagic`` must remain valid after return from this method. This method does only work with interrupt driver.

.. cpp:type:: void CSerialDevice::TMagicReceivedHandler (void)

CUSBKeyboardDevice
^^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/usb/usbkeyboard.h>

.. cpp:class:: CUSBKeyboardDevice : public CUSBHIDDevice

	This class is a driver for USB standard keyboards. An instance of this class is automatically created, when a compatible USB keyboard is found in the USB device enumeration process. Therefore only the class methods needed to use the keyboard by an application are described here, not the methods used for initialization. This device has the name ``"ukbdN"`` (N >= 1) in the device name service.

.. note::

	This driver class supports two keyboard modes: cooked and raw mode. In cooked mode the driver reports ISO-8859-1 character strings and the keyboard LEDs are handled automatically. There are six available keyboard maps (DE, ES, FR, IT, UK, US), which can be selected with the ``DEFAULT_KEYMAP`` configurable system option or the ``keymap=`` setting in the file `cmdline.txt` on the SD card.

	In raw mode the driver reports the raw USB keyboard codes and modifier information and the LEDs have to be set manually by the application.

.. cpp:function:: void CUSBKeyboardDevice::RegisterKeyPressedHandler (TKeyPressedHandler *pKeyPressedHandler)

	Registers a function, which gets called, when a key is pressed in cooked mode:

.. c:type:: void TKeyPressedHandler (const char *pString)

	``pString`` points to a C-string, which contains the ISO-8859-1 code of the pressed key. This is normally only one character, but can be one of the following control sequences for special purpose keys:

	==============	=========
	Sequence	Key
	==============	=========
	\\E		Escape
	\\177		Backspace
	^I		Tabulator
	^J		Return
	\\E[2~		Insert
	\\E[1~		Home
	\\E[5~		PageUp
	\\E[3~		Delete
	\\E[4~		End
	\\E[6~		PageDown
	\\E[A		Up
	\\E[B		Down
	\\E[D		Left
	\\E[C		Right
	\\E[[A		F1
	\\E[[B		F2
	\\E[[C		F3
	\\E[[D		F4
	\\E[[E		F5
	\\E[17~		F6
	\\E[18~		F7
	\\E[19~		F8
	\\E[20~		F9
	\\E[G		KP_Center
	\\E[1;5A	Ctrl-Up
	\\E[1;5B	Ctrl-Down
	\\E[1;5D	Ctrl-Left
	\\E[1;5C	Ctrl-Right
	\\E[1;5H	Ctrl-Home
	\\E[1;5F	Ctrl-End
	\\E[5;5~	Ctrl-PageUp
	\\E[6;5~	Ctrl-PageDown
	==============	=========

	^X = Control character, \\E = Escape (\\x1b), \\nnn = Octal code

.. cpp:function:: void CUSBKeyboardDevice::RegisterSelectConsoleHandler (TSelectConsoleHandler *pSelectConsoleHandler)

	Registers a function, which gets called, when the `Alt` key is pressed together with a function key `F1` to `F12` in cooked mode. This is used to select the console in some systems.

.. c:type:: void TSelectConsoleHandler (unsigned nConsole)

	``nConsole`` is the number of the console to select (0-11).

.. cpp:function:: void CUSBKeyboardDevice::RegisterShutdownHandler (TShutdownHandler *pShutdownHandler)

	Registers a function, which gets called, when the `Ctrl`, `Alt` and `Del` keys are pressed together in cooked mode. This is used to shutdown or reboot some systems.

.. c:type:: void TShutdownHandler (void)

.. cpp:function:: void CUSBKeyboardDevice::UpdateLEDs (void)

	In cooked mode this method has to be called from TASK_LEVEL from time to time, so that the status of the keyboard LEDs can be updated.

.. cpp:function:: u8 CUSBKeyboardDevice::GetLEDStatus (void) const

	Returns the LED status mask of the keyboard in cooked mode, with the following bit masks:

	* LED_NUM_LOCK
	* LED_CAPS_LOCK
	* LED_SCROLL_LOCK

.. cpp:function:: boolean CUSBKeyboardDevice::SetLEDs (u8 ucStatus)

	Sets the keyboard LEDs according to the bit mask values listed under ``GetLEDStatus()``. This method can be called on TASK_LEVEL only. It works in cooked and raw mode.

.. cpp:function:: void CUSBKeyboardDevice::RegisterKeyStatusHandlerRaw (TKeyStatusHandlerRaw *pKeyStatusHandlerRaw, boolean bMixedMode = FALSE)

	Registers a function, which gets called to report the keyboard status in raw mode. If ``bMixedMode`` is ``FALSE``, then the cooked mode handlers are ignored. You can set it to ``TRUE`` to be able to use cooked mode and raw mode handlers together.

.. note::

	It depends on the used USB keyboard, if the raw status handler gets called on status changes only or repeatedly after some delay too. The application must be able to handle both cases.

.. c:type:: void TKeyStatusHandlerRaw (unsigned char ucModifiers, const unsigned char RawKeys[6])

	``RawKeys`` contains up to six raw USB keyboard codes or zero in each byte. ``ucModifiers`` contains a mask of the pressed modifier keys, with the following bit masks:

	* LCTRL
	* LSHIFT
	* ALT
	* LWIN
	* RCTRL
	* RSHIFT
	* ALTGR
	* RWIN

.. cpp:function:: void CUSBKeyboardDevice::RegisterKeyStatusHandlerRaw (TKeyStatusHandlerRawEx *pKeyStatusHandlerRaw, boolean bMixedMode, void *pArg)

	Alternative version of ``RegisterKeyStatusHandlerRaw()``, which gets an additional user argument, which is handed over to this raw mode keyboard status handler:

.. c:type:: void TKeyStatusHandlerRawEx (unsigned char	ucModifiers, const unsigned char RawKeys[6], void *pArg)

.. cpp:function:: void CUSBKeyboardDevice::UnregisterKeyStatusHandlerRaw (void)

	Remove registration of a previously registered raw mode keyboard status handler.

CMouseDevice
^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/input/mouse.h>

.. cpp:class:: CMouseDevice : public CDevice

	This class is the generic mouse interface device. An instance of this class is automatically created, when a compatible USB mouse or USB gamepad with touchpad is found in the USB device enumeration process. Therefore only the class methods, needed to use the mouse by an application, are described here, not the method used for creation. This device has the name ``"mouseN"`` (N >= 1) in the device name service.

.. note::

	This class supports two mouse modes: cooked and raw mode. In cooked mode a mouse cursor is shown on the screen and automatically controlled by the driver, which reports several mouse events (down, up, move, wheel).

	In raw mode the driver directly reports the raw mouse displacement, button and wheel information.

.. note::

	The Raspberry Pi 5 does not support a mouse cursor in its firmware. Therefore a mouse cursor is not shown on this model, but the cooked mode can still be used to maintain the mouse position, without showing a cursor. The :ref:`LVGL` support implements a LVGL software cursor on the Raspberry Pi 5.

.. cpp:function:: boolean CMouseDevice::Setup (CDisplay *pDisplay, boolean bCursor = TRUE)

	Setup mouse device in cooked mode. Show the mouse on ``pDisplay``. Set ``bCursor`` to ``TRUE`` to support a mouse cursor. Returns ``FALSE`` on failure. This method must be called first in the setup process for a mouse in cooked mode.

.. cpp:function:: void CMouseDevice::Release (void)

	Undo ``Setup()``. Call this before resizing the screen!

.. cpp:function:: void CMouseDevice::RegisterEventHandler (TMouseEventHandler *pEventHandler)

	Registers an mouse event handler in cooked mode. ``pEventHandler`` is a pointer to the event handler with the following prototype:

.. c:type:: void TMouseEventHandler (TMouseEvent Event, unsigned nButtons, unsigned nPosX, unsigned nPosY, int nWheelMove)

	``nPosX`` is the X-coordinate of the current mouse cursor position in pixels (0 is on the left border). ``nPosY`` is the Y-coordinate of the position in pixels (0 is on the top border). These parameters are always valid (also in button and wheel events). ``Event`` is the reported mouse event with these possible values:

.. c:enum:: TMouseEvent

	======================	======================================	====================
	Event			Reports					Parameter
	======================	======================================	====================
	MouseEventMouseDown	one button, which has been pressed	``nButtons``
	MouseEventMouseUp	one button, which has been released	``nButtons``
	MouseEventMouseMove	a mouse move to a new screen position	``nPosX``, ``nPosY``
	MouseEventMouseWheel	a wheel move (raw displacement -/+)	``nWheelMove``
	======================	======================================	====================

.. c:macro:: MOUSE_BUTTON_LEFT
.. c:macro:: MOUSE_BUTTON_RIGHT
.. c:macro:: MOUSE_BUTTON_MIDDLE
.. c:macro:: MOUSE_BUTTON_SIDE1
.. c:macro:: MOUSE_BUTTON_SIDE2

	Bit masks for the ``nButtons`` parameter.

.. cpp:function:: boolean CMouseDevice::SetCursor (unsigned nPosX, unsigned nPosY)

	Sets the mouse cursor to a specific screen position in cooked mode. ``nPosX`` is the X-coordinate of the position in pixels (0 is on the left border). ``nPosY`` is the Y-coordinate of the position in pixels (0 is on the top border). Returns ``FALSE`` on failure.

.. cpp:function:: boolean CMouseDevice::ShowCursor (boolean bShow)

	Switches the mouse cursor on the screen on or off in cooked mode. Set ``bShow`` to ``TRUE`` to show the mouse cursor. Returns the previous state.

.. cpp:function:: void CMouseDevice::UpdateCursor (void)

	This method must be called frequently from ``TASK_LEVEL`` in cooked mode to update the mouse cursor on screen.

.. cpp:function:: void CMouseDevice::RegisterStatusHandler (TMouseStatusHandler *pStatusHandler)

	Registers the mouse status handler in raw mode. ``pStatusHandler`` is a pointer to the status handler with the following prototype:

.. c:type:: void TMouseStatusHandler (unsigned nButtons, int nDisplacementX, int nDisplacementY, int nWheelMove)

	``nButtons`` is the raw button mask reported from the mouse device. Use the same bit masks ``MOUSE_BUTTON_LEFT`` etc. listed above. ``nDisplacementX`` and ``nDisplacementY`` are the raw displacement values reported from the mouse device, with these limits:

.. c:macro:: MOUSE_DISPLACEMENT_MIN
.. c:macro:: MOUSE_DISPLACEMENT_MAX

.. cpp:function:: void CMouseDevice::RegisterStatusHandler (TMouseStatusHandlerEx *pStatusHandler, void *pArg)

	Alternative version of ``RegisterStatusHandler()``, which gets an additional user argument, which is handed over to this raw mode mouse status handler:

.. c:type:: void TMouseStatusHandlerEx (unsigned nButtons, int nDisplacementX, int nDisplacementY, int nWheelMove, void *pArg)

.. cpp:function:: unsigned CMouseDevice::GetButtonCount (void) const

	Returns the number of supported buttons for this mouse device.

.. cpp:function:: boolean CMouseDevice::HasWheel (void) const

	Returns ``TRUE``, if the mouse supports a mouse wheel.

CUSBGamePadDevice
^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/usb/usbgamepad.h>

.. cpp:class:: CUSBGamePadDevice : public CUSBHIDDevice

	This class is the base class for USB gamepad drivers and the generic application interface for USB gamepads. There are a number of different derived classes, which implement the drivers for specific gamepads. Circle automatically creates an instance of the right class, when a compatible USB gamepad is found in the USB device enumeration process. Therefore only the class methods, needed to use the gamepad by an application, are described here, not the methods used for initialization. This device has the name ``"upadN"`` (N >= 1) in the device name service.

.. note::

	Circle supports gamepads, which are compatible with the USB HID-class specification and some other gamepads. To use a specific gamepad an application must normally know the mapping of the gamepad controls (buttons, axes etc.) to the gamepad report items. This mapping is not defined by the specification, but known for some widely available gamepads. A supported gamepad with a known mapping is called a "known gamepad" here and its driver offers additional services. The properties of a gamepad can be requested using ``GetProperties()``.

	The *sample/27-usbgamepad* is working with all supported gamepads, but has limited function. The *sample/37-showgamepad* is only working with with known gamepads with more function.

.. c:struct:: TGamePadState

	This structure is used to report the current state of the gamepad controls to the application. It can be fetched using ``GetInitialState()`` or by registering a status handler using ``RegisterStatusHandler()``.

.. code-block:: c

	#define MAX_AXIS	16
	#define MAX_HATS	6

	struct TGamePadState
	{
		int naxes;		// Number of available axes and analog buttons
		struct
		{
			int value;	// Current position value
			int minimum;	// Minimum position value (normally 0)
			int maximum;	// Maximum position value (normally 255)
		}
		axes[MAX_AXIS];		// Array of axes and analog buttons

		int nhats;		// Number of available hat controls
		int hats[MAX_HATS];	// Current position value of hat controls

		int nbuttons;		// Number of available digital buttons
		unsigned buttons;	// Current status of digital buttons (bit mask)

		int acceleration[3];	// Current state of acceleration sensor (x, y, z)
		int gyroscope[3];	// Current state of gyroscope sensor (x, y, z)
	};

	#define GAMEPAD_AXIS_DEFAULT_MINIMUM	0
	#define GAMEPAD_AXIS_DEFAULT_MAXIMUM	255

.. c:enum:: TGamePadButton

	Defines bit masks for the ``TGamePadState::buttons`` field for known gamepads. If the digital button is pressed, the respective bit is set. The following buttons are defined:

	======================	======================================	=================
	Digital button		Alias					Comment
	======================	======================================	=================
	GamePadButtonGuide	GamePadButtonXbox, GamePadButtonPS,
				GamePadButtonHome
	GamePadButtonLT		GamePadButtonL2, GamePadButtonLZ
	GamePadButtonRT		GamePadButtonR2, GamePadButtonRZ
	GamePadButtonLB		GamePadButtonL1, GamePadButtonL
	GamePadButtonRB		GamePadButtonR1, GamePadButtonR
	GamePadButtonY		GamePadButtonTriangle
	GamePadButtonB		GamePadButtonCircle
	GamePadButtonA		GamePadButtonCross
	GamePadButtonX		GamePadButtonSquare
	GamePadButtonSelect	GamePadButtonBack, GamePadButtonShare,
				GamePadButtonCapture
	GamePadButtonL3		GamePadButtonSL				left axis button
	GamePadButtonR3		GamePadButtonSR				right axis button
	GamePadButtonStart	GamePadButtonOptions			optional
	GamePadButtonUp
	GamePadButtonRight
	GamePadButtonDown
	GamePadButtonLeft
	GamePadButtonPlus						optional
	GamePadButtonMinus						optional
	GamePadButtonTouchpad						optional
	======================	======================================	=================

.. c:macro:: GAMEPAD_BUTTONS_STANDARD
.. c:macro:: GAMEPAD_BUTTONS_ALTERNATIVE
.. c:macro:: GAMEPAD_BUTTONS_WITH_TOUCHPAD

	Number of digital buttons (19, 21 or 22) for known gamepads with different properties.

.. c:enum:: TGamePadAxis

	Defines indices for the ``TGamePadState::axes`` field for known gamepads. This field covers the state information of axes and analog buttons. The following axes are defined:

	==============================	===================
	Axes				Alias
	==============================	===================
	GamePadAxisLeftX
	GamePadAxisLeftY
	GamePadAxisRightX
	GamePadAxisRightY
	GamePadAxisButtonLT		GamePadAxisButtonL2
	GamePadAxisButtonRT		GamePadAxisButtonR2
	GamePadAxisButtonUp
	GamePadAxisButtonRight
	GamePadAxisButtonDown
	GamePadAxisButtonLeft
	GamePadAxisButtonL1
	GamePadAxisButtonR1
	GamePadAxisButtonTriangle
	GamePadAxisButtonCircle
	GamePadAxisButtonCross
	GamePadAxisButtonSquare
	==============================	===================

.. cpp:function:: unsigned CUSBGamePadDevice::GetProperties (void)

	Returns the properties of the gamepad as a bit mask of ``TGamePadProperty`` constants, which are:

	======================================	============================================
	Constant				Description
	======================================	============================================
	GamePadPropertyIsKnown			is a known gamepad
	GamePadPropertyHasLED			supports ``SetLEDMode()``
	GamePadPropertyHasRGBLED		if set, ``GamePadPropertyHasLED`` is set too
	GamePadPropertyHasRumble		supports ``SetRumbleMode()``
	GamePadPropertyHasGyroscope		provides sensor info in ``TGamePadState``
	GamePadPropertyHasTouchpad		has touchpad with button
	GamePadPropertyHasAlternativeMapping	has additional +/- buttons, no START button
	======================================	============================================

.. cpp:function:: const TGamePadState *CUSBGamePadDevice::GetInitialState (void)

	Returns a pointer to the current gamepad state. This allows to initially request the information about the different gamepad controls. The control's state fields may have some default value, when a report from the gamepad has not been received yet. ``GetReport()`` is an deprecated alias for this method.

.. cpp:function:: void CUSBGamePadDevice::RegisterStatusHandler (TGamePadStatusHandler *pStatusHandler)

	Registers a handler function to be called on gamepad state changes. ``pStatusHandler`` is a pointer to this function, with this prototype:

.. c:type:: void TGamePadStatusHandler (unsigned nDeviceIndex, const TGamePadState *pGamePadState)

	``nDeviceIndex`` is the zero-based device index of this gamepad. The gamepad with the name ``"upadN"`` (N >= 1) in the device name service has the device index ``N-1``. ``pGamePadState`` is a pointer to the current gamepad state.

.. cpp:function:: boolean CUSBGamePadDevice::SetLEDMode (TGamePadLEDMode Mode)

	Sets LED(s) on gamepads with multiple uni-color LEDs. ``Mode`` selects the LED mode to be set. Returns ``TRUE`` if the LED mode is supported and was successfully set. A gamepad may support only a subset of the defined ``TGamePadLEDMode`` modes:

	* GamePadLEDModeOff
	* GamePadLEDModeOn1
	* GamePadLEDModeOn2
	* GamePadLEDModeOn3
	* GamePadLEDModeOn4
	* GamePadLEDModeOn5
	* GamePadLEDModeOn6
	* GamePadLEDModeOn7
	* GamePadLEDModeOn8
	* GamePadLEDModeOn9
	* GamePadLEDModeOn10


.. cpp:function:: boolean CUSBGamePadDevice::SetLEDMode (u32 nRGB, u8 uchTimeOn, u8 uchTimeOff)

	Sets the LED on gamepads with a single flash-able RGB-color LED. The property bit ``GamePadPropertyHasRGBLED`` is set, if this method is supported by a gamepad. ``nRGB`` is the color value to be set (0x00rrggbb). ``uchTimeOn`` is the duration, while the LED is on in 1/100 seconds. ``uchTimeOff`` is the duration, while the LED is off in 1/100 seconds. Returns ``TRUE``, if the operation was successful.

.. cpp:function:: boolean CUSBGamePadDevice::SetRumbleMode (TGamePadRumbleMode Mode)

	Sets the rumble mode ``Mode``, if the gamepad supports it (``GamePadPropertyHasRumble`` is set). Returns ``TRUE``, if the operation was successful. The following modes are defined:

	* GamePadRumbleModeOff
	* GamePadRumbleModeLow
	* GamePadRumbleModeHigh

CUSBSerialDevice
^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/usb/usbserial.h>

.. cpp:class:: CUSBSerialDevice : public CUSBFunction

	This class is the base class for USB serial device (aka interface, adapter) drivers and the generic application interface for USB serial devices. There are a number of different derived classes, which implement the drivers for specific devices. Circle automatically creates an instance of the appropriate class, when a compatible USB serial device is found in the USB device enumeration process. Therefore only the class methods, needed to use the USB serial device by an application, are described here, not the methods used for initialization. This device has the name ``"uttyN"`` (N >= 1) in the device name service.

.. note::

	Circle currently supports USB serial devices, which are compatible with the USB CDC-class specification (interfaces 2-2-0 and 2-2-1) and other devices, which use the following controllers: CH341, CP210x, FT231x, PL2303.

	There are many different combinations of USB vendor and device IDs for these devices and Circle supports only a small subset of these combinations, which were available for tests. If you have a USB serial device, which is not detected, there is still some chance, that the device can work with a Circle driver. You have to add the vendor/device ID combination of your device to the array ``DeviceIDTable[]`` at the end of the respective source file *lib/usb/usbserial\*.cpp* and test it. Please report newly found vendor/device ID combinations and the used driver!

.. cpp:function:: int CUSBSerialDevice::Read (void *pBuffer, size_t nCount)
.. cpp:function:: int CUSBSerialDevice::Write (const void *pBuffer, size_t nCount)

	Reads/writes data from/to the USB serial device (see :cpp:class:`CDevice`).

.. cpp:function:: boolean CUSBSerialDevice::SetBaudRate (unsigned nBaudRate)

	Sets the interface speed to a specific baud (bit) rate. ``nBaudRate`` is the rate in bits per second. Returns ``TRUE`` on success.

.. cpp:function:: boolean CUSBSerialDevice::SetLineProperties (TUSBSerialDataBits DataBits, TUSBSerialParity Parity, TUSBSerialStopBits StopBits)

	Sets the communication parameters number of data bits (``DataBits``), parity (``Parity``) and number of stop bits (``StopBits``) to the following values.  Returns ``TRUE`` on success.

.. code-block:: c

	enum TUSBSerialDataBits
	{
		USBSerialDataBits5 = 5,
		USBSerialDataBits6 = 6,
		USBSerialDataBits7 = 7,
		USBSerialDataBits8 = 8,
	};

	enum TUSBSerialStopBits
	{
		USBSerialStopBits1 = 1,
		USBSerialStopBits2 = 2,
	};

	enum TUSBSerialParity
	{
		USBSerialParityNone,
		USBSerialParityOdd,
		USBSerialParityEven,
	};

.. cpp:function:: unsigned CUSBSerialDevice::GetOptions (void) const

	Returns the current serial options mask.

.. cpp:function:: void CUSBSerialDevice::SetOptions (unsigned nOptions)

	Sets the serial options mask to ``nOptions``. See :cpp:func:`CSerialDevice::SetOptions()` for the supported options.

CUSBPrinterDevice
^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/usb/usbprinter.h>

.. cpp:class:: CUSBPrinterDevice : public CUSBFunction

	This class is a simple driver for printers with USB interface. Only printers are supported, which are by default able to print ASCII characters on their own, not GDI printers. There is only one method of interest for applications, which writes the characters out to the printer. The printer device has the name ``"uprnN"`` (N >= 1) in the device name service.

.. cpp:function:: int CUSBPrinterDevice::Write (const void *pBuffer, size_t nCount)

	See :cpp:func:`CDevice::Write()`.

CTouchScreenDevice
^^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/input/touchscreen.h>

.. cpp:class:: CTouchScreenDevice : public CDevice

	This class is the generic touchscreen interface device. An instance of this class is automatically created, when a compatible USB touchscreen is found in the USB device enumeration process. When the class :cpp:class:`CRPiTouchScreen` is manually instantiated, it is created too. This device has the name ``"touchN"`` (N >= 1) in the device name service.

.. cpp:function:: void CTouchScreenDevice::Setup (CDisplay *pDisplay)

	This method must be be called before any other method with ``pDisplay`` set to the display, which will receive the touch events.

.. cpp:function:: void CTouchScreenDevice::Update (void)

	This method must be called about 60 times per second. This is required for the Raspberry Pi official touchscreen only, but to be prepared for any touchscreen, you should call it in any case.

.. cpp:function:: void CTouchScreenDevice::RegisterEventHandler (TTouchScreenEventHandler *pEventHandler)

	Registers a handler function, which will be called on events from the touchscreen. The prototype of the handler is:

.. c:type:: void TTouchScreenEventHandler (TTouchScreenEvent Event, unsigned nID, unsigned nPosX, unsigned nPosY)

	``Event`` specifies the received event. ``nID`` is an zero based identifier of the finger (for multi-touch). This first finger has always the ID zero. ``nPosX`` and ``nPosY`` specify the pixel position of the finger on the screen (for finger down and move events), where ``(0,0)`` is the top left position. The following touchscreen events are defined:

.. c:enum:: TTouchScreenEvent

	* TouchScreenEventFingerDown
	* TouchScreenEventFingerUp
	* TouchScreenEventFingerMove

.. cpp:function:: boolean CTouchScreenDevice::SetCalibration (const unsigned Coords[4])

	Sets the calibration parameters for the touchscreen. ``Coords`` are the usable coordinates (min-x, max-x, min-y, max-y) of the touchscreen. Returns ``TRUE``, if the calibration information is valid.

.. note::

	The calibration parameters for a touchscreen can be determined with the `Touchscreen calibrator <https://github.com/rsta2/circle/tree/master/tools/touchscreen-calibrator>`_.

CRPiTouchScreen
^^^^^^^^^^^^^^^

.. note::

	This class does currently not work on the Raspberry Pi 5.

.. code-block:: cpp

	#include <circle/input/rpitouchscreen.h>

.. cpp:class:: CRPiTouchScreen

	This class is a driver for the official Raspberry Pi touchscreen. If you want to use this touchscreen, you have to create an instance of this class and initialize it. For the further use of this touchscreen an instance of the class :cpp:class:`CTouchScreenDevice` is automatically created.

.. c:macro:: RPITOUCH_SCREEN_MAX_POINTS

	The maximum number of detected fingers on the touchscreen (10).

.. cpp:function:: boolean CRPiTouchScreen::Initialize (void)

	Initializes the driver. Returns ``TRUE`` on success.

.. note::

	The driver cannot detect, if an official Raspberry Pi touchscreen is actually connected. Normally it returns ``TRUE`` in any case.

CConsole
^^^^^^^^

.. code-block:: cpp

	#include <circle/input/console.h>

.. cpp:class:: CConsole : public CDevice

	This class implements a console with input and output stream and a line editor using the screen ``tty1`` and USB keyboard ``ukbd1`` devices or alternate device(s) (e.g. serial interface). The console device itself has the name ``console`` in the device name service. The `sample/32-i2cshell <https://github.com/rsta2/circle/tree/master/sample/32-i2cshell>`_ demonstrates, how this class can be used to implement a simple shell.

.. note::

	This class does not create instances of the devices, which are used for input and output. This has to be done by the application. The device ``ukbd1`` is created in the USB device enumeration process, when an USB keyboard is found.

.. cpp:function:: CConsole::CConsole (CDevice *pAlternateDevice = 0, boolean bPlugAndPlay = FALSE)

	Creates an instance of this class. ``pAlternateDevice`` is an alternate device to be used, if the USB keyboard is not attached (default none). ``bPlugAndPlay`` must be set to ``TRUE`` to enable USB plug-and-play support for the console. This constructor is mandatory for USB plug-and-play operation.

.. cpp:function:: CConsole::CConsole (CDevice *pInputDevice, CDevice *pOutputDevice)

	Creates an instance of this class. ``pInputDevice`` is the device used for input (instead of the USB keyboard) and ``pOutputDevice`` is the device used for output (instead of the screen).

.. cpp:function:: boolean CConsole::Initialize (void)

	Initializes the console class. Returns ``TRUE``, if the operation has been successful.

.. cpp:function:: void CConsole::UpdatePlugAndPlay (void)

	Updates the USB plug-and-play configuration. This method must be called continuously, if the USB-plug-and-play support has been enabled in the constructor.

.. cpp:function:: boolean CConsole::IsAlternateDeviceUsed (void) const

	Returns ``TRUE``, if the alternate device is used instead of screen/USB keyboard?

.. cpp:function:: int CConsole::Read (void *pBuffer, size_t nCount)

	See :cpp:func:`CDevice::Read()`.  This method does not block! It has to be called until ``!= 0`` is returned.

.. cpp:function:: int CConsole::Write (const void *pBuffer, size_t nCount)

	See :cpp:func:`CDevice::Write()`.

.. cpp:function:: unsigned CConsole::GetOptions (void) const

	Returns the console options bit mask.

.. cpp:function:: void CConsole::SetOptions (unsigned nOptions)

	Sets the console options bit mask to ``nOptions``.

	The following bits are defined:

.. c:macro:: CONSOLE_OPTION_ICANON

	Canonic input using a line editor is enabled (default).

.. c:macro:: CONSOLE_OPTION_ECHO

	Echo input to output is enabled (default).

CNullDevice
^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/nulldevice.h>

.. cpp:class:: CNullDevice : public CDevice

	This class implements the null device, which accepts all written characters and returns 0 (EOF) on read. It can be used instead of other character device classes, for instance as target for the :ref:`System log`. This device has the name ``"null"`` in the device name service.

.. cpp:function:: int CNullDevice::Read (void *pBuffer, size_t nCount)

	Returns always 0 (EOF).

.. cpp:function:: int CNullDevice::Write (const void *pBuffer, size_t nCount)

	Returns the number of written bytes, but ignores them.
