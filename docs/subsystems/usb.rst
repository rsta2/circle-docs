USB
~~~

The USB (Universal Serial Bus) subsystem provides services and device drivers, which support the access to USB 2.0 and USB 3.0 (on the Raspberry Pi 4 and 5 only) devices. Essentially, this concerns drivers for the DWHCI OTG USB controller of the Raspberry Pi 1-3 and Zero (host and gadget mode) and the xHCI USB controller(s) of the Raspberry Pi 4, 400, 5 and Compute Module 4 (host mode only), USB device class drivers, some vendor specific USB device drivers and support classes for all these drivers.

Most of the operations in this subsystem are hidden from applications behind device driver interfaces, which will be described in the :ref:`Devices` section. An application, which uses the USB, has especially to deal with the initialization of the USB support at system startup and optionally with detecting newly attached USB devices, while the system is running (USB plug-and-play). This section is limited to these topics.

Please read the file `doc/usb-plug-and-play.txt <https://github.com/rsta2/circle/blob/master/doc/usb-plug-and-play.txt>`_ for general information about the (optional) USB plug-and-play support in Circle.

.. important::

	Please note that Circle does not support OTG protocols, so the USB controller always works in host or gadget mode and the connected peer must work in the opposite mode.

CUSBController
^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/usb/usbcontroller.h>

.. cpp:class:: CUSBController

	This class defines the interface to all USB (host or gadget) controllers in Circle. It is provided to allow a unique handling of USB host and gadget controllers in applications, which support both kind of controllers.

.. cpp:function:: virtual boolean CUSBController::Initialize (boolean bScanDevices = TRUE) = 0

	Initializes the USB (host or gadget) controller. The parameter ``bScanDevices`` is currently only supported for USB host controllers. See :cpp:func:`CUSBHCIDevice::Initialize()` for its description. Returns ``TRUE`` on success.

.. cpp:function:: virtual boolean CUSBController::UpdatePlugAndPlay (void) = 0

	Updates the information about connected USB devices. This method must be called continuously from ``TASK_LEVEL``, when USB plug-and-play is enabled. Returns ``TRUE``, if the USB device tree might have been changed. The application should test for the existence of devices, which it supports, by invoking :cpp:func:`CDeviceNameService::GetDevice()` then. ``UpdatePlugAndPlay()`` always returns ``TRUE`` on its first call.

USB host support
^^^^^^^^^^^^^^^^

In host mode an USB controller supports the connection of one or more USB devices (aka gadgets, peripherals).

CUSBHCIDevice
"""""""""""""

.. code-block:: cpp

	#include <circle/usb/usbhcidevice.h>

.. cpp:class:: CUSBHCIDevice : public CUSBHostController

	This class is the base of the USB host support in a Circle application. To use USB in host mode, you should create a member of this class in the ``CKernel`` class of your application.

.. note::

	Actually there is not really a class ``CUSBHCIDevice`` available in Circle. Instead, three classes ``CDWHCIDevice``, ``CXHCIDevice`` (both derived from ``CUSBHostController``) and ``CUSBSubSystem`` (on Raspberry Pi 5) exist for the respective USB host controllers of the target Raspberry Pi model, and ``CUSBHCIDevice`` is only an alias for these class names, defined as macro. To ensure, that an application can be built for each Raspberry Pi model, you should use the name ``CUSBHCIDevice`` only.

	Some methods available via ``CUSBHCIDevice`` are defined in its base class :ref:`CUSBHostController` and can be called using a pointer to a ``CUSBHostController`` object too.

.. cpp:function:: CUSBHCIDevice::CUSBHCIDevice (CInterruptSystem *pInterruptSystem, CTimer *pTimer, boolean bPlugAndPlay = FALSE)

	Creates an instance of this class. ``pInterruptSystem`` is a pointer to the interrupt system object and ``pTimer`` a pointer to the system timer object. ``bPlugAndPlay`` must be set to ``TRUE`` to enable the USB plug-and-play support. This is optional and requires further support by the application.

.. cpp:function:: boolean CUSBHCIDevice::Initialize (boolean bScanDevices = TRUE)

	Initializes the USB host subsystem. Normally this includes a bus scan and the initialization of all attached USB devices, which takes some time. To speed-up the USB initialization, ``bScanDevices`` can be set to ``FALSE``, if USB plug-and-play was enabled in the constructor of this class (``bPlugAndPlay = TRUE``). The device initialization will be deferred to a later call of ``UpdatePlugAndPlay()`` then.

.. cpp:function:: void CUSBHCIDevice::ReScanDevices (void)

	This method can be invoked to re-scan the USB for newly attached devices, in case USB plug-and-play support has not been enabled, when calling the constructor of this class (``bPlugAndPlay = FALSE``).

.. _CUSBHostController:

CUSBHostController
""""""""""""""""""

.. code-block:: cpp

	#include <circle/usb/usbhostcontroller.h>

.. cpp:class:: CUSBHostController : public CUSBController

	This is the base class of ``CDWHCIDevice`` and ``CXHCIDevice`` (aka ``CUSBHCIDevice``). The following methods can be called for an instance of these classes too.

.. cpp:function:: boolean CUSBHostController::IsPlugAndPlay (void)

	Returns ``TRUE``, if USB plug-and-play is supported by the USB subsystem.

.. cpp:function:: boolean CUSBHostController::UpdatePlugAndPlay (void)

	If USB plug-and-play is enabled, this method must be called continuously from ``TASK_LEVEL``, so that the internal USB device tree can be updated, if new devices have been attached or devices have been removed from the USB. Returns ``TRUE``, if the USB device tree might have been changed. The application should test for the existence of devices, which it supports, by invoking ``CDeviceNameService::GetDevice()`` then. ``UpdatePlugAndPlay()`` always returns ``TRUE`` on its first call.

