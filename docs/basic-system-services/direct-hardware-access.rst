Direct hardware access
~~~~~~~~~~~~~~~~~~~~~~

Circle applications may need to directly read or write registers of hardware devices, if a driver for the respective device does not exist yet. This subsection describes the Circle support for accessing hardware registers.

Functions
^^^^^^^^^

.. code-block:: c

	#include <circle/memio.h>

.. c:function:: u8 read8 (uintptr nAddress)
.. c:function:: u16 read16 (uintptr nAddress)
.. c:function:: u32 read32 (uintptr nAddress)

	Reads a value with the specified bit size from the memory-mapped I/O address ``nAddress`` and returns it.

.. c:function:: void write8 (uintptr nAddress, u8 uchValue)
.. c:function:: void write16 (uintptr nAddress, u16 usValue)
.. c:function:: void write32 (uintptr nAddress, u32 nValue)

	Writes a value with the specified bit size to the memory-mapped I/O address ``nAddress``. ``uchValue``, ``usValue`` and ``nValue`` are the respective values.

.. note::

	An access to a memory-mapped I/O device register must normally be aligned to the access size.

Macros
^^^^^^

The detailed definitions for the different hardware devices of the Raspberry Pi cannot be listed here. Please read the respective header file for details.

.. code-block:: c

	#include <circle/bcm2835.h>

This header file provides macro definitions of memory-mapped I/O addresses for all Raspberry Pi models, described in the `BCM2835 ARM Peripherals <https://www.raspberrypi.org/documentation/hardware/raspberrypi/bcm2835/BCM2835-ARM-Peripherals.pdf>`_ document, especially:

.. c:macro:: ARM_IO_BASE

	Base address of the 16 MB sized main memory-mapped I/O block, valid on the ARM CPU of the respective Raspberry Pi model. This address is normally used from the Circle application.

.. c:macro:: GPU_IO_BASE

	Base address of the 16 MB sized main memory-mapped I/O block, valid on the GPU co-processor. This address is used for operations, which are executed by the GPU or connected devices (e.g. DMA controllers).

.. note::

	A Raspberry Pi has several processing units. We only distinguish here between the ARM CPU, where the Circle application is running on, and all other processing units, where the firmware, accelerated graphics processing and more is executed. We call these processors the GPU or VPU. Please note that from the point of view of the boot order, the ARM CPU is the secondary co-processor.

.. c:macro:: GPU_MEM_BASE

	Base address of the lower (starting at address 0x0 on the ARM CPU) 1 GB memory address range, valid on the GPU and connected devices (e.g. DMA controllers). The legacy platform DMA controller, for instance, can only access this address space for data transfers.

.. c:macro:: BUS_ADDRESS(address)

	Converts the memory address ``address``, valid on the ARM CPU, to a GPU bus address, valid on the GPU and connected devices.

.. code-block:: c

	#include <circle/bcm2836.h>

This header file provides macro definitions of memory-mapped I/O addresses for the Raspberry Pi 2 to 4 and compatible models, described in the `ARM Quad A7 core <https://www.raspberrypi.org/documentation/hardware/raspberrypi/bcm2836/QA7_rev3.4.pdf>`_ document, especially:

.. c:macro:: ARM_LOCAL_BASE

	Base address of the 256 MB sized local memory-mapped I/O block. A number of registers from this block are local to the respective ARM CPU core.

.. code-block:: c

	#include <circle/bcm2711.h>

This header file provides macro definitions of memory-mapped I/O addresses for the Raspberry Pi 4 and compatible models, described in the `BCM2711 ARM Peripherals <https://datasheets.raspberrypi.org/bcm2711/bcm2711-peripherals.pdf>`_ document.

I/O barriers
^^^^^^^^^^^^

The following I/O barriers are especially required on the Raspberry Pi 1 and Zero. On other Raspberry Pi models they have no function.

.. code-block:: c

	#include <circle/synchronization.h>

.. c:macro:: PeripheralEntry()

	If your code directly accesses memory-mapped hardware registers, you should insert this special barrier before the first access to a specific hardware device.

.. c:macro:: PeripheralExit()

	If your code directly accesses memory-mapped hardware registers, you should insert this special barrier after the last access to a specific hardware device.

.. note::

	Most programs would work without ``PeripheralEntry()`` and ``PeripheralExit()``, but to be sure, it should be used as noted. In a few tests there have been issues on the Raspberry Pi 1, where invalid data was read from hardware registers, without these barriers inserted.

	You do not need to care about this, when you access hardware devices using a Circle device driver class, because this is handled inside the driver.
