Block devices
~~~~~~~~~~~~~

Block devices provide the access to physical or logical drives (e.g. SD card, USB flash drive). They allow to read and write consecutive blocks of bytes of a fixed block size. Circle supports only block devices with a size of 512 bytes. All block devices provide the following methods, which are derived from the class :cpp:class:`CDevice`. The detailed class descriptions below list additional class-specific methods only.

==============	======================================	============================
Method		Purpose					Description
==============	======================================	============================
Read()		read block(s) from device		:cpp:func:`CDevice::Read()`
Write()		write block(s) to device		:cpp:func:`CDevice::Write()`
Seek()		set read/write pointer position		:cpp:func:`CDevice::Seek()`
==============	======================================	============================

CEMMCDevice
^^^^^^^^^^^

.. code-block:: cpp

	#include <SDCard/emmc.h>

.. cpp:class:: CEMMCDevice : public CDevice

	This class provides the physical access to SD cards and to embedded MMC memory on the Compute Module 4. This class has to be manually instantiated, if an application wants to access one of these devices. This is demonstrated in `addon/SDCard/sample <https://github.com/rsta2/circle/tree/master/addon/SDCard/sample>`_. There can be only one instance of this device, which has the name ``emmc1`` in the device name service.

	This class has drivers for two different interfaces, the SDHOST interface and the EMMC interface. The SDHOST interface is enabled by default on the Raspberry Pi 1-3 and Zero, when the system option ``REALTIME`` is not enabled. On the Raspberry Pi 4 the EMMC interface is used in any case, but can be used on the earlier models with the system option ``NO_SDHOST`` too.  This is not possible, when you want to access the on-board WLAN device at the same time. To access the embedded MMC on the Compute Module 4, the system option ``USE_EMBEDDED_MMC_CM4`` has to be enabled.

.. note::

	On the Raspberry Pi 5 the SDHOST interface is currently not supported by Circle.

.. cpp:function:: CEMMCDevice::CEMMCDevice (CInterruptSystem *pInterruptSystem, CTimer *pTimer, CActLED *pActLED = 0)

	Creates the instance of this class. ``pInterruptSystem`` is a pointer to the system interrupt object. ``pTimer`` is a pointer to the system timer object. ``pActLED`` can be specified to use the green Activity LED to inform the user, when the SD card is currently accessed. This is optional.

.. cpp:function:: boolean CEMMCDevice::Initialize (void)

	Initializes the EMMC or SDHOST device and detects the inserted SD card. Returns ``TRUE`` on success.

.. cpp:function:: const u32 *CEMMCDevice::GetID (void)

	Returns a pointer to the 32 byte (four ``u32`` words) long identifier of the inserted SD card. This information can be used to recognize a specific SD card again and is only valid, when ``Initialize()`` was successfully called before.

CUSBBulkOnlyMassStorageDevice
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/usb/usbmassdevice.h>

.. cpp:class:: CUSBBulkOnlyMassStorageDevice : public CUSBFunction

	This class provides the physical access to USB mass-storage devices (e.g. flash drive, hard disk), which support the `USB Mass Storage Bulk Only 1.0 <https://usb.org/document-library/mass-storage-bulk-only-10>`_ specification. An instance of this class is automatically created in the USB device enumeration process, when a compatible USB device (interface 8-6-50) is found. These devices have the name ``umsdN`` (N >= 1) in the device name service.

.. cpp:function:: unsigned CUSBBulkOnlyMassStorageDevice::GetCapacity (void) const

	Returns the capacity of the device in number of 512 Byte blocks.

.. note::

	Circle supports USB mass-storage devices with up to 2 TBytes capacity.

CPartition
^^^^^^^^^^

.. code-block:: cpp

	#include <circle/fs/partition.h>

.. cpp:class:: CPartition : public CDevice

	This class encapsulates one primary partition of a block device with Master Boot Record (MBR). An instance of this class is automatically created, when a block device object is initialized and a primary partition is found, when the MBR is scanned. Extended partitions (partition types 0x05 and 0x0F) and EFI partitions (type 0xEF) will be ignored in this process. These partition devices have the name ``DEV-N`` (N >= 1) in the device name service, where DEV is the name of the physical block device. For example the first found partition on a SD card has the name ``emmc1-1``. This class only supports the standard methods of the :cpp:class:`CDevice` class.

.. note::

	These partition devices are only accessed by the Circle-native FAT filesystem support (class :cpp:class:`CFATFileSystem`), but not by the :ref:`FatFs library`, which implements its own MBR management.
