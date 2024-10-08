Direct Memory Access (DMA)
~~~~~~~~~~~~~~~~~~~~~~~~~~

Circle supports Direct Memory Access (DMA) using the platform DMA controller of the Raspberry Pi. This is implemented in the class ``CDMAChannel``. The Raspberry Pi 5 has an additional DMA controller in the RP1 southbridge, which is controlled by the class ``CDMAChannelRP1``.

CDMAChannel
^^^^^^^^^^^

.. code-block:: c++

	#include <circle/dmachannel.h>

.. cpp:class:: CDMAChannel

.. cpp:function:: CDMAChannel::CDMAChannel (unsigned nChannel, CInterruptSystem *pInterruptSystem = 0)

	Creates an instance of  ``CDMAChannel`` and allocates a channel of the platform DMA controller. ``nChannel`` must be ``DMA_CHANNEL_NORMAL`` (normal DMA engine), ``DMA_CHANNEL_LITE`` (lite (or normal) DMA engine), ``DMA_CHANNEL_EXTENDED`` ("large address" DMA4 engine, on Raspberry Pi 4 and 5 only) or an explicit channel number (0-15). ``pInterruptSystem`` is a pointer to the instance of ``CInterruptSystem`` and is only needed for interrupt operation.

.. note::

	Currently only "large address" DMA4 engines are supported on the Raspberry Pi 5. Any dynamic DMA channel allocation (non-explicit channel number) will use one of these engines. The defined ``DREQSource*`` values are currently valid for the Raspberry Pi 1-4 and Zero only.

.. cpp:function:: void CDMAChannel::SetupMemCopy (void *pDestination, const void *pSource, size_t nLength, unsigned nBurstLength = 0, boolean bCached = TRUE)

	Setup a DMA memory copy transfer from ``pSource`` to ``pDestination`` with length ``nLength``. ``nBurstLength`` > 0 increases the speed, but may congest the system bus. ``bCached`` determines, if the source and destination address ranges are in cached memory.

.. cpp:function:: void CDMAChannel::SetupIORead (void *pDestination, uintptr nIOAddress, size_t nLength, TDREQ DREQ)

	Setup a DMA read transfer from the I/O port ``nIOAddress`` to ``pDestination`` with length ``nLength``. ``DREQ`` paces the transfer from these devices:

* DREQSourceNone (no wait)
* DREQSourceEMMC
* DREQSourcePCMRX
* DREQSourceSMI
* DREQSourceSPIRX
* DREQSourceUARTRX

.. cpp:function:: void CDMAChannel::SetupIOWrite (uintptr nIOAddress, const void *pSource, size_t nLength, TDREQ DREQ)

	Setup a DMA write transfer to the I/O port ``nIOAddress`` from ``pSource`` with length ``nLength``. ``DREQ`` paces the transfer to these devices:

* DREQSourceNone (no wait)
* DREQSourceEMMC
* DREQSourcePCMTX
* DREQSourcePWM
* DREQSourcePWM1 (on Raspberry Pi 4 only)
* DREQSourceSMI
* DREQSourceSPITX
* DREQSourceUARTTX

.. cpp:function:: void CDMAChannel::SetupCyclicIOWrite (uintptr ulIOAddress, const void *ppSources[], unsigned nBuffers, size_t ulLength, TDREQ DREQ)

	Setup a cyclic DMA write transfer to the I/O port ``ulIOAddress`` (ARM-side or bus address) for ``nBuffers`` concatenated DMA buffers (max. 4) at ``ppSources`` (pointer to array of pointers) with length ``ulLength`` bytes per buffer. ``DREQ`` paces the transfer (see :cpp:func:`CDMAChannel::SetupIOWrite` for the possible devices). The transfer starts from first buffer again, when last buffer has been sent.

