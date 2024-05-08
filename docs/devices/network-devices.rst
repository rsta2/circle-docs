Network devices
~~~~~~~~~~~~~~~

Network devices allow the low-level access to network interfaces, and provide methods for sending and receiving network frames and additional functions to manage the network access. Circle supports the access to IEEE 802.3 Ethernet and to IEEE 802.11 wireless LAN (WLAN) interfaces. All network device classes are derived from the base class :cpp:class:`CNetDevice`, which defines the low-level API for network applications. Please note, that applications normally use the high-level TCP/IP socket interface, which is provided by the class :cpp:class:`CSocket`.

CNetDevice
^^^^^^^^^^

.. code-block:: cpp

	#include <circle/netdevice.h>

.. cpp:class:: CNetDevice

	This class is the base class for all network device driver classes and defines the low-level API for specific network applications, which want to directly exchange frames via a network interface. Network devices are not registered in the device name service and can be found using the methods :cpp:func:`CNetDevice::GetNetDevice()`.

.. c:macro:: FRAME_BUFFER_SIZE

	This macro defines the maximum size of a sent or received frame on a network interface. Network buffers usually have this size in Circle.

.. cpp:function:: virtual TNetDeviceType CNetDevice::GetType (void)

	Returns the type of this network device, which is one of these:

.. c:type:: TNetDeviceType

	* NetDeviceTypeEthernet
	* NetDeviceTypeWLAN

.. cpp:function:: virtual const CMACAddress *CNetDevice::GetMACAddress (void) const

	Returns a pointer to a MAC address object, which holds our own address at this network interface device.

.. cpp:function:: virtual boolean CNetDevice::IsSendFrameAdvisable (void)

	Returns ``TRUE``, if it is advisable to call ``SendFrame()``.

.. note::

	``SendFrame()`` can be called at any time, but may fail, when the TX queue is full. This method gives a hint, if calling ``SendFrame()`` is advisable.

.. cpp:function:: virtual boolean CNetDevice::SendFrame (const void *pBuffer, unsigned nLength)

	Sends a valid frame to the network. ``pBuffer`` is a pointer to the frame, which does not contain the frame checking sequence (FCS). ``nLength`` is the frame length in bytes. The frame does not need to be padded by the application.

.. cpp:function:: virtual boolean CNetDevice::ReceiveFrame (void *pBuffer, unsigned *pResultLength)

	Polls for a frame, which has been received via the network interface. ``pBuffer`` is a pointer to a buffer, where the frame will be placed, and must have the size :c:macro:`FRAME_BUFFER_SIZE`. ``pResultLength`` is a pointer to a variable, which receives the valid frame length. Returns ``TRUE``, if a frame is returned in the buffer, ``FALSE``, if nothing has been received.

.. cpp:function:: virtual boolean CNetDevice::IsLinkUp (void)

	Returns ``TRUE``, if the physical link (PHY) is active.

.. cpp:function:: virtual TNetDeviceSpeed CNetDevice::GetLinkSpeed (void)

	Returns the speed of the physical link (PHY), if it is active, or ``NetDeviceSpeedUnknown``, if it is not known. The following link speeds are defined:

.. c:type:: TNetDeviceSpeed

	* NetDeviceSpeed10Half
	* NetDeviceSpeed10Full
	* NetDeviceSpeed100Half
	* NetDeviceSpeed100Full
	* NetDeviceSpeed1000Half
	* NetDeviceSpeed1000Full
	* NetDeviceSpeedUnknown

.. cpp:function:: virtual boolean CNetDevice::UpdatePHY (void)

	Updates the device settings according to physical link (PHY) status. Returns ``FALSE``, if this function is not supported.

.. note::

	This method is called continuously every two seconds by the PHY task of the :ref:`TCP/IP networking` subsystem. If you do not use this subsystem, you have to call this method on your own.

.. cpp:function:: static const char *CNetDevice::GetSpeedString (TNetDeviceSpeed Speed)

	Returns a description for the link speed value ``Speed``, which normally has been returned from ``GetLinkSpeed()``.

.. cpp:function:: static CNetDevice *CNetDevice::GetNetDevice (unsigned nDeviceNumber)

	Returns a pointer to the network device object for the zero-based number ``nDeviceNumber`` of a network device, or 0, if the device is not available.

.. cpp:function:: static CNetDevice *CNetDevice::GetNetDevice (TNetDeviceType Type)

	Returns a pointer to the first network device object of the type ``Type``, which is either a specific network device type (see :cpp:func:`CNetDevice::GetType()`), or ``NetDeviceTypeAny`` to search for any network device.

CSMSC951xDevice
^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/usb/smsc951x.h>

.. cpp:class:: CSMSC951xDevice : public CUSBFunction, CNetDevice

	This class is a driver for the SMSC9512 and SMSC9514 Ethernet network interface devices, which are attached to the internal USB hub of Raspberry Pi 1, 2 and 3 Model B boards. This class is automatically instantiated in the USB device enumeration process, when a device of this type is found. This class does not provide specific methods, its API is defined by the base class :cpp:class:`CNetDevice`.

