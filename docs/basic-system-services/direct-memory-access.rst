Direct Memory Access (DMA)
~~~~~~~~~~~~~~~~~~~~~~~~~~

Circle supports Direct Memory Access (DMA) using the platform DMA controller of the Raspberry Pi. This is implemented in the class ``CDMAChannel``.

CDMAChannel
^^^^^^^^^^^

.. code-block:: c++

	#include <circle/dmachannel.h>

.. cpp:class:: CDMAChannel

.. cpp:function:: CDMAChannel::CDMAChannel (unsigned nChannel, CInterruptSystem *pInterruptSystem = 0)

	Creates an instance of  ``CDMAChannel`` and allocates a channel of the platform DMA controller. ``nChannel`` must be ``DMA_CHANNEL_NORMAL`` (normal DMA engine), ``DMA_CHANNEL_LITE`` (lite (or normal) DMA engine), ``DMA_CHANNEL_EXTENDED`` ("large address" DMA4 engine, on Raspberry Pi 4 only) or an explicit channel number (0-15). ``pInterruptSystem`` is a pointer to the instance of ``CInterruptSystem`` and is only needed for interrupt operation.

.. cpp:function:: void CDMAChannel::SetupMemCopy (void *pDestination, const void *pSource, size_t nLength, unsigned nBurstLength = 0, boolean bCached = TRUE)

	Setup a DMA memory copy transfer from ``pSource`` to ``pDestination`` with length ``nLength``. ``nBurstLength`` > 0 increases the speed, but may congest the system bus. ``bCached`` determines, if the source and destination address ranges are in cached memory.

.. cpp:function:: void CDMAChannel::SetupIORead (void *pDestination, u32 nIOAddress, size_t nLength, TDREQ DREQ)

	Setup a DMA read transfer from the I/O port ``nIOAddress`` to ``pDestination`` with length ``nLength``. ``DREQ`` paces the transfer from these devices:

* DREQSourceNone (no wait)
* DREQSourceEMMC
* DREQSourcePCMRX
* DREQSourceSMI
* DREQSourceSPIRX
* DREQSourceUARTRX

.. cpp:function:: void CDMAChannel::SetupIOWrite (u32 nIOAddress, const void *pSource, size_t nLength, TDREQ DREQ)

	Setup a DMA write transfer to the I/O port ``nIOAddress`` from ``pSource`` with length ``nLength``. ``DREQ`` paces the transfer to these devices:

* DREQSourceNone (no wait)
* DREQSourceEMMC
* DREQSourcePCMTX
* DREQSourcePWM
* DREQSourcePWM1 (on Raspberry Pi 4 only)
* DREQSourceSMI
* DREQSourceSPITX
* DREQSourceUARTTX

.. cpp:function:: void CDMAChannel::SetupMemCopy2D (void *pDestination, const void *pSource, size_t nBlockLength, unsigned nBlockCount, size_t nBlockStride, unsigned nBurstLength = 0)

	Setup a 2D DMA memory copy transfer of ``nBlockCount`` blocks of ``nBlockLength`` length from ``pSource`` to ``pDestination``. Skip ``nBlockStride`` bytes after each block on destination. Source is continuous. The destination cache, if any, is not touched. ``nBurstLength`` > 0 increases speed, but may congest the system bus. This method can be used to copy data to the framebuffer and is not supported with ``DMA_CHANNEL_LITE``.

.. cpp:function:: void CDMAChannel::SetCompletionRoutine (TDMACompletionRoutine *pRoutine, void *pParam)

	Sets a DMA completion routine for interrupt operation. ``pRoutine`` is called, when the transfer is completed. ``pParam`` is a user parameter, which is handed over to the completion routine. ``TDMACompletionRoutine`` has the following prototype:

.. code-block:: c++

	void TDMACompletionRoutine (unsigned nChannel, boolean bStatus, void *pParam);

``nChannel`` is the channel number. ``bStatus`` is ``TRUE``, if the transfer completed successfully.

.. cpp:function:: void CDMAChannel::Start (void)

	Starts the DMA transfer, which has been setup before.

.. cpp:function:: boolean CDMAChannel::Wait (void)

	Waits for the completion of the DMA transfer (for synchronous non-interrupt operation without completion routine). Returns ``TRUE``, if the transfer was successful.

.. cpp:function:: boolean CDMAChannel::GetStatus (void)

	Returns ``TRUE``, if the last completed transfer was successful.

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