.. cpp:function:: static boolean CUSBHostController::IsActive (void)

	Returns ``TRUE``, if the USB subsystem is available.

.. cpp:function:: static CUSBHostController *CUSBHostController::Get (void)

	Returns a pointer to the only instance of ``CUSBHostController`` (aka ``CUSBHCIDevice``) in the system.

USB gadget support
^^^^^^^^^^^^^^^^^^

In gadget (aka device, peripheral) mode an USB controller supports the connection to exactly one USB host.

.. note::

	USB gadget support is currently not available for Raspberry Pi 5.

CUSBCDCGadget
"""""""""""""

.. code-block:: cpp

	#include <circle/usb/gadget/usbcdcgadget.h>

.. cpp:class:: CUSBCDCGadget : public CDWUSBGadget

	This class implements an USB serial CDC gadget, which can transfer data to/from the USB host via a serial interface. The device appears in the host system as a USB serial device (e.g. `/dev/ttyACM0`). To use USB for this purpose, you should create a member of this class in the ``CKernel`` class of your application. Only the constructor of this class is described here. More methods are described for the base class :cpp:class:`CDWUSBGadget`. The gadget driver automatically creates an instance of the interface device :cpp:class:`CUSBSerialDevice`, when the gadget is connected to an USB host.

.. note::

	The `test/usb-serial-cdc-gadget` is prepared to work as a serial CDC gadget. Please read the *README* file in the test's directory for information about the required configuration. You have to define your own USB vendor ID as system option ``USB_GADGET_VENDOR_ID``.

.. cpp:function:: CUSBCDCGadget::CUSBCDCGadget (CInterruptSystem *pInterruptSystem)

	Creates an instance of this class. ``pInterruptSystem`` is a pointer to the interrupt system object.

CUSBMSDGadget
"""""""""""""

.. code-block:: cpp

	#include <circle/usb/gadget/usbmsdgadget.h>

.. cpp:class:: CUSBMSDGadget : public CDWUSBGadget

	This class implements an USB mass-storage device gadget, which can be mounted to an USB host. The device appears in the host system as an external USB drive. You should create a member of this class in the ``CKernel`` class of your application. Only a few methods of this class are described here. More methods are described for the base class :cpp:class:`CDWUSBGadget`.

.. note::

	The `test/usb-msd-gadget` is prepared to work as a mass-storage device gadget. Please read the *README* file in the test's directory for information about the required configuration. You have to define your own USB vendor ID as system option ``USB_GADGET_VENDOR_ID``.

.. cpp:function:: CUSBMSDGadget::CUSBMSDGadget (CInterruptSystem *pInterruptSystem, CDevice *pDevice = nullptr)

	Creates an instance of this class. ``pInterruptSystem`` is a pointer to the interrupt system object. ``pDevice`` can be a pointer to the block device, to be controlled by this gadget. The block device must be initialized yet, when it is specified here. :cpp:func:`SetDevice()` has to be called later, when ``pDevice`` is not specified here.

.. cpp:function:: void CUSBMSDGadget::SetDevice (CDevice *pDevice)

	Call this, if ``pDevice`` has not been specified to the constructor. ``pDevice`` Is a pointer to the block device, to be controlled by this gadget

.. cpp:function:: void CUSBMSDGadget::Update (void)

	This method must be called periodically from ``TASK_LEVEL`` to allow I/O operations.

CUSBMIDIGadget
""""""""""""""

.. code-block:: cpp

	#include <circle/usb/gadget/usbmidigadget.h>

.. cpp:class:: CUSBMIDIGadget : public CDWUSBGadget

	This class implements an USB MIDI (v1.0) gadget, which can receive MIDI events from the USB host (e.g. from a sequencer program) and/or can send MIDI events to the host. To use USB for this purpose, you should create a member of this class in the ``CKernel`` class of your application. Only the constructor of this class is described here. More methods are described for the base class :cpp:class:`CDWUSBGadget`. The gadget driver automatically creates an instance of the interface device :cpp:class:`CUSBMIDIDevice`, when the gadget is connected to an USB host.

.. note::

	The `sample/29-miniorgan` is prepared to work as a MIDI gadget. Please read the *README* file in the sample's directory for information about the required configuration. Beside the define ``USB_GADGET_MODE``, which enables the gadget mode in the sample, you have to define your own USB vendor ID as system option ``USB_GADGET_VENDOR_ID``.

.. cpp:function:: CUSBMIDIGadget::CUSBMIDIGadget (CInterruptSystem *pInterruptSystem)

	Creates an instance of this class. ``pInterruptSystem`` is a pointer to the interrupt system object.

CDWUSBGadget
""""""""""""

.. code-block:: cpp

	#include <circle/usb/gadget/dwusbgadget.h>

.. cpp:class:: CDWUSBGadget : public CUSBController

	This class is the base class of all USB gadgets in Circle. It is supported for the Raspberry Pi models (3)A(+), Zero (2) (W) and 4B only. Only the methods, which are interesting for application usage, are described here. More methods are described for the base class :cpp:class:`CUSBController`.

.. note::

	USB gadgets always support USB plug-and-play in Circle.

.. cpp:function:: virtual const void *CDWUSBGadget::GetDescriptor (u16 wValue, u16 wIndex, size_t *pLength) = 0

	Returns a device-specific USB descriptor. ``wValue`` is a parameter from setup packet (descriptor type (MSB) and index (LSB)). ``wIndex`` is a parameter from setup packet (e.g. language ID for string descriptors). ``pLength`` is a pointer to a variable, which receives the descriptor size. Returns a pointer to the descriptor or ``nullptr``, if it is not available.

.. note::

	You may override this virtual method to provide user-specific (e.g. string) descriptors for your gadget.
