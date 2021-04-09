USB
~~~

The USB (Universal Serial Bus) subsystem provides services and device drivers, which support the access to USB 2.0 and USB 3.0 (on the Raspberry Pi 4 only) devices. Essentially, this concerns drivers for the DWHCI OTG USB controller of the Raspberry Pi 1-3 and Zero (host mode only) and the xHCI USB controller(s) of the Raspberry Pi 4 (400 and Compute Module 4 too), USB device class drivers, some vendor specific USB device drivers and support classes for all these drivers.

Most of the operations in this subsystem are hidden from applications behind device driver interfaces, which will be described in the `Devices` section. An application, which uses the USB, has especially to deal with the initialization of the USB support at system startup and optionally with detecting newly attached USB devices, while the system is running (USB plug-and-play). This section is limited to these topics.

Please read the file `doc/usb-plug-and-play.txt <https://github.com/rsta2/circle/blob/master/doc/usb-plug-and-play.txt>`_ for general information about the (optional) USB plug-and-play support in Circle.

CUSBHCIDevice
^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/usb/usbhcidevice.h>

.. cpp:class:: CUSBHCIDevice : public CUSBHostController

	This class is the base of the USB support in a Circle application. To use USB, you should create a member of this class in the ``CKernel`` class of your application.

.. note::

	Actually there is not really a class ``CUSBHCIDevice`` available in Circle. Instead, two classes ``CDWHCIDevice`` and ``CXHCIDevice`` (both derived from ``CUSBHostController``) exist for the respective USB host controllers of the target Raspberry Pi model, and ``CUSBHCIDevice`` is only an alias for these class names, defined as macro. To ensure, that an application can be built for each Raspberry Pi model, you should use the name ``CUSBHCIDevice`` only.

	Some methods available via ``CUSBHCIDevice`` are defined in its base class :ref:`CUSBHostController` and can be called using a pointer to a ``CUSBHostController`` object too.

.. cpp:function:: CUSBHCIDevice::CUSBHCIDevice (CInterruptSystem *pInterruptSystem, CTimer *pTimer, boolean bPlugAndPlay = FALSE)

	Creates an instance of this class. ``pInterruptSystem`` is a pointer to the interrupt system object and ``pTimer`` a pointer to the system timer object. ``bPlugAndPlay`` must be set to ``TRUE`` to enable the USB plug-and-play support. This is optional and requires further support by the application.

.. cpp:function:: boolean CUSBHCIDevice::Initialize (boolean bScanDevices = TRUE)

	Initializes the USB subsystem. Normally this includes a bus scan and the initialization of all attached USB devices, which takes some time. To speed-up the USB initialization, ``bScanDevices`` can be set to ``FALSE``, if USB plug-and-play was enabled in the constructor of this class (``bPlugAndPlay = TRUE``). The device initialization will be deferred to a later call of ``UpdatePlugAndPlay()`` then.

.. cpp:function:: void CUSBHCIDevice::ReScanDevices (void)

	This method can be invoked to re-scan the USB for newly attached devices, in case USB plug-and-play support has not been enabled, when calling the constructor of this class (``bPlugAndPlay = FALSE``).

.. _CUSBHostController:

CUSBHostController
^^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/usb/usbhostcontroller.h>

.. cpp:class:: CUSBHostController

	This is the base class of ``CDWHCIDevice`` and ``CXHCIDevice`` (aka ``CUSBHCIDevice``). The following methods can be called for an instance of these classes too.

.. cpp:function:: static boolean CUSBHostController::IsPlugAndPlay (void)

	Returns ``TRUE``, if USB plug-and-play is supported by the USB subsystem.

.. cpp:function:: boolean CUSBHostController::UpdatePlugAndPlay (void)

	If USB plug-and-play is enabled, this method must be called continuously from ``TASK_LEVEL``, so that the internal USB device tree can be updated, if new devices have been attached or devices have been removed from the USB. Returns ``TRUE``, if the USB device tree might have been changed. The application should test for the existence of devices, which it supports, by invoking ``CDeviceNameService::GetDevice()`` then. ``UpdatePlugAndPlay()`` always returns ``TRUE`` on its first call.

.. cpp:function:: static boolean CUSBHostController::IsActive (void)

	Returns ``TRUE``, if the USB subsystem is available.

.. cpp:function:: static CUSBHostController *CUSBHostController::Get (void)

	Returns a pointer to the only instance of ``CUSBHostController`` (aka ``CUSBHCIDevice``) in the system.
