.. _Audio devices:

Audio devices
~~~~~~~~~~~~~

Circle supports the generation of sound via several hardware (PWM, I2S, HDMI, USB) and software (VCHIQ) interfaces. It allows to capture sound data via the I2S hardware interface and via USB audio streaming devices. Furthermore it is able to exchange MIDI data via USB and via a serial interface (UART). The latter has to be implemented in the application using the class :cpp:class:`CSerialDevice`.

.. important::

	The support for USB audio streaming devices is only available on the Raspberry Pi 4, 400 and Compute Module 4.

The base class of all sound generating and capturing devices is ``CSoundBaseDevice``. The following table lists the provided classes for the different interfaces. The higher level support provides an additional conversion function for sound data in different formats as an example, which can be easily adapted for other sound classes.

==============	======================	======================	====================
Interface	Connector		Low level support	Higher level support
==============	======================	======================	====================
PWM		3.5" headphone jack	CPWMSoundBaseDevice	CPWMSoundDevice
I2S		GPIO header		CI2SSoundBaseDevice
HDMI		HDMI(0)			CHDMISoundBaseDevice
USB		Jack of USB device	CUSBSoundBaseDevice
VCHIQ		HDMI or headphone jack	CVCHIQSoundBaseDevice	CVCHIQSoundDevice
==============	======================	======================	====================

.. note::

	The class ``CUSBSoundBaseDevice`` depends on more lower level drivers (e.g. class ``CUSBAudioStreamingDevice``) in the USB library, which are normally not accessed directly by an application. Technical details of the USB audio streaming architecture are described in the file *lib/usb/usbaudiostreaming.cpp*.

Several sample programs demonstrate functions of the different audio devices:

* sample/12-pwmsound (playback a short sound sample using the PWM sound device)
* sample/29-miniorgan (using the PWM, HDMI, I2S or USB sound device, USB or serial MIDI, using sound controller to modify volume)
* sample/34-sounddevices (integrating multiple sound devices in one application)
* sample/42-soundinput (I2S or USB to PWM sound data converter and recorder)
* addon/vc4/sound/sample (HDMI or PWM sound support via VCHIQ interface)
* test/sound-controller (set controls of a sound controller of several sound devices)

The separate project `MiniSynth Pi <https://github.com/rsta2/minisynth>`_ is a more extensive example for an application, which generates sound via the PWM, I2S or USB interfaces in a multi-core environment, controlled with an USB or serial MIDI stream.

CSoundBaseDevice
^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/sound/soundbasedevice.h>

.. cpp:class:: CSoundBaseDevice : public CDevice

	This class is the base for all sound generating and capturing classes in Circle. Normally it is not used directly in applications, but instead the derived class for the used interface is instantiated. Because this base class defines the common interface for all sound classes, it is described here first.

	This class provides methods to start and stop the sound output and input, and to setup and manipulate one sound queue for each direction. Applications can use these queue(s) to provide/retrieve sound data with ``Write()`` and/or ``Read()``. Alternatively they can override the methods ``GetChunk()`` and/or ``PutChunk()`` to directly write/read the audio samples to/from a provided DMA buffer.

.. important::

	In a multi-core environment all methods, except if otherwise noted, have to be called or will be called (for callbacks) on CPU core 0.

Device information
""""""""""""""""""

.. cpp:function:: unsigned CSoundBaseDevice::GetHWTXChannels (void) const

	Returns the number of hardware output channels. This method can be called on any CPU core.

.. cpp:function:: unsigned CSoundBaseDevice::GetHWRXChannels (void) const

	Returns the number of hardware input channels. This method can be called on any CPU core.

