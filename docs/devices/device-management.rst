Device management
~~~~~~~~~~~~~~~~~

In Circle most devices are represented by two things:

* By a device specific object, an instance of a class, which is derived from the class ``CDevice``.
* By a device name, a C-string, which allows to retrieve a pointer to the device object, using the Circle device name service, which is implemented by the class ``CDeviceNameService``.

.. note::

	The I/O system of Circle is not as uniform as that of Linux, for example. Some device classes have specific interfaces, different from the well-known ``Read()`` and ``Write()`` interface and are not derived from ``CDevice``.

CDevice
^^^^^^^

.. code-block:: c++

	#include <circle/device.h>

.. cpp:class:: CDevice

	This class is the base class for most device classes in Circle.

.. cpp:function:: virtual int CDevice::Read (void *pBuffer, size_t nCount)

	Performs a read operation of up to ``nCount`` bytes from a device to ``pBuffer``. Returns the number of read bytes or < 0 on failure.

.. cpp:function:: virtual int CDevice::Write (const void *pBuffer, size_t nCount)

	Performs a write operation of up to ``nCount`` bytes to a device from ``pBuffer``. Returns the number of written bytes or < 0 on failure.

.. cpp:function:: virtual u64 CDevice::Seek (u64 ullOffset)

	Sets the position of the read/write pointer of a device to the byte offset ``ullOffset``. Returns the resulting offset, or ``(u64) -1`` on failure. This method is only implemented for block devices, character devices always return failure.

.. cpp:function:: virtual u64 CDevice::GetSize (void) const

	Returns the total byte size of a block device, or ``(u64) -1`` on failure. This method is only implemented for block devices, character devices always return failure.

.. cpp:function:: virtual boolean CDevice::RemoveDevice (void)

	Requests the remove of a device from the system for pseudo plug-and-play. This is only implemented for USB devices (e.g. for USB mass-storage devices). Returns ``TRUE`` on the successful removal of the device.

.. cpp:function:: CDevice::TRegistrationHandle CDevice::RegisterRemovedHandler (TDeviceRemovedHandler *pHandler, void *pContext = 0)

	Registers a callback, which is invoked, when this device is removed from the system in terms of hot-plugging. ``pHandler`` gets called, before the device object is deleted. ``pContext`` is a user pointer, which is handed over to the handler. Returns a handle to be handed over to :cpp:func:`CDevice::UnregisterRemovedHandler()`. This method can be called multiple times for a specific device, where the registered handlers will be called in reverse order. Calling this method with ``pHandler = 0`` to unregister is not supported any more.

.. code-block:: c++

	void TDeviceRemovedHandler (CDevice *pDevice, void *pContext);

.. cpp:function:: void CDevice::UnregisterRemovedHandler (TRegistrationHandle hRegistration)

	Undo the registration of a device removed handler. ``hRegistration`` is the handle, which has been returned by :cpp:func:`CDevice::RegisterRemovedHandler()`.

.. note::

	See the file `doc/usb-plug-and-play.txt <https://github.com/rsta2/circle/blob/master/doc/usb-plug-and-play.txt>`_ for detailed information on USB plug-and-play support in Circle!

CDeviceNameService
^^^^^^^^^^^^^^^^^^

.. code-block:: c++

	#include <circle/devicenameservice.h>

.. cpp:class:: CDeviceNameService

	In Circle devices can be registered by name and retrieved later using the same name. This is implemented in the class ``CDeviceNameService``.

.. note::

	A device name usually consists of an alpha name prefix, followed by a decimal device index number, which is >= 1. Partitions on block devices have another partition index, which is >= 1 too. Sound devices do not have a device index number. Examples:

	==============	====================================================
	Device name	Description
	==============	====================================================
	tty1		First screen device
	ukbd1		First USB keyboard device
	umsd1		First USB mass-storage device (e.g. flash drive)
	umsd1-1		First partition on the first USB mass-storage device
	sndpwm		PWM sound device
	null		Null device
	==============	====================================================

.. cpp:function:: static CDeviceNameService *CDeviceNameService::Get (void)

	Returns a pointer to the single ``CDeviceNameService`` instance in the system.

.. cpp:function:: CDevice *CDeviceNameService::GetDevice (const char *pName, boolean bBlockDevice)

	Returns a pointer to the device object of the device, with the name ``pName`` and the device type ``bBlockDevice``, or 0 if the device is not found. ``bBlockDevice`` is ``TRUE``, if this is a block device, otherwise it is a character device.

.. cpp:function:: CDevice *CDeviceNameService::GetDevice (const char *pPrefix, unsigned nIndex, boolean bBlockDevice)

	Returns a pointer to the device object of the device, with the name prefix ``pName``, the device index ``nIndex`` and the device type ``bBlockDevice``, or 0 if the device is not found. ``bBlockDevice`` is ``TRUE``, if this is a block device, otherwise it is a character device. The resulting name consists of the name prefix followed by the decimal device index (e.g. ``umsd1`` for the first USB mass-storage device).

.. cpp:function:: void CDeviceNameService::ListDevices (CDevice *pTarget)

	Generates a textual device name listing and writes it to the device ``pTarget``.

.. cpp:function:: void CDeviceNameService::AddDevice (const char *pName, CDevice *pDevice, boolean bBlockDevice)

	Adds the pointer ``pDevice`` to a device object with the name ``pName`` to the device name registry. ``bBlockDevice`` is ``TRUE``, if this is a block device, otherwise it is a character device. This method is usually only used by device driver classes.

.. cpp:function:: void CDeviceNameService::AddDevice (const char *pPrefix, unsigned nIndex, CDevice *pDevice, boolean bBlockDevice)

	Adds the pointer ``pDevice`` to a device object with the name prefix ``pName`` and device index ``nIndex`` to the device name registry. ``bBlockDevice`` is ``TRUE``, if this is a block device, otherwise it is a character device. The resulting name consists of the name prefix followed by the decimal device index (e.g. ``umsd1`` for the first USB mass-storage device). This method is usually only used by device driver classes.

.. cpp:function:: void CDeviceNameService::RemoveDevice (const char *pName, boolean bBlockDevice)

	Removes the device with the name ``pName`` and the device type ``bBlockDevice`` from the device name registry. ``bBlockDevice`` is ``TRUE``, if this is a block device, otherwise it is a character device. This method is usually only used by device driver classes.

.. cpp:function:: void CDeviceNameService::RemoveDevice (const char *pPrefix, unsigned nIndex, boolean bBlockDevice)

	Removes the device with the name prefix ``pPrefix``, the device index ``nIndex`` and the device type ``bBlockDevice`` from the device name registry. ``bBlockDevice`` is ``TRUE``, if this is a block device, otherwise it is a character device. The resulting name consists of the name prefix followed by the decimal device index (e.g. ``umsd1`` for the first USB mass-storage device). This method is usually only used by device driver classes.