CLAN7800Device
^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/usb/lan7800.h>

.. cpp:class:: CLAN7800Device : public CUSBFunction, CNetDevice

	This class is a driver for the LAN7800 Gigabit Ethernet network interface device, which is attached to an internal USB hub of the Raspberry Pi 3 Model B+ board. This class is automatically instantiated in the USB device enumeration process, when a device of this type is found. This class does not provide specific methods, its API is defined by the base class :cpp:class:`CNetDevice`.

CUSBCDCEthernetDevice
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/usb/usbcdcethernet.h>

.. cpp:class:: CUSBCDCEthernetDevice : public CUSBFunction, CNetDevice

	This class is a driver for the USB CDC Ethernet network interface device, which is supported by QEMU. This class is automatically instantiated in the USB device enumeration process, when a device of this type is found. This class does not provide specific methods, its API is defined by the base class :cpp:class:`CNetDevice`.

CBcm54213Device
^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/bcm54213.h>

.. cpp:class:: CBcm54213Device : public CNetDevice

	This class is a driver for the BCM54213PE Gigabit Ethernet Transceivers of the Raspberry Pi 4, 400 and Compute Module 4. It is instantiated in the :ref:`TCP/IP networking` subsystem, but has to be manually instantiated by applications, which do not use this subsystem. This class does not provide specific methods, its API is defined by the base class :cpp:class:`CNetDevice`.

CMACBDevice
^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/macb.h>

.. cpp:class:: CMACBDevice : public CNetDevice

	This class is a driver for the MACB / GEM Gigabit Ethernet Transceiver of the Raspberry Pi 5. It is instantiated in the :ref:`TCP/IP networking` subsystem, but has to be manually instantiated by applications, which do not use this subsystem. This class does not provide specific methods, its API is defined by the base class :cpp:class:`CNetDevice`.

CBcm4343Device
^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <wlan/bcm4343.h>

.. cpp:class:: CBcm4343Device : public CNetDevice

	This class is a driver for the BCM4343x WLAN interface device of the Raspberry Pi 3, 4, 5 and Zero (2) W. It has to be instantiated manually, and is normally used together with the class :cpp:class:`CNetSubSystem` from the :ref:`TCP/IP networking` subsystem and the class :cpp:class:`CWPASupplicant` from the submodule `hostap <https://github.com/rsta2/hostap/tree/hostap_0_7_0-circle>`_. This class provides the interface, defined in its base class :cpp:class:`CNetDevice`, and additional methods, which are needed to manage the association with a WLAN access point (AP). The following description covers only the methods, which are specific to this class.

.. cpp:function:: CBcm4343Device::CBcm4343Device (const char *pFirmwarePath)

	Creates an instance of this class. ``pFirmwarePath`` points to the path, where the firmware files for the WLAN controller are provided (e.g. "SD:/firmware/").

.. cpp:function:: boolean CBcm4343Device::Initialize (void)

	Initializes the WLAN controller and driver. Returns ``TRUE`` on success.

.. cpp:function:: void CBcm4343Device::RegisterEventHandler (TBcm4343EventHandler *pHandler, void *pContext)

	Registers the event handler ``pHandler``, which is called on some specific WLAN events (e.g. disassociation from AP). ``pContext`` is a user pointer, which is handed over to the event handler. ``pHandler`` can be 0 to unregister the event handler.

.. cpp:function:: boolean CBcm4343Device::Control (const char *pFormat, ...)

	Sends the device specific control command ``pFormat`` with optional parameters to the WLAN device driver. Returns ``TRUE`` on success.

.. cpp:function:: boolean CBcm4343Device::ReceiveScanResult (void *pBuffer, unsigned *pResultLength)

	Polls for a received scan result message. ``pBuffer`` is a pointer to a buffer, where the message will be placed. The buffer must have the size :c:macro:`FRAME_BUFFER_SIZE`. ``pResultLength`` is a pointer to a variable, which receives the valid message length. Returns ``TRUE``, if a message is returned in the buffer, or ``FALSE`` if nothing has been received.

.. cpp:function:: const CMACAddress *CBcm4343Device::GetBSSID (void)

	Returns the BSSID of the associated AP.

.. cpp:function:: boolean CBcm4343Device::JoinOpenNet (const char *pSSID)

	Joins the open WLAN network with the SSID ``pSSID``. Returns ``TRUE`` on success.

.. cpp:function:: boolean CBcm4343Device::CreateOpenNet (const char *pSSID, int nChannel, bool bHidden)

	Creates an open WLAN network (AP mode) with the SSID ``pSSID`` on channel ``nChannel``. The SSID is hidden, if ``bHidden`` is ``TRUE``. Returns ``TRUE`` on success.

.. cpp:function:: boolean CBcm4343Device::DestroyOpenNet (void)

	Destroys a created open WLAN network. It can be created afterwards again. Returns ``TRUE`` on success.