Device activation
"""""""""""""""""

.. cpp:function:: virtual boolean CSoundBaseDevice::Start (void)

	Starts the transmission of sound data and initializes the device at the first call. Returns ``TRUE``, if the operation was successful?

.. cpp:function:: virtual void CSoundBaseDevice::Cancel (void)

	Cancels the transmission of sound data. Cancel takes effect after a short delay.

.. cpp:function:: virtual boolean CSoundBaseDevice::IsActive (void) const

	Returns ``TRUE``, if sound data transmission is currently running? This method can be called on any CPU core.

Output queue
""""""""""""

These methods are used to output sound using a write queue. They are not used, if ``GetChunk()`` is overwritten instead.

.. cpp:function:: boolean CSoundBaseDevice::AllocateQueue (unsigned nSizeMsecs)

	Allocates the queue used for ``Write()``. ``nSizeMsecs`` is the size of the queue in milliseconds duration of the stream.

.. cpp:function:: boolean CSoundBaseDevice::AllocateQueueFrames (unsigned nSizeFrames)

	Allocates the queue used for ``Write()``. ``nSizeFrames`` is the size of the queue in number of audio frames.

.. cpp:function:: void CSoundBaseDevice::SetWriteFormat (TSoundFormat Format, unsigned nChannels = 2)

	Sets the format of sound data provided to ``Write()`` to ``Format``. ``nChannels`` is the number of logical channels and can be 1 to 32. If an audio device supports more hardware channels than the given value, the remaining channels will send null level. If an audio device supports less hardware channels than the given value, the remaining written audio samples will be ignored. The following (interleaved little endian) write formats are allowed:

	* SoundFormatUnsigned8
	* SoundFormatSigned16
	* SoundFormatSigned24 (occupies 3 bytes)
	* SoundFormatSigned24_32 (occupies 4 bytes)

.. cpp:function:: int CSoundBaseDevice::Write (const void *pBuffer, size_t nCount)

	Appends audio samples from ``pBuffer`` to the output queue. ``nCount`` is the size of the buffer in bytes and must be a multiple of the frame size. Returns the number of bytes from the buffer, which have to be consumed successfully. This value may be smaller than ``nCount``, in which case some frames have been ignored. This method can be called on any CPU core.

.. cpp:function:: unsigned CSoundBaseDevice::GetQueueSizeFrames (void)

	Returns the output queue size in number of frames. This method can be called on any CPU core.

.. cpp:function:: unsigned CSoundBaseDevice::GetQueueFramesAvail (void)

	Returns the number of frames currently available in the output queue, which are waiting to be sent to the hardware interface. This method can be called on any CPU core.

.. cpp:function:: void CSoundBaseDevice::RegisterNeedDataCallback (TSoundDataCallback *pCallback, void *pParam)

	Registers the callback function ``pCallback``, which is called, when more sound data is needed, which means that at least half of the queue is empty. ``pParam`` is a user parameter to be handed over to the callback. The callback function has the following prototype:

.. c:type:: void TSoundDataCallback (void *pParam)

	``pParam`` is the user parameter, which has been handed over to ``RegisterNeedDataCallback()``.

Input queue
"""""""""""

These methods are used to input sound data using a read queue. They are not used, if ``PutChunk()`` is overwritten instead.

.. cpp:function:: boolean CSoundBaseDevice::AllocateReadQueue (unsigned nSizeMsecs)

	Allocates the queue used for ``Read()``. ``nSizeMsecs`` is the size of the queue in milliseconds duration of the stream.

.. cpp:function:: boolean CSoundBaseDevice::AllocateReadQueueFrames (unsigned nSizeFrames)

	Allocates the queue used for ``Read()``. ``nSizeFrames`` is the size of the queue in number of audio frames.

.. cpp:function:: void CSoundBaseDevice::SetReadFormat (TSoundFormat Format, unsigned nChannels = 2, boolean bLeftChannel = TRUE)

	Sets the format of sound data returned from ``Read()`` to ``Format``. ``nChannels`` is the number of logical channels and can be 1 to 32. If an audio device supports more hardware channels than the given value, the remaining channels will be ignored. If an audio device supports less hardware channels than the given value, the remaining read audio samples will return null level. If ``bLeftChannel`` is ``TRUE``, ``Read()`` returns the left channel, if ``nChannels == 1``. The following (interleaved little endian) read formats are allowed:

	* SoundFormatUnsigned8
	* SoundFormatSigned16
	* SoundFormatSigned24 (occupies 3 bytes)
	* SoundFormatSigned24_32 (occupies 4 bytes)

.. cpp:function:: int CSoundBaseDevice::Read (void *pBuffer, size_t nCount)

	Moves up to ``nCount`` bytes of audio samples into ``pBuffer`` from the input queue and returns the number of returned bytes, which is a multiple of the frame size in any case, or 0 if no data is available. ``nCount`` must be a multiple of the frame size. This method can be called on any CPU core.

