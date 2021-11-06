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
	==============	======================================	=====================

	^X = Control character, \\E = Escape (\\x1b), %d = Numerical parameter (ASCII)

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

	This device has the name ``"ttySN"`` (N >= 1) in the device name service, where ``N = nDevice+1``.

.. note::

	This driver can be used in two modes: polling or interrupt driven. The mode is selected with the parameter ``pInterruptSystem`` of the constructor.

.. c:macro:: SERIAL_BUF_SIZE

	This macro defines the size of the read and write ring buffers for the interrupt driver (default 2048). If you want to increase the buffer size, you have to specify a value, which is a power of two.

.. cpp:function:: CSerialDevice::CSerialDevice (CInterruptSystem *pInterruptSystem = 0, boolean bUseFIQ = FALSE, unsigned nDevice = 0)

	Constructs a ``CSerialDevice`` object. Multiple instances are possible on the Raspberry Pi 4. ``nDevice`` selects the used serial device (see the table above). ``pInterruptSystem`` is a pointer to interrupt system object, or 0 to use the polling driver. The interrupt driver uses the IRQ by default. Set ``bUseFIQ`` to ``TRUE`` to use the FIQ instead. This is recommended for higher baud rates.

.. cpp:function:: boolean CSerialDevice::Initialize (unsigned nBaudrate = 115200, unsigned nDataBits = 8, unsigned nStopBits = 1, TParity Parity = ParityNone)

	Initializes the serial device and sets the baud rate to ``nBaudrate`` bits per second. ``nDataBits`` selects the number of data bits (5..8, default 8) and ``nStopBits`` the number of stop bits (1..2, default 1). ``Parity`` can be ``CSerialDevice::ParityNone`` (default), ``CSerialDevice::ParityOdd`` or ``CSerialDevice::ParityEven``. Returns ``TRUE`` on success.

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

.. cpp:function:: void CSerialDevice::RegisterMagicReceivedHandler (const char *pMagic, TMagicReceivedHandler *pHandler)

	Registers a magic received handler ``pHandler``, which is called, when the string ``pMagic`` is found in the received data. ``pMagic`` must remain valid after return from this method. This method does only work with interrupt driver.

.. cpp:type:: void CSerialDevice::TMagicReceivedHandler (void)

CUSBKeyboardDevice
""""""""""""""""""

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

CMouseDevice
""""""""""""

.. code-block:: cpp

	#include <circle/input/mouse.h>

.. cpp:class:: CMouseDevice : public CDevice

	This class is the generic mouse interface device. An instance of this class is automatically created, when a compatible USB mouse or USB gamepad with touchpad is found in the USB device enumeration process. Therefore only the class methods, needed to use the mouse by an application, are described here, not the method used for creation. This device has the name ``"mouseN"`` (N >= 1) in the device name service.

.. note::

	This class supports two mouse modes: cooked and raw mode. In cooked mode a mouse cursor is shown on the screen and automatically controlled by the driver, which reports several mouse events (down, up, move, wheel).

	In raw mode the driver directly reports the raw mouse displacement, button and wheel information.

.. cpp:function:: boolean CMouseDevice::Setup (unsigned nScreenWidth, unsigned nScreenHeight)

	Setup mouse device in cooked mode. ``nScreenWidth`` and ``nScreenHeight`` are the width and height of the screen in pixels. Returns ``FALSE`` on failure. This method must be called first in the setup process for a mouse in cooked mode.

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

.. cpp:function:: unsigned CMouseDevice::GetButtonCount (void) const

	Returns the number of supported buttons for this mouse device.

.. cpp:function:: boolean CMouseDevice::HasWheel (void) const

	Returns ``TRUE``, if the mouse supports a mouse wheel.

CUSBGamePadDevice
"""""""""""""""""

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

.. CUSBSerialDevice
.. CUSBPrinterDevice
.. CTouchScreenDevice
.. CRPiTouchScreen
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
