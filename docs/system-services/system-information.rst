System information
~~~~~~~~~~~~~~~~~~

This section describes the classes ``CMachineInfo`` and ``CKernelOptions``, which provide information about the Raspberry Pi model, on which the application is running, and the runtime options, which can be defined in the file *cmdline.txt* on the SD card.

CMachineInfo
^^^^^^^^^^^^

Normally there is exactly one instance of the class ``CMachineInfo`` in the system, which is created by the Circle system initialization code. If another instance is created, it acts as an alias for the first instance.

.. code-block:: c++

	#include <circle/machineinfo.h>

.. cpp:class:: CMachineInfo

.. cpp:function:: static CMachineInfo *CMachineInfo::Get (void)

	Returns a pointer to the first instance of ``CMachineInfo``.

Model information
"""""""""""""""""

.. cpp:function:: TMachineModel CMachineInfo::GetMachineModel (void) const

	Returns the Raspberry Pi model, the application is running on. Possible values are:

* MachineModelA
* MachineModelBRelease1MB256
* MachineModelBRelease2MB256
* MachineModelBRelease2MB512
* MachineModelAPlus
* MachineModelBPlus
* MachineModelZero
* MachineModelZeroW
* MachineModel2B
* MachineModel3B
* MachineModel3APlus
* MachineModel3BPlus
* MachineModelCM
* MachineModelCM3
* MachineModelCM3Plus
* MachineModel4B
* MachineModel400
* MachineModelCM4
* MachineModelUnknown

.. cpp:function:: const char *CMachineInfo::GetMachineName (void) const

	Returns the name of the Raspberry Pi model, the application is running on.

.. cpp:function:: unsigned CMachineInfo::GetModelMajor (void) const

	Returns the major version (1-4) of the Raspberry Pi model, the application is running on, or zero if it is unknown.

.. cpp:function:: unsigned CMachineInfo::GetModelRevision (void) const

	Returns the revision number (1-) of the Raspberry Pi model, the application is running on, or zero if it is unknown.

.. cpp:function:: TSoCType CMachineInfo::GetSoCType (void) const

	Returns the type of the SoC (System on a Chip), the application is running on. Possible values are:

* SoCTypeBCM2835
* SoCTypeBCM2836
* SoCTypeBCM2837
* SoCTypeBCM2711
* SoCTypeUnknown

.. cpp:function:: unsigned CMachineInfo::GetRAMSize (void) const

	Returns the size of the SDRAM in MBytes of the Raspberry Pi model, the application is running on, or zero if it is unknown.

.. cpp:function:: const char *CMachineInfo::GetSoCName (void) const

	Returns the name of the SoC (System on a Chip), the application is running on.

.. cpp:function:: u32 CMachineInfo::GetRevisionRaw (void) const

	Returns the raw `revision code <https://www.raspberrypi.org/documentation/hardware/raspberrypi/revision-codes/README.md>`_ of the Raspberry Pi model, the application is running on.

Clocks and peripherals
""""""""""""""""""""""

.. cpp:function:: unsigned CMachineInfo::GetActLEDInfo (void) const

	Returns the information, about how the green Activity LED is connected to the system. The result has to be masked with ``ACTLED_PIN_MASK`` to extract the GPIO pin number. If the result masked with ``ACTLED_ACTIVE_LOW`` is not zero, the LED is on, when the value 0 is written to the GPIO pin. If the result masked with ``ACTLED_VIRTUAL_PIN`` is not zero, the LED is connected to a GPIO expander, which is controlled by the firmware.

.. cpp:function:: unsigned CMachineInfo::GetClockRate (u32 nClockId) const

	Returns the current frequency in Hz of the system clock, selected by ``nClockId``, which can have the following values:

* CLOCK_ID_CORE
* CLOCK_ID_ARM
* CLOCK_ID_UART
* CLOCK_ID_EMMC
* CLOCK_ID_EMMC2

.. cpp:function:: unsigned CMachineInfo::GetGPIOPin (TGPIOVirtualPin Pin) const

	Returns the physical GPIO pin number of the PWM audio pins. ``Pin`` can have the values ``GPIOPinAudioLeft`` or ``GPIOPinAudioRight``.