.. cpp:function:: unsigned CSoundBaseDevice::GetReadQueueSizeFrames (void)

	Returns the input queue size in number of frames. This method can be called on any CPU core.

.. cpp:function:: unsigned CSoundBaseDevice::GetReadQueueFramesAvail (void)

	Returns the number of frames currently available in the input queue, which are waiting to be read by the application. This method can be called on any CPU core.

.. cpp:function:: void CSoundBaseDevice::RegisterHaveDataCallback (TSoundDataCallback *pCallback, void *pParam)

	Registers the callback function ``pCallback``, which is called, when enough sound data is available for ``Read()``, which means that at least half of the queue is full. ``pParam`` is a user parameter to be handed over to the callback. The callback function has this prototype: :c:func:`TSoundDataCallback`.

Alternate interface
"""""""""""""""""""

Optionally an application can bypass the output and/or input queues and can directly provide/consume the audio samples to/from a buffer, which is handed over to the callback methods ``GetChunk()`` and/or ``PutChunk()``. This/These method(s) have to be overwritten to use the alternate interface. The format of the samples depends on the used hardware/software interface:

==============	======================	====================================================
Interface	Format			Remarks
==============	======================	====================================================
PWM		SoundFormatUnsigned32	range max. depends on sample rate and PWM clock rate
I2S		SoundFormatSigned24_32	occupies 4 bytes
HDMI		SoundFormatIEC958	special frame format (S/PDIF)
USB		SoundFormatSigned16 or
		SoundFormatSigned24
VCHIQ		SoundFormatSigned16	occupies 4 bytes
==============	======================	====================================================

.. cpp:function:: virtual int CSoundBaseDevice::GetRangeMin (void) const
.. cpp:function:: virtual int CSoundBaseDevice::GetRangeMax (void) const

	Return the minimum/maximum value of one sample. These methods can be called on any CPU core.

.. cpp:function:: boolean CSoundBaseDevice::AreChannelsSwapped (void) const

	Returns ``TRUE``, if the application has to write the right channel first into buffer in ``GetChunk()``.

.. cpp:function:: virtual unsigned CSoundBaseDevice::GetChunk (s16 *pBuffer, unsigned nChunkSize)
.. cpp:function:: virtual unsigned CSoundBaseDevice::GetChunk (u32 *pBuffer, unsigned nChunkSize)

	You may override one of these methods to provide the sound samples. The first method is used for the VCHIQ interface and the USB interface, the second for all other interfaces (including USB for 24-bit resolution, each sample occupies 3 bytes here). ``pBuffer`` is a pointer to the buffer, where the samples have to be placed. ``nChunkSize`` is the size of the buffer in words. Returns the number of words written to the buffer, which is normally ``nChunkSize``, or 0 to stop the transfer. Each sample consists of ``GetHWTXChannels()`` words, where each word must be between ``GetRangeMin()`` and ``GetRangeMax()``. The HDMI interface requires a special frame format here, which can be applied using ``ConvertIEC958Sample()``.

.. cpp:function:: virtual void CSoundBaseDevice::PutChunk (const s16 *pBuffer, unsigned nChunkSize)
.. cpp:function:: virtual void CSoundBaseDevice::PutChunk (const u32 *pBuffer, unsigned nChunkSize)

	You may override this method to consume the received sound samples. The first method is used for the USB interface, the second for I2S (or USB for 24-bit resolution, each sample occupies 3 bytes here). ``pBuffer`` is a pointer to the buffer, where the samples have been placed. ``nChunkSize`` is the size of the buffer in words. Each sample consists of ``GetHWRXChannels()`` words.

.. cpp:function:: u32 CSoundBaseDevice::ConvertIEC958Sample (u32 nSample, unsigned nFrame)

	This method can be called from ``GetChunk()`` to apply the framing on IEC958 (S/PDIF) samples. ``nSample`` is a 24-bit signed sample value as ``u32``, where upper bits don't care. ``nFrame`` is the number of the IEC958 frame, this sample belongs to (0..191).

.. _Sound controller:

Sound controller
""""""""""""""""

A sound device can optionally provide a sound controller, which offers the following functions:

* Return information about the output and input properties of the device.
* Enable a specific jack of sound devices with multiple connectors or connector configurations.
* Disable a specific jack (with multi-jack operation only).
* Return information about audio controls (e.g. volume), which influence the output or input of sound.
* Set a specific value of an audio control (e.g. mute off/on).

.. cpp:function:: virtual CSoundController *CSoundBaseDevice::GetController (void)

	This method returns a pointer to the sound controller object of a sound device or ``nullptr``, if a sound controller is not supported or not (yet) available. The sound controller is only available, after :cpp:func:`CSoundBaseDevice::Start()` has been called for the sound device, and only while the device is active.

.. code-block:: cpp

	#include <circle/sound/soundcontroller.h>

.. cpp:class:: CSoundController

	This class represents the interface of a sound controller to an application. A pointer to a sound controller object can be fetched by calling :cpp:func:`CSoundBaseDevice::GetController()` on a created driver object for a sound device.

	Please note that the enum values, given below, are valid in the name space of the class ``CSoundController`` only, so you have to use the prefix ``CSoundController::`` on them.

.. important::

	Methods of the sound controller can be called only at ``TASK_LEVEL``.

.. cpp:function:: u32 CSoundController::GetOutputProperties (void) const
.. cpp:function:: u32 CSoundController::GetInputProperties (void) const

	Returns a bit-mask with values defined in :cpp:enum:`CSoundController::TProperty` or'ed together. The first method returns the properties of the output direction of the controlled sound device, the second method the properties of the input direction of the device.

.. cpp:enum:: CSoundController::TProperty

	The follwing values are defined:

	* PropertyDirectionSupported (Is the respective output / input direction supported?)
	* PropertyMultiJackOperation (Is it possible to enable multiple jacks at once for this direction?)

.. cpp:function:: boolean CSoundController::EnableJack (TJack Jack)

	Enables the specified ``Jack`` of the sound device. Returns ``TRUE`` on success. This method can be called multiple times for different jacks, if ``PropertyMultiJackOperation`` is available. Otherwise a call to this method automatically disables the previously active jack.

.. cpp:enum:: CSoundController::TJack

	Output jacks:

	* JackDefaultOut (default or currently active output jack)
	* JackLineOut
	* JackSpeaker
	* JackHeadphone
	* JackHDMI
	* JackSPDIFOut

	Input jacks:

	* JackDefaultIn (default or currently active input jack)
	* JackLineIn
	* JackMicrophone

.. cpp:function:: boolean CSoundController::DisableJack (TJack Jack)

	Disables a specific ``Jack`` of the sound device. Returns ``TRUE`` on success. This method always fails without ``PropertyMultiJackOperation`` available.

.. cpp:function:: const CSoundController::TControlInfo CSoundController::GetControlInfo (TControl Control, TJack Jack, TChannel Channel) const

	Returns information about a specific ``Control`` of a specific ``Jack`` and ``Channel`` of a sound device. Please note that a control may be supported for ``ChannelAll``, but not for ``ChannelLeft`` and ``ChannelRight``.

.. cpp:enum:: CSoundController::TControl

	* ControlMute (mute value is 0 (disable) or 1 (enable))
	* ControlVolume (volume value in dB)

.. cpp:enum:: CSoundController::TChannel

	* ChannelAll (all channels)
	* ChannelLeft = Channel1
	* ChannelRight = Channel2
	* Channel3
	* ...
	* Channel32

.. cpp:struct:: CSoundController::TControlInfo

.. code:: c++

	struct TControlInfo
	{
		boolean	Supported;	// Is control supported?
		int	RangeMin;	// Minimum allowed value
		int	RangeMax;	// Maximum allowed value
	};

.. cpp:function:: boolean CSoundController::SetControl (TControl Control, TJack Jack, TChannel Channel, int nValue)

	Sets the value ``nValue`` of a specific ``Control`` of a specific ``Jack`` and affected ``Channel`` of a sound device. Returns ``TRUE`` on success.

CPWMSoundBaseDevice
^^^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/sound/pwmsoundbasedevice.h>

.. cpp:class:: CPWMSoundBaseDevice : public CSoundBaseDevice

	This class is a driver for the PWM sound interface. The generated sound is available via the 3.5" headphone jack, provided by most Raspberry Pi models. Most of the methods, available for using this class, are provided by the base class :cpp:class:`CSoundBaseDevice`. Only the constructor is specific to this class. This device has the name ``"sndpwm"`` in the device name service (character device).

.. note::

	On the Raspberry Pi Zero, which does not have a headphone jack, the output from the PWM sound interface can be used via the GPIO header. You have to define the system option ``USE_PWM_AUDIO_ON_ZERO`` for this purpose. See the file `include/circle/sysconfig.h <https://github.com/rsta2/circle/blob/master/include/circle/sysconfig.h>`_ for details!

.. cpp:function:: CPWMSoundBaseDevice::CPWMSoundBaseDevice (CInterruptSystem *pInterrupt, unsigned nSampleRate = 44100, unsigned nChunkSize = 2048)

	Constructs an instance of this class. There can be only one. ``pInterrupt`` is a pointer to the interrupt system object. ``nSampleRate`` is the sample rate in Hz. ``nChunkSize`` is twice the number of samples (words) to be handled with one call to ``GetChunk()`` (one word per stereo channel). Decreasing this value also decreases the latency on this interface, but increases the IRQ load on CPU core 0.

CPWMSoundDevice
^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/sound/pwmsounddevice.h>

.. cpp:class:: CPWMSoundDevice : public CPWMSoundBaseDevice

	This class is a PWM playback device for sound data, which is available in main memory. It extents the class :cpp:class:`CPWMSoundBaseDevice`, but has its own interface. The sample rate is fixed at 44100 Hz.

.. cpp:function:: CPWMSoundDevice::CPWMSoundDevice (CInterruptSystem *pInterrupt)

	Constructs an instance of this class. There can be only one. ``pInterrupt`` is a pointer to the interrupt system object.

.. cpp:function:: void CPWMSoundDevice::Playback (void *pSoundData, unsigned nSamples, unsigned nChannels, unsigned  nBitsPerSample)

	Starts playback of the sound data at ``pSoundData`` via the PWM sound device. ``nSamples`` is the number of samples, where for Stereo the L/R samples are count as one. ``nChannels`` is 1  for Mono or 2  for Stereo. ``nBitsPerSample`` is 8 (unsigned char sound data) or 16 (signed short sound data).

.. cpp:function:: boolean CPWMSoundDevice::PlaybackActive (void) const

	Returns ``TRUE``, while the playback is active.

.. cpp:function:: void CPWMSoundDevice::CancelPlayback (void)

	Cancels the playback. The operation takes affect with a short delay, after which ``PlaybackActive()`` returns ``FALSE``.

CI2SSoundBaseDevice
^^^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/sound/i2ssoundbasedevice.h>

.. cpp:class:: CI2SSoundBaseDevice : public CSoundBaseDevice

	This class is a driver for the I2S sound interface. The generated sound is available via the GPIO header in the format: two 32-bit wide channels with 24-bit signed data. Most of the methods, available for using this class, are provided by the base class :cpp:class:`CSoundBaseDevice`. Only the constructor is specific to this class. This device has the name ``"sndi2s"`` in the device name service (character device).

.. note::

	The following GPIO pins have to be connected (SoC numbers, not header positions):

	==============	==============	===============	==================================
	Name		Pin number	On early models	Description
	==============	==============	===============	==================================
	PCM_CLK		GPIO18		GPIO28		Bit clock (output or input)
	PCM_FS		GPIO19		GPIO29		Frame clock (output or input)
	PCM_DIN		GPIO20		GPIO30		Data input (not for TX only mode)
	PCM_DOUT	GPIO21		GPIO31		Data output (not for RX only mode)
	==============	==============	===============	==================================

	The clock pins are outputs in master mode, or inputs in slave mode. On early models the signals are exposed on the separate P5 header.

.. note::

	This driver class supports several I2S interfaces. Some interfaces require an additional I2C connection to work. The following interfaces are known to work:

	* pHAT DAC (with PCM5102A DAC)
	* PiFi DAC+ v2.0 (with PCM5122 DAC)
	* `Adafruit I2S Audio Bonnet <https://www.adafruit.com/product/4037>`_ (with UDA1334A DAC)
	* `Adafruit I2S 3W Class D Amplifier Breakout <https://www.adafruit.com/product/3006>`_ (with MAX98357A DAC)
	* `Waveshare WM8960 Audio HAT <https://www.waveshare.com/wm8960-audio-hat.htm>`_ (with WM8960 DAC)

.. cpp:function:: CI2SSoundBaseDevice::CI2SSoundBaseDevice (CInterruptSystem *pInterrupt, unsigned nSampleRate = 192000, unsigned nChunkSize = 8192, boolean bSlave = FALSE, CI2CMaster *pI2CMaster = 0, u8 ucI2CAddress = 0, TDeviceMode DeviceMode  = DeviceModeTXOnly)

	Constructs an instance of this class. There can be only one. ``pInterrupt`` is  a pointer to the interrupt system object. ``nSampleRate`` is the sample rate in Hz. ``nChunkSize`` is twice the number of samples (words) to be handled with one call to ``GetChunk()`` (one word per stereo channel). Decreasing this value also decreases the latency on this interface, but increases the IRQ load on CPU core 0.

	``bSlave`` enables the slave mode (PCM clock and FS clock are inputs). ``pI2CMaster`` is a pointer to an I2C master object (0 if no I2C DAC initialization is required). ``ucI2CAddress`` is the I2C slave address of the DAC (0 for auto probing the addresses 0x4C, 0x4D and 0x1A). ``DeviceMode`` selects, which transfer direction will be used, with these supported values:

	* DeviceModeTXOnly (output)
	* DeviceModeRXOnly (input)
	* DeviceModeTXRX (output and input)

CUSBSoundBaseDevice
^^^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/sound/usbsoundbasedevice.h>

.. cpp:class:: CUSBSoundBaseDevice : public CSoundBaseDevice

	This class is a driver for USB audio streaming devices. Most of the methods, available for using this class, are provided by the base class :cpp:class:`CSoundBaseDevice`. Only the constructor is specific to this class. The first device has the name ``"sndusb"`` in the device name service (character device). If there are multiple instances of this class, the second instance has the name ``"sndusb1"`` on so on.

.. important::

	The support for USB audio streaming devices is only available on the Raspberry Pi 4, 400 and Compute Module 4.

.. note::

	Circle supports USB audio streaming devices with up to 32 PCM channels and 16-bit (default) or 24-bit resolution. For the latter the option ``soundopt=24`` has to be specified in the file *cmdline.txt*. The number of channels has to be selected with the option `usbsoundchannels=TX,RX <https://github.com/rsta2/circle/blob/master/doc/cmdline.txt#L37>`_ in the same file.

.. cpp:function:: CUSBSoundBaseDevice::CUSBSoundBaseDevice (unsigned nSampleRate = 48000, TDeviceMode DeviceMode = DeviceModeTXOnly, unsigned nDevice = 0)

	Constructs an instance of this class. ``nSampleRate`` is the sample rate in Hz. The selected value must be supported by the attached USB audio streaming device (48000 should work with most devices). ``DeviceMode`` selects, which transfer direction will be used, with these supported values:

	* DeviceModeTXOnly (output)
	* DeviceModeRXOnly (input)
	* DeviceModeTXRX (output and input)

	Theoretically there may be multiple instances of this class at once. ``nDevice`` selects the attached USB audio streaming device to be accessed (0 is the first one found in USB device enumeration).

.. important::

	The class ``CUSBSoundBaseDevice`` must be instantiated, when the USB host controller is initialized already. Therefore it cannot be a class member of the class ``CKernel``. Use a pointer to the driver object instead and create it with the ``new`` operator.

CHDMISoundBaseDevice
^^^^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/sound/hdmisoundbasedevice.h>

.. cpp:class:: CHDMISoundBaseDevice : public CSoundBaseDevice

	This class is a driver for HDMI displays with audio support. It directly accesses the hardware and does not require :ref:`Multitasking` support and the :ref:`VCHIQ driver` in the system. Most of the methods, available for using this class, are provided by the base class :cpp:class:`CSoundBaseDevice`. This device has the name ``"sndhdmi"`` in the device name service (character device).

.. note::

	This driver supports only two channels (Stereo).

	This driver does not support HDMI1 on the Raspberry Pi 4 and 400 (HDMI0 only).

	This driver supports a DMA and a polling mode. The latter is intended for very time critical and cache-sensitive applications, which cannot use interrupts.

.. note::

	In Circle releases before 44.5, this driver swapped the channels of the Stereo signal. This has been corrected in this and later releases.

.. cpp:function:: CHDMISoundBaseDevice::CHDMISoundBaseDevice (CInterruptSystem *pInterrupt, unsigned nSampleRate = 48000, unsigned nChunkSize = 384 * 10)

	Constructs an instance of this class to work in DMA mode. There can be only one. ``pInterrupt`` is  a pointer to the interrupt system object. ``nSampleRate`` is the sample rate in Hz. ``nChunkSize`` is twice the number of samples (words) to be handled with one call to ``GetChunk()`` (one word per stereo channel, must be a multiple of 384). Decreasing this value also decreases the latency on this interface, but increases the IRQ load on CPU core 0.

.. cpp:function:: CHDMISoundBaseDevice::CHDMISoundBaseDevice (unsigned nSampleRate = 48000)

	Constructs an instance of this class to work in polling mode.  There can be only one. ``nSampleRate`` is the sample rate in Hz.

.. cpp:function:: boolean CHDMISoundBaseDevice::IsWritable (void)

	Returns if the data FIFO has room for at least one sample to be written? This method can be called in polling mode only.

.. cpp:function:: void CHDMISoundBaseDevice::WriteSample (s32 nSample)

	Writes one sample to the data FIFO. ``nSample`` is the 24-bit signed sample to be written. This method can be called in polling mode only and only, when :cpp:func:`IsWritable()` returned ``TRUE`` before. Must be called twice for each frame (for left and right channel).

CVCHIQSoundBaseDevice
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <vc4/sound/vchiqsoundbasedevice.h>

.. cpp:class:: CVCHIQSoundBaseDevice : public CSoundBaseDevice

	This class provides low-level access to the VCHIQ sound service, which is able to output sound via HDMI displays with audio support, or via the 3.5" headphone jack of Raspberry Pi models, which have it. This class requires, that the :ref:`Multitasking` support and the :ref:`VCHIQ driver` are available in the system. Most of the methods, available for using this class, are provided by the base class :cpp:class:`CSoundBaseDevice`. This class description covers only the methods, which are specific to this class. This device has the name ``"sndvchiq"`` in the device name service (character device).

.. cpp:function:: CVCHIQSoundBaseDevice::CVCHIQSoundBaseDevice (CVCHIQDevice *pVCHIQDevice, unsigned nSampleRate = 44100, unsigned nChunkSize  = 4000, TVCHIQSoundDestination Destination = VCHIQSoundDestinationAuto)

	Constructs an instance of this class. There can be only one. ``pVCHIQDevice`` is a pointer to the VCHIQ interface device. ``nSampleRate`` is the sample rate in Hz (44100..48000). ``nChunkSize`` is the number of samples transferred at once. ``Destination`` is the target device, the sound data is sent to (detected automatically, if equal to ``VCHIQSoundDestinationAuto``), with these possible values:

.. c:enum:: TVCHIQSoundDestination

	* VCHIQSoundDestinationAuto
	* VCHIQSoundDestinationHeadphones
	* VCHIQSoundDestinationHDMI
	* VCHIQSoundDestinationUnknown

.. cpp:function:: void CVCHIQSoundBaseDevice::SetControl (int nVolume, TVCHIQSoundDestination Destination = VCHIQSoundDestinationUnknown)

	Sets the output volume to ``nVolume`` (-10000..400, in 1/100 dB) and optionally the target device to ``Destination`` (not modified, if equal to ``VCHIQSoundDestinationUnknown``). This method can be called, while the sound data transmission is running. The following macros are defined for specifying the volume:

.. c:macro:: VCHIQ_SOUND_VOLUME_MIN
.. c:macro:: VCHIQ_SOUND_VOLUME_DEFAULT
.. c:macro:: VCHIQ_SOUND_VOLUME_MAX

.. note::

	The :ref:`Sound controller` provides a more generic solution for setting controls of a sound device.

CVCHIQSoundDevice
^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <vc4/sound/vchiqsounddevice.h>

.. cpp:class:: CVCHIQSoundDevice : private CVCHIQSoundBaseDevice

	This class is a VCHIQ playback device for sound data, which is available in main memory. It extents the class :cpp:class:`CVCHIQSoundBaseDevice`, but has its own interface. The sample rate is fixed at 44100 Hz.

.. cpp:function:: CVCHIQSoundDevice::CVCHIQSoundDevice (CVCHIQDevice *pVCHIQDevice, TVCHIQSoundDestination Destination = VCHIQSoundDestinationAuto)

	Constructs an instance of this class. There can be only one. ``pVCHIQDevice`` is a pointer to the VCHIQ interface device. ``Destination`` is the target device, the sound data is sent to (see :c:enum:`TVCHIQSoundDestination` for the available options).

.. cpp:function:: boolean CVCHIQSoundDevice::Playback (void *pSoundData, unsigned nSamples, unsigned nChannels, unsigned nBitsPerSample)

	Starts playback of the sound data at ``pSoundData`` via the VCHIQ sound device. ``nSamples`` is the number of samples, where for Stereo the L/R samples are count as one. ``nChannels`` is 1  for Mono or 2  for Stereo. ``nBitsPerSample`` is 8 (unsigned char sound data) or 16 (signed short sound data). Returns ``TRUE`` on success.

.. cpp:function:: boolean CVCHIQSoundDevice::PlaybackActive (void) const

	Returns ``TRUE``, while the playback is active.

.. cpp:function:: void CVCHIQSoundDevice::CancelPlayback (void)

	Cancels the playback. The operation takes affect with a short delay, after which ``PlaybackActive()`` returns ``FALSE``.

.. cpp:function:: void CVCHIQSoundDevice::SetControl (int nVolume, TVCHIQSoundDestination Destination = VCHIQSoundDestinationUnknown)

	See :cpp:func:`CVCHIQSoundBaseDevice::SetControl()`.

CUSBMIDIDevice
^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/usb/usbmidi.h>

.. cpp:class:: CUSBMIDIDevice : public CUSBFunction

	This class is a driver for USB Audio Class MIDI 1.0 devices. An instance of this class is automatically created, when a compatible device is found in the USB device enumeration process. Therefore only the class methods needed to use an USB MIDI device by an application are described here, not the methods used for initialization. This device has the name ``"umidiN"`` (N >= 1) in the device name service (character device).

.. note::

	See the `Universal Serial Bus Device Class Definition for MIDI Devices, Release 1.0 <https://usb.org/document-library/usb-midi-devices-10>`_ for information about USB MIDI packets and virtual MIDI cables!

.. cpp:function:: void CUSBMIDIDevice::RegisterPacketHandler (TMIDIPacketHandler *pPacketHandler)

	Registers a callback function, which is called, when a MIDI packet arrives. ``pPacketHandler`` is a pointer to the function, which has the following prototype:

.. c:type:: void TMIDIPacketHandler (unsigned nCable, u8 *pPacket, unsigned nLength)

	``nCable`` is the number of the virtual MIDI cable (0..15). ``pPacket`` is a pointer to one received MIDI packet. ``nLength`` is the number of valid bytes in the packet (1..3).

.. cpp:function:: void CUSBMIDIDevice::RegisterPacketHandler (TMIDIPacketHandlerEx *pPacketHandler, void *pParam)

	Alternative version of ``RegisterPacketHandler()``, which gets an additional user parameter, which is handed over to this callback function:

.. c:type:: void TMIDIPacketHandlerEx (unsigned nCable, u8 *pPacket, unsigned nLength, unsigned nDevice, void *pParam)

.. cpp:function:: boolean CUSBMIDIDevice::SendEventPackets (const u8 *pData, unsigned nLength)

	Sends one or more packets in the encoded USB MIDI event packet format. ``pData`` is a pointer to the packet buffer. ``nLength`` is the length of the packet buffer in bytes, which must be a multiple of 4. Returns ``TRUE``, if the operation has been successful. This function fails, if ``nLength`` is not a multiple of 4 or the send function is not supported. The format of the USB MIDI event packets is not validated.

.. cpp:function:: boolean CUSBMIDIDevice::SendPlainMIDI (unsigned nCable, const u8 *pData, unsigned nLength)

	Sends one or more messages in plain MIDI message format. ``nCable`` is the number of the virtual MIDI cable (0..15). ``pData`` is a pointer to the message buffer. ``nLength`` is the length of the message buffer in bytes. Returns ``TRUE``, if the operation has been successful. This function fails, if the message format is invalid or the send function is not supported.

.. cpp:function:: void CUSBMIDIDevice::SetAllSoundOffOnUSBError (boolean bEnable)

	If this method has been called with ``bEnable`` equal to ``TRUE``, the driver generates MIDI Control Change "All Sound Off" (120) messages for each MIDI channel (1-16) on MIDI cable 0, when an USB error is detected by the driver.
