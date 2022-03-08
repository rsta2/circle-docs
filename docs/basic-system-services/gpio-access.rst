GPIO access
~~~~~~~~~~~

This section presents the Circle classes, which implement digital input/output operations, using the pins exposed on the 40-pin GPIO (General Purpose Input/Output) header of the Raspberry Pi (26-pin on older models). This covers writing and reading a single or all exposed GPIO pin(s), providing GPIO clocks for different purposes (e.g. for Pulse Width Modulation (PWM) output), triggering interrupts from GPIO pins, using I2C and SPI interfaces, and switching the (green) Act LED.

Please see these documents for a description of the GPIO hardware:

* `BCM2835 ARM Peripherals <https://datasheets.raspberrypi.com/bcm2835/bcm2835-peripherals.pdf>`_ (for Raspberry Pi 1 and Zero)
* `BCM2711 ARM Peripherals <https://datasheets.raspberrypi.com/bcm2711/bcm2711-peripherals.pdf>`_ (for Raspberry Pi 4)

The first document is also valid for the Raspberry Pi 2, 3 and Zero 2 with some modifications (e.g. I/O base address).

.. important::

	The Circle documentation (including all READMEs) uses SoC (BCM) numbers (0-53), when referring to specific GPIO pins. These numbers are different from the physical pin position numbers (1-40) on the GPIO header. Please see `pinout.xyz <https://pinout.xyz>`_ for the mapping of SoC numbers to the header position.

CGPIOPin
^^^^^^^^

This class encapsulates a GPIO pin, which can be read, write or inverted. A GPIO pin can trigger an interrupt, when a specific GPIO event occurs.

.. code-block:: c++

	#include <circle/gpiopin.h>

.. cpp:class:: CGPIOPin

.. c:macro:: GPIO_PINS

	Number of available GPIO pins (54).

Initialization
""""""""""""""

.. cpp:function:: CGPIOPin::CGPIOPin (void)

	Default constructor. Object must be initialized afterwards using ``AssignPin()``, ``SetMode()`` and optionally ``SetPullMode()``.

.. cpp:function:: CGPIOPin::CGPIOPin (unsigned nPin, TGPIOMode Mode, CGPIOManager *pManager = 0)

	Creates and initializes a ``CGPIOPin`` instance for GPIO pin number ``nPin``, set pin mode ``Mode``. ``pManager`` must be specified only, if this pin will trigger interrupts (IRQ). ``nPin`` can have a numeric value (0-53) or these special values:

	* GPIOPinAudioLeft (GPIO pin, which gates the left PWM audio channel)
	* GPIOPinAudioRight (GPIO pin, which gates the right PWM audio channel)

	``Mode`` can have these values:

	* GPIOModeInput
	* GPIOModeOutput
	* GPIOModeInputPullUp
	* GPIOModeInputPullDown
	* GPIOModeAlternateFunction0
	* GPIOModeAlternateFunction1
	* GPIOModeAlternateFunction2
	* GPIOModeAlternateFunction3
	* GPIOModeAlternateFunction4
	* GPIOModeAlternateFunction5

.. cpp:function:: void CGPIOPin::AssignPin (unsigned nPin)

	Assigns a GPIO pin number to the object. To be used together with the default constructor and ``SetMode()``. See ``CGPIOPin::CGPIOPin()`` for the possible values for ``nPin``.

.. cpp:function:: void CGPIOPin::SetMode (TGPIOMode Mode, boolean bInitPin = TRUE)

	Sets GPIO pin to ``Mode``. See ``CGPIOPin::CGPIOPin()`` for the possible values. If ``bInitPin`` is ``TRUE``, this method initializes the pull-up/down mode and output level (LOW) too. To be used together with the default constructor and ``AssignPin()`` or for dynamic changes of the direction for input/output pins.

.. cpp:function:: void CGPIOPin::SetPullMode (TGPIOPullMode Mode)

	Sets the pull-up/down mode to one of the following values:

	* GPIOPullModeOff
	* GPIOPullModeDown
	* GPIOPullModeUp

Input / Output
""""""""""""""