.. cpp:function:: void CDMAChannel::SetupMemCopy2D (void *pDestination, const void *pSource, size_t nBlockLength, unsigned nBlockCount, size_t nBlockStride, unsigned nBurstLength = 0)

	Setup a 2D DMA memory copy transfer of ``nBlockCount`` blocks of ``nBlockLength`` length from ``pSource`` to ``pDestination``. Skip ``nBlockStride`` bytes after each block on destination. Source is continuous. The destination cache, if any, is not touched. ``nBurstLength`` > 0 increases speed, but may congest the system bus. This method can be used to copy data to the framebuffer and is not supported with ``DMA_CHANNEL_LITE``.

.. cpp:function:: void CDMAChannel::SetCompletionRoutine (TDMACompletionRoutine *pRoutine, void *pParam)

	Sets a DMA completion routine for interrupt operation. ``pRoutine`` is called, when the transfer is completed. ``pParam`` is a user parameter, which is handed over to the completion routine. ``TDMACompletionRoutine`` has the following prototype:

.. c:type:: void TDMACompletionRoutine (unsigned nChannel, unsigned nBuffer, boolean bStatus, void *pParam)

``nChannel`` is the channel number. ``nBuffer`` is the number of the cyclic buffer (always 0 for non-cyclic transfers). ``bStatus`` is ``TRUE``, if the transfer completed successfully.

.. cpp:function:: void CDMAChannel::Start (void)

	Starts the DMA transfer, which has been setup before.

.. cpp:function:: boolean CDMAChannel::Wait (void)

	Waits for the completion of the DMA transfer (for synchronous non-interrupt operation without completion routine). Returns ``TRUE``, if the transfer was successful.

.. cpp:function:: boolean CDMAChannel::GetStatus (void)

	Returns ``TRUE``, if the last completed transfer was successful.

CDMAChannelRP1
^^^^^^^^^^^^^^

.. code-block:: c++

	#include <circle/dmachannel-rp1.h>

.. cpp:class:: CDMAChannelRP1

	This class controls the RP1 DMA controller of the Raspberry Pi 5, which is normally used to transfer data between peripherals in the RP1 southbridge and the system memory.

.. cpp:function:: CDMAChannelRP1::CDMAChannelRP1 (unsigned nChannel, CInterruptSystem *pInterruptSystem)

	Creates an instance of ``CDMAChannelRP1`` for RP1 DMA channel ``nChannel`` (0-7). There is currently no dynamic channel allocation for RP1 DMA channels. ``pInterruptSystem`` is a pointer to the interrupt system object.

.. cpp:function:: void CDMAChannelRP1::SetupMemCopy (void *pDestination, const void *pSource, size_t ulLength, boolean bCached = TRUE)

	Setup a DMA memory copy transfer from ``pSource`` to ``pDestination`` with length ``nLength`` bytes. ``bCached`` determines, if the source and destination address ranges are in cached memory.

.. cpp:function:: void CDMAChannelRP1::SetupIORead (void *pDestination, uintptr ulIOAddress, size_t ulLength, TDREQ DREQ, unsigned nIORegWidth = 4)

	Setup a DMA read transfer from the I/O port ``ulIOAddress`` (ARM-side or bus address) to ``pDestination`` with length ``nLength`` bytes. ``nIORegWidth`` specifies the width of the accessed I/O register (1 or 4 bytes). ``DREQ`` paces the transfer from these devices:

	* DREQSourceNone (no wait)
	* DREQSourceSPI0RX
	* DREQSourceSPI0TX
	* DREQSourceSPI1RX
	* DREQSourceSPI1TX
	* DREQSourceSPI2RX
	* DREQSourceSPI2TX
	* DREQSourceSPI3RX
	* DREQSourceSPI3TX
	* DREQSourceSPI5RX
	* DREQSourceSPI5TX
	* DREQSourcePWM0
	* DREQSourceI2S0RX
	* DREQSourceI2S0TX
	* DREQSourceI2S1RX
	* DREQSourceI2S1TX