.. cpp:function:: unsigned CMachineInfo::GetGPIOClockSourceRate (unsigned nSourceId)

	This method allows to enumerate the different clock sources for GPIO clocks. It returns the frequency in Hz of the GPIO clock source with the ID ``nSourceId``, which can be zero to ``GPIO_CLOCK_SOURCE_ID_MAX``. The returned value is ``GPIO_CLOCK_SOURCE_UNUSED``, if the clock source is unused.

.. cpp:function:: unsigned CMachineInfo::GetDevice (TDeviceId DeviceId) const

	Returns the device number of the default I2C master in the system. ``DeviceId`` has to be set to ``DeviceI2CMaster``.

.. cpp:function:: boolean CMachineInfo::ArePWMChannelsSwapped (void) const

	Returns ``TRUE``, if the left PWM audio channel is PWM1 (not PWM0).

DMA channels
""""""""""""

.. cpp:function:: unsigned CMachineInfo::AllocateDMAChannel (unsigned nChannel)

	Allocates an available DMA channel from the platform DMA controller. ``nChannel`` can be ``DMA_CHANNEL_NORMAL`` (normal DMA engine requested), ``DMA_CHANNEL_LITE`` (lite (or normal) DMA engine requested), ``DMA_CHANNEL_EXTENDED`` ("large address" DMA4 engine requested, on Raspberry Pi 4 only) or an explicit channel number (0-15). Returns the allocated channel number or ``DMA_CHANNEL_NONE`` on failure.

.. cpp:function:: void CMachineInfo::FreeDMAChannel (unsigned nChannel)

	Release an allocated DMA channel. ``nChannel`` is the channel number (0-15).

CKernelOptions
^^^^^^^^^^^^^^

The class ``CKernelOptions`` provides the values of runtime options, which can be defined in the file *cmdline.txt* on the SD card. The supported options are listed in `doc/cmdline.txt <https://github.com/rsta2/circle/blob/master/doc/cmdline.txt>`_. There is exactly one or no instance of this class in the system. Only relatively simple programs can work without an instance of ``CKernelOptions``.

.. code-block:: c++

	#include <circle/koptions.h>

.. cpp:class:: CKernelOptions

.. cpp:function:: static CKernelOptions *CKernelOptions::Get (void)

	Returns a pointer to the only instance of ``CKernelOptions``.

.. cpp:function:: unsigned CKernelOptions::GetWidth (void) const
.. cpp:function:: unsigned CKernelOptions::GetHeight (void) const

	Return the requested width and height of the screen, or zero if not specified. These values will normally handed over to the constructor for the class ``CScreenDevice``.

.. cpp:function:: const char *CKernelOptions::GetLogDevice (void) const
.. cpp:function:: unsigned CKernelOptions::GetLogLevel (void) const

	Return the name of the target device for the system log (default ``tty1``) and the log level (default ``LogDebug``), to be handed over to the constructor of the class ``CLogger``.

.. cpp:function:: const char *CKernelOptions::GetKeyMap (void) const

	Returns the country code of the requested keyboard map (option ``keymap=``). The default can be set with the system option ``DEFAULT_KEYMAP``.

.. cpp:function:: unsigned CKernelOptions::GetUSBPowerDelay (void) const

	Returns the requested USB power-on delay in milliseconds, or zero to use the default value.

.. cpp:function:: boolean CKernelOptions::GetUSBFullSpeed (void) const

	Returns ``TRUE``, if the option ``usbspeed=full`` is given in *cmdline.txt*.

.. cpp:function:: const char *CKernelOptions::GetSoundDevice (void) const

	Returns the configured sound device (option ``sounddev=``). Defaults to an empty string.

.. cpp:function:: unsigned CKernelOptions::GetSoundOption (void) const

	Returns the value configured with the option ``soundopt=`` in *cmdline.txt* (0-2, default 0).

.. cpp:function:: TCPUSpeed CKernelOptions::GetCPUSpeed (void) const

	Returns ``CPUSpeedMaximum``, if the option ``fast=true`` is given in *cmdline.txt*, or ``CPUSpeedLow`` otherwise.

.. cpp:function:: unsigned CKernelOptions::GetSoCMaxTemp (void) const

	Returns the enforced maximal temperature of the SoC (option ``socmaxtemp=``) in degrees Celsius (default 60).