.. cpp:function:: void CGPIOPin::Write (unsigned nValue)

	Sets the GPIO pin to ``nValue`` (output), which can be ``LOW`` (0) or ``HIGH`` (1).

.. cpp:function:: unsigned CGPIOPin::Read (void) const

	Returns the value, read from the GPIO pin (input). Can be ``LOW`` (0) or ``HIGH`` (1).

.. cpp:function:: void CGPIOPin::Invert (void)

	Sets the GPIO pin to the inverted value. For output pins only.

.. cpp:function:: static void CGPIOPin::WriteAll (u32 nValue, u32 nMask)

	Sets the GPIO pins 0-31 at once. ``nValue`` specifies the levels of GPIO pins 0-31 in the respective bits to be written, where ``nMask`` is a bit mask for the written value. Only those GPIO pins are affected, for which the respective bit is set in ``nMask``. The other pins are not touched.

.. cpp:function:: static u32 CGPIOPin::ReadAll (void)

	Returns the level of the GPIO pins 0-31 in the respective bits.

Interrupts
""""""""""

A GPIO pin can trigger an interrupt (IRQ) under certain conditions. The ``CGPIOPin`` object must be initialized with a pointer to an instance of the class ``CGPIOManager`` for this purpose. There is maximal one instance of ``CGPIOManager`` in the system.

.. cpp:function:: void CGPIOPin::ConnectInterrupt (TGPIOInterruptHandler *pHandler, void *pParam, boolean bAutoAck = TRUE)

	Connects the interrupt handler function ``pHandler`` to the GPIO pin, to be called on a GPIO event. ``pParam`` is a user parameter, which will be handed over to the interrupt handler. If ``bAutoAck`` is TRUE, the GPIO event detect status will be automatically acknowledged, when the interrupt occurs. Otherwise, the interrupt handler must call ``AcknowledgeInterrupt()``. The GPIO interrupt handler has the following prototype:

.. code-block:: c

	void TGPIOInterruptHandler (void *pParam);

.. cpp:function:: void CGPIOPin::DisconnectInterrupt (void)

	Disconnects the interrupt handler from the GPIO pin. The interrupt source(s) must be disabled before using ``DisableInterrupt()`` and ``DisableInterrupt2()``, if they were enabled before.

.. cpp:function:: void CGPIOPin::EnableInterrupt (TGPIOInterrupt Interrupt)

	Enables a specific event condition to trigger an interrupt for this GPIO pin. ``Interrupt`` can be:

	* GPIOInterruptOnRisingEdge
	* GPIOInterruptOnFallingEdge
	* GPIOInterruptOnHighLevel
	* GPIOInterruptOnLowLevel
	* GPIOInterruptOnAsyncRisingEdge
	* GPIOInterruptOnAsyncFallingEdge

.. cpp:function:: void CGPIOPin::DisableInterrupt (void)

	Disables a previously enabled event condition from triggering an interrupt.

.. cpp:function:: void CGPIOPin::EnableInterrupt2 (TGPIOInterrupt Interrupt)

	Same function as ``EnableInterrupt()`` for a second interrupt source.

.. cpp:function:: void CGPIOPin::DisableInterrupt2 (void)

	Same function as ``DisableInterrupt()`` for a second interrupt source.

.. cpp:function:: void CGPIOPin::AcknowledgeInterrupt (void)

	Manually acknowledges the GPIO event detect status. To be called from the from interrupt handler, if ``bAutoAck`` was ``FALSE``, when calling ``ConnectInterrupt()``.

CGPIOPinFIQ
^^^^^^^^^^^

This class encapsulates a special GPIO pin, which is using the FIQ (Fast Interrupt Request) to handle GPIO interrupts with low latency. There is only one GPIO pin of this type allowed in the system.

.. code-block:: c++

	#include <circle/gpiopinfiq.h>

.. cpp:class:: CGPIOPinFIQ : public CGPIOPin

``CGPIOPinFIQ`` is derived from ``CGPIOPin`` and inherits its methods. For initialization it provides this special constructor:

.. cpp:function:: CGPIOPinFIQ::CGPIOPinFIQ (unsigned nPin, TGPIOMode Mode, CInterruptSystem *pInterrupt)

	The parameters are the same as for ``CGPIOPin::CGPIOPin()``, with one exception: ``pInterrupt`` is a pointer the single interrupt system object in the system. A ``CGPIOPinFIQ`` object does not need an instance of ``CGPIOManager`` to generate interrupts.

CGPIOManager
^^^^^^^^^^^^

This class implements an interrupt multiplexer for ``CGPIOPin`` instances. There must be exactly one instance of ``CGPIOManager`` in the system, if at least one GPIO pin triggers interrupts using the IRQ.

.. code-block:: c++

	#include <circle/gpiomanager.h>

.. cpp:class:: CGPIOManager

.. cpp:function:: CGPIOManager::CGPIOManager (CInterruptSystem *pInterrupt)

	Creates a ``CGPIOManager`` instance. ``pInterrupt`` is a pointer to the interrupt system object.

.. cpp:function:: boolean CGPIOManager::Initialize (void)

	Initializes the ``CGPIOManager`` object. Usually called from ``CKernel::Initialize()``. Returns ``TRUE``, if the initialization was successful.

CGPIOClock
^^^^^^^^^^

A GPIO clock is a programmable digital clock generator. A Raspberry Pi computer provides several of these clocks. Their output is used for special system purposes (e.g. for the PWM and PCM / I2S devices) or can be directly connected to some GPIO pins. GPIO clocks are driven by an internal clock source with a specific clock frequency.

.. code-block:: c++

	#include <circle/gpioclock.h>

.. cpp:class:: CGPIOClock

.. cpp:function:: CGPIOClock::CGPIOClock (TGPIOClock Clock, TGPIOClockSource Source = GPIOClockSourceUnknown)

	Creates a ``CGPIOClock`` instance for GPIO clock ``Clock`` with clock source ``Source``. ``Clock`` can be:

	==============	==================================================
	Clock		Connected to
	==============	==================================================
	GPIOClock0	GPIO4 (ALT0) or GPIO20 (ALT5)
	GPIOClock1	GPIO5 (ALT0) or GPIO21 (ALT5), Raspberry Pi 4 only
	GPIOClock2	GPIO6 (ALT0)
	GPIOClockPCM	PCM / I2S device
	GPIOClockPWM	PWM device
	==============	==================================================

	The respective GPIO pin has to be set to the given ``GPIOModeAlternateFunctionN`` (ALTn), using a ``CGPIOPin`` object, so that the signal can be accessed at the GPIO header. ``Source`` can be:

	==============================	======================	======================
	Source				Raspberry Pi 1-3	Raspberry Pi 4
	==============================	======================	======================
	GPIOClockSourceOscillator	19.2 MHz		54 MHz
	GPIOClockSourcePLLC		1000 MHz (varies)	1000 MHz (may vary)
	GPIOClockSourcePLLD		500 MHz			750 MHz
	GPIOClockSourceHDMI		216 MHz			unused
	==============================	======================	======================

	If ``Source`` is set to ``GPIOClockSourceUnknown``, the clock source is selected automatically, when ``StartRate()`` is called.

.. cpp:function:: void CGPIOClock::Start (unsigned nDivI, unsigned nDivF = 0, unsigned nMASH = 0)

	Starts the clock using the given integer divider ``nDivI`` (1-4095). The MASH modes with ``nDivF > 0`` are described in the `BCM2835 ARM Peripherals`_ document.

.. cpp:function:: boolean CGPIOClock::StartRate (unsigned nRateHZ)

	Starts the clock with the given target frequency ``nRateHZ`` in Hertz. Assigns the clock source automatically. Returns ``FALSE``, if the requested rate cannot be generated.

.. cpp:function:: void CGPIOClock::Stop (void)

	Stops the clock.

CPWMOutput
^^^^^^^^^^

This class provides access to the Pulse Width Modulator (PWM) device, which can be used to generate (pseudo) analog signals on the GPIO pins 18 and 19 (two channels). These pins have to be set to ``GPIOModeAlternateFunction5`` using the class ``CGPIOPin`` for that purpose.

.. code-block:: c++

	#include <circle/pwmoutput.h>

.. cpp:class:: CPWMOutput

.. cpp:function:: CPWMOutput::CPWMOutput (TGPIOClockSource Source, unsigned nDivider, unsigned nRange, boolean bMSMode)

	Creates a ``CPWMOutput`` object with clock source ``Source`` and the divider ``nDivider`` (equivalent to ``nDivI``, 1-4095). See ``CGPIOClock`` for these parameters. For the parameters ``nRange`` (Range) and ``bMSMode`` (M/S mode) see the `BCM2835 ARM Peripherals`_ document.

.. cpp:function:: void CPWMOutput::Start (void)

	Starts the PWM clock and device.

.. cpp:function:: void CPWMOutput::Stop (void)

	Stops the PWM clock and device.

.. cpp:function:: void CPWMOutput::Write (unsigned nChannel, unsigned nValue)

	Write ``nValue`` (0-Range) to PWM channel ``nChannel`` (1 or 2).

.. c:macro:: PWM_CHANNEL1
.. c:macro:: PWM_CHANNEL2

	Macros to be used for the ``nChannel`` parameter.

CI2CMaster
^^^^^^^^^^

This class is a driver for the I2C master devices of the Raspberry Pi computer. The GPIO pin mapping for the I2C master devices is as follows:

=======	=======================	=======================	===================
nDevice	nConfig 0 (SDA SCL)	nConfig 1 (SDA SCL)	Raspberry Pi boards
=======	=======================	=======================	===================
0	GPIO0	GPIO1					Rev. 1
1	GPIO2	GPIO3					All other
2							None
3	GPIO2	GPIO3		GPIO4	GPIO5		Raspberry Pi 4 only
4	GPIO6	GPIO7		GPIO8	GPIO9		Raspberry Pi 4 only
5	GPIO10	GPIO11		GPIO12	GPIO13		Raspberry Pi 4 only
6	GPIO22	GPIO23					Raspberry Pi 4 only
=======	=======================	=======================	===================

The ``Read()`` and ``Write()`` methods (see below) may return the following error codes as a negative value:

======================	=====================================
Value			Description
======================	=====================================
I2C_MASTER_INALID_PARM	Invalid parameter
I2C_MASTER_ERROR_NACK	Received a NACK
I2C_MASTER_ERROR_CLKT	Received clock stretch timeout
I2C_MASTER_DATA_LEFT	Not all data has been sent / received
======================	=====================================

.. code-block:: c++

	#include <circle/i2cmaster.h>

.. cpp:class:: CI2CMaster

.. cpp:function:: CI2CMaster::CI2CMaster (unsigned nDevice, boolean bFastMode = FALSE, unsigned nConfig = 0)

	Creates a ``CI2CMaster`` object for I2C master ``nDevice`` (0-6), with configuration ``nConfig`` (0 or 1). See the mapping above for these parameters. The default I2C clock is 100 KHz or 400 KHz, if ``bFastMode`` is ``TRUE``. This can be modified with ``SetClock()`` for a specific transfer.

.. cpp:function:: boolean CI2CMaster::Initialize (void)

	Initializes the ``CI2CMaster`` object. Usually called from ``CKernel::Initialize()``. Returns ``TRUE``, if the initialization was successful.

.. cpp:function:: void CI2CMaster::SetClock (unsigned nClockSpeed)

	Modifies the default clock before a specific transfer. ``nClockSpeed`` is the wanted I2C clock frequency in Hertz.

.. cpp:function:: int CI2CMaster::Read (u8 ucAddress, void *pBuffer, unsigned nCount)

	Reads ``nCount`` bytes from the I2C slave device with address ``ucAddress`` into ``pBuffer``. Returns the number of read bytes or < 0 on failure. See the error codes above.

.. cpp:function:: int CI2CMaster::Write (u8 ucAddress, const void *pBuffer, unsigned nCount)

	Writes ``nCount`` bytes to the I2C slave device with address ``ucAddress`` from ``pBuffer``. Returns the number of written bytes or < 0 on failure. See the error codes above.

CI2CSlave
^^^^^^^^^

This class is a driver for the I2C slave device. The GPIO pin mapping is as follows:

==============	======	======
Raspberry Pi	SDA	SCL
==============	======	======
1-3, Zero	GPIO18	GPIO19
4		GPIO10	GPIO11
==============	======	======

.. code-block:: c++

	#include <circle/i2cslave.h>

.. cpp:class:: CI2CSlave

.. cpp:function:: CI2CSlave::CI2CSlave (u8 ucAddress)

	Creates the ``CI2CSlave`` object and assigns the I2C address ``ucAddress``.

.. cpp:function:: boolean CI2CSlave::Initialize (void)

	Initializes the ``CI2CSlave`` object. Usually called from ``CKernel::Initialize()``. Returns ``TRUE``, if the initialization was successful.

.. cpp:function:: int CI2CSlave::Read (void *pBuffer, unsigned nCount)

	Reads ``nCount`` bytes from the I2C master into ``pBuffer``. Returns the number of read bytes or < 0 on failure.

.. cpp:function:: int CI2CSlave::Write (const void *pBuffer, unsigned nCount)

	Writes ``nCount`` bytes to the I2C master from ``pBuffer``. Returns the number of written bytes or < 0 on failure.

CSPIMaster
^^^^^^^^^^

The class ``CSPIMaster`` is a driver for SPI master devices, with these features:

* SPI non-AUX devices only
* Standard mode (3-wire) only
* Chip select lines (CE0, CE1) are active low
* Polled operation only

The GPIO pin mapping is as follows:

=======	=======	=======	=======	=======	=======	===================
nDevice	MISO	MOSI	SCLK	CE0	CE1	Support
=======	=======	=======	=======	=======	=======	===================
0	GPIO9	GPIO10	GPIO11	GPIO8	GPIO7 	All boards
1						class CSPIMasterAUX
2						None
3	GPIO1	GPIO2	GPIO3	GPIO0	GPIO24	Raspberry Pi 4 only
4	GPIO5	GPIO6	GPIO7	GPIO4	GPIO25	Raspberry Pi 4 only
5	GPIO13	GPIO14	GPIO15	GPIO12	GPIO26	Raspberry Pi 4 only
6	GPIO19	GPIO20	GPIO21	GPIO18	GPIO27	Raspberry Pi 4 only
=======	=======	=======	=======	=======	=======	===================

GPIO0 and GPIO1 are normally reserved for the ID EEPROM of hat boards.

.. code-block:: c++

	#include <circle/spimaster.h>

.. cpp:class:: CSPIMaster

.. cpp:function:: CSPIMaster::CSPIMaster (unsigned nClockSpeed = 500000, unsigned CPOL = 0, unsigned CPHA = 0, unsigned nDevice = 0)

	Creates an ``CSPIMaster`` instance for access to SPI master ``nDevice`` (see table above), with default SPI clock frequency ``nClockSpeed`` in Hertz, clock polarity ``CPOL`` (0 or 1) and clock phase ``CPHA`` (0 or 1).

.. cpp:function:: boolean CSPIMaster::Initialize (void)

	Initializes the SPI master. Usually called from ``CKernel::Initialize()``. Returns ``TRUE``, if initialization was successful.

.. cpp:function:: void CSPIMaster::SetClock (unsigned nClockSpeed)

	Modifies the default SPI clock frequency before a specific transfer. ``nClockSpeed`` is the SPI clock frequency in Hertz. This method is not protected by an internal spin lock for multi-core operation.

.. cpp:function:: void CSPIMaster::SetMode (unsigned CPOL, unsigned CPHA)

	Modifies the default clock polarity / phase before a specific transfer. ``CPOL`` is the clock polarity (0 or 1) and ``CPHA`` is the clock phase (0 or 1). These parameters must match the SPI slave settings. This method is not protected by an internal spin lock for multi-core operation.

.. cpp:function:: void CSPIMaster::SetCSHoldTime (unsigned nMicroSeconds)

	Sets the additional time, CE# stays active after the transfer. The set value is valid for the next transfer only. Normally CE# goes inactive very soon after the transfer, this sets the additional time, CE# stays active.

.. cpp:function:: int CSPIMaster::Read (unsigned nChipSelect, void *pBuffer, unsigned nCount)

	Reads ``nCount`` bytes into ``pBuffer``. Activates chip select ``nChipSelect`` (CE#, 0, 1 or ``ChipSelectNone``). Returns the number of read bytes or < 0 on failure.

.. cpp:function:: int CSPIMaster::Write (unsigned nChipSelect, const void *pBuffer, unsigned nCount)

	Writes ``nCount`` bytes from ``pBuffer``. Activates chip select ``nChipSelect`` (CE#, 0, 1 or ``ChipSelectNone``). Returns the number of written bytes or < 0 on failure.

.. cpp:function:: int CSPIMaster::WriteRead (unsigned nChipSelect, const void *pWriteBuffer, void *pReadBuffer, unsigned nCount)

	Simultaneous writes and reads ``nCount`` bytes from ``pWriteBuffer`` and to ``pReadBuffer``. Activates chip select ``nChipSelect`` (CE#, 0, 1 or ``ChipSelectNone``). Returns the number of transferred bytes or < 0 on failure.

CSPIMasterAUX
^^^^^^^^^^^^^

The class ``CSPIMasterAUX`` is a polling driver for the auxiliary SPI master (SPI1). The GPIO pin mapping is as follows:

======	======	======	======	======	======
MISO	MOSI	SCLK	CE0	CE1	CE2
======	======	======	======	======	======
GPIO19	GPIO20	GPIO21	GPIO18	GPIO17	GPIO16
======	======	======	======	======	======

The CE# signals are active low.

.. code-block:: c++

	#include <circle/spimasteraux.h>

.. cpp:class:: CSPIMasterAUX

.. cpp:function:: CSPIMasterAUX::CSPIMasterAUX (unsigned nClockSpeed = 500000)

	Creates a ``CSPIMasterAUX`` object. Sets the default SPI clock frequency to ``nClockSpeed`` in Hertz.

.. cpp:function:: boolean CSPIMasterAUX::Initialize (void)

	Initializes the SPI1 AUX master. Usually called from ``CKernel::Initialize()``. Returns ``TRUE``, if initialization was successful.

.. cpp:function:: void CSPIMasterAUX::SetClock (unsigned nClockSpeed)

	Modifies the default SPI clock frequency before a specific transfer. ``nClockSpeed`` is the SPI clock frequency in Hertz. This method is not protected by an internal spin lock for multi-core operation.

.. cpp:function:: int CSPIMasterAUX::Read (unsigned nChipSelect, void *pBuffer, unsigned nCount)

	Reads ``nCount`` bytes into ``pBuffer``. Activates chip select ``nChipSelect`` (CE#, 0, 1 or 2). Returns the number of read bytes or < 0 on failure.

.. cpp:function:: int CSPIMasterAUX::Write (unsigned nChipSelect, const void *pBuffer, unsigned nCount)

	Writes ``nCount`` bytes from ``pBuffer``. Activates chip select ``nChipSelect`` (CE#, 0, 1 or 2). Returns the number of written bytes or < 0 on failure.

.. cpp:function:: int CSPIMasterAUX::WriteRead (unsigned nChipSelect, const void *pWriteBuffer, void *pReadBuffer, unsigned nCount)

	Simultaneous writes and reads ``nCount`` bytes from ``pWriteBuffer`` and to ``pReadBuffer``. Activates chip select ``nChipSelect`` (CE#, 0, 1 or 2). Returns the number of transferred bytes or < 0 on failure.

CSPIMasterDMA
^^^^^^^^^^^^^

The class ``CSPIMasterDMA`` is a driver for the SPI0 master device. It implements an asynchronous DMA operation. Optionally one can do synchronous polling transfers (e.g. for small amounts of data). The GPIO pin mapping of the SPI0 master device is as follows:

======	======	======	======	======
MISO	MOSI	SCLK	CE0	CE1
======	======	======	======	======
GPIO9	GPIO10	GPIO11	GPIO8	GPIO7
======	======	======	======	======

.. code-block:: c++

	#include <circle/spimasterdma.h>

.. cpp:class:: CSPIMasterDMA

.. cpp:function:: CSPIMasterDMA::CSPIMasterDMA (CInterruptSystem *pInterruptSystem, unsigned nClockSpeed = 500000, unsigned CPOL = 0, unsigned CPHA = 0, boolean bDMAChannelLite = TRUE)

	Creates a ``CSPIMasterDMA`` object. Sets the default SPI clock frequency to ``nClockSpeed`` in Hertz, the clock polarity to ``CPOL`` (0 or 1) and the clock phase to ``CPHA`` (0 or 1). ``pInterruptSystem`` is a pointer to the interrupt system object. Set ``bDMAChannelLite`` to ``FALSE`` for very high speeds or transfer sizes >= 64K.

.. cpp:function:: boolean CSPIMasterDMA::Initialize (void)

	Initializes the SPI0 master. Usually called from ``CKernel::Initialize()``. Returns ``TRUE``, if initialization was successful.

.. cpp:function:: void CSPIMasterDMA::SetClock (unsigned nClockSpeed)

	Modifies the default SPI clock frequency before a specific transfer. ``nClockSpeed`` is the SPI clock frequency in Hertz.

.. cpp:function:: void CSPIMasterDMA::SetMode (unsigned CPOL, unsigned CPHA)

	Modifies the default clock polarity / phase before a specific transfer. ``CPOL`` is the clock polarity (0 or 1) and ``CPHA`` is the clock phase (0 or 1). These parameters must match the SPI slave settings.

.. cpp:function:: void CSPIMasterDMA::SetCompletionRoutine (TSPICompletionRoutine *pRoutine, void *pParam)

	Sets a completion routine ``pRoutine`` to be called, when the next transfer completes. ``pParam`` is a user parameter, which is handed over to the completion routine. The prototype of the completion routine looks like this:

.. code-block:: c

	void TSPICompletionRoutine (boolean bStatus, void *pParam);

``bStatus`` is ``TRUE`` on success.

.. cpp:function:: void CSPIMasterDMA::StartWriteRead (unsigned nChipSelect, const void *pWriteBuffer, void *pReadBuffer, unsigned nCount)

	Starts a simultaneous write and read transfer of ``nCount`` bytes from ``pWriteBuffer`` and to ``pReadBuffer``. Chip select ``nChipSelect`` (CE#, 0, 1 or ``ChipSelectNone``) will be activated during the transfer. The buffers must be aligned to the size of a data-cache-line (see :ref:`dma-buffers`).

.. cpp:function:: int CSPIMasterDMA::WriteReadSync (unsigned nChipSelect, const void *pWriteBuffer, void *pReadBuffer, unsigned nCount)

	Simultaneous writes and reads ``nCount`` bytes from ``pWriteBuffer`` and to ``pReadBuffer``. Activates chip select ``nChipSelect`` (CE#, 0, 1 or ``ChipSelectNone``). Returns the number of transferred bytes or < 0 on failure. Synchronous (polled) operation for small amounts of data.

CActLED
^^^^^^^

This class switches the (green) Act(ivity) LED on or off. It automatically determines the Raspberry Pi model to use the right LED pin for the model.

.. code-block:: c++

	#include <circle/actled.h>

.. cpp:class:: CActLED

.. cpp:function:: CActLED::CActLED (boolean bSafeMode = FALSE)

	Creates the ``CActLED`` object. Safe mode works with LEDs connected to GPIO expander and chain boot, but is not as quick.

.. cpp:function:: void CActLED::On (void)

	Switches the Act LED on.

.. cpp:function:: void CActLED::Off (void)

	Switches the Act LED off.

.. cpp:function:: void CActLED::Blink (unsigned nCount, unsigned nTimeOnMs = 200, unsigned nTimeOffMs = 500)

	Blinks the Act LED ``nCount`` times. The LED is ``nTimeOnMs`` milliseconds on and ``nTimeOffMs`` milliseconds off.

.. cpp:function:: static CActLED *CActLED::Get (void)

	Returns a pointer to the single ``CActLED`` instance in the system (if any).