.. cpp:function:: void CDMAChannelRP1::SetupCyclicIORead (void *ppDestinations[], uintptr ulIOAddress, unsigned nBuffers, size_t ulLength, TDREQ DREQ, unsigned nIORegWidth = 4)

	Setup a cyclic DMA read transfer from the I/O port ``ulIOAddress`` (ARM-side or bus address) for ``nBuffers`` concatenated DMA buffers (max. 4) at ``ppDestinations`` (pointer to array of pointers) with length ``nLength`` bytes per buffer. ``nIORegWidth`` specifies the width of the accessed I/O register (1 or 4 bytes). ``DREQ`` paces the transfer (see :cpp:func:`CDMAChannelRP1::SetupIORead` for the possible devices). The transfer starts from first buffer again, when last buffer has been filled.

.. cpp:function:: void CDMAChannelRP1::SetupIOWrite (uintptr ulIOAddress, const void *pSource, size_t ulLength, TDREQ DREQ, unsigned nIORegWidth = 4)

	Setup a DMA write transfer to the I/O port ``ulIOAddress`` (ARM-side or bus address) from ``pSource`` with length ``nLength`` bytes. ``nIORegWidth`` specifies the width of the accessed I/O register (1 or 4 bytes). ``DREQ`` paces the transfer (see :cpp:func:`CDMAChannelRP1::SetupIORead` for the possible devices).

.. cpp:function:: void CDMAChannelRP1::SetupCyclicIOWrite (uintptr ulIOAddress, const void *ppSources[], unsigned nBuffers, size_t ulLength, TDREQ DREQ, unsigned nIORegWidth = 4)

	Setup a cyclic DMA write transfer to the I/O port ``ulIOAddress`` (ARM-side or bus address) for ``nBuffers`` concatenated DMA buffers (max. 4) at ``ppSources`` (pointer to array of pointers) with length ``nLength`` bytes per buffer. ``nIORegWidth`` specifies the width of the accessed I/O register (1 or 4 bytes). ``DREQ`` paces the transfer (see :cpp:func:`CDMAChannelRP1::SetupIORead` for the possible devices). The transfer starts from first buffer again, when last buffer has been sent.

.. cpp:function:: void CDMAChannelRP1::SetCompletionRoutine (TCompletionRoutine *pRoutine, void *pParam)

	Sets a DMA completion routine for interrupt operation. ``pRoutine`` is called, when a transfer is completed. ``pParam`` is a user parameter, which is handed over to the completion routine. For cyclic transfer the completion routine is called after each buffer again. ``TCompletionRoutine`` has the following prototype:

.. cpp:type:: void CDMAChannelRP1::TCompletionRoutine (unsigned nChannel, unsigned nBuffer, boolean bStatus, void *pParam)

	``nChannel`` is the index of the RP1 DMA channel (0-7). ``nBuffer`` is the index of the cyclic buffer (0-N, 0 if not cyclic). ``bStatus`` is ``TRUE`` for a successful transfer, ``FALSE`` on error. ``pParam`` is the user parameter, handed over to ``SetCompletionRoutine()``.

.. cpp:function:: void CDMAChannelRP1::Start (void)

	Starts the DMA transfer, which has been setup before.

.. cpp:function:: void CDMAChannelRP1::Cancel (void)

	Cancels a running DMA transfer and waits for its termination.

.. _dma-buffers:

DMA buffers
^^^^^^^^^^^

.. code-block:: c++

	#include <circle/synchronize.h>

.. c:macro:: DMA_BUFFER(type, name, num)

	Defines a buffer with ``name`` and ``num`` elements of ``type`` to be used for DMA transfers.

	See `doc/dma-buffer-requirements.txt <https://github.com/rsta2/circle/blob/master/doc/dma-buffer-requirements.txt>`_ for more information on DMA buffers.

Cache maintenance
^^^^^^^^^^^^^^^^^

.. code-block:: c++

	#include <circle/synchronize.h>

.. c:function:: void CleanAndInvalidateDataCacheRange (uintptr nAddress, size_t nLength)

	Cleans and invalidates a memory range in the data cache.
