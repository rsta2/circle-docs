Firmware access
~~~~~~~~~~~~~~~

In order for a device driver to make certain settings (e.g. size of the frame buffer), it sometimes needs to communicate with the firmware, which runs on the VPU co-processor. This may be necessary from application code too, if specific settings should be made, which are not supported by Circle. The firmware can be accessed using the `Mailbox property interface <https://github.com/raspberrypi/firmware/wiki/Mailbox-property-interface>`_. This is supported in Circle by the class ``CBcmPropertyTags``.

.. note::

	This `Mailbox property interface <https://github.com/raspberrypi/firmware/wiki/Mailbox-property-interface>`_ wiki article does not describe all supported functions. Information on more functions can only be retrieved from the Linux source code.

	The firmware of the Raspberry Pi 5 supports only a small subset of the functions, which are described in this wiki article (and some additional ones).

CBcmPropertyTags
^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/bcmpropertytags.h>

.. cpp:class:: CBcmPropertyTags

.. cpp:function:: boolean CBcmPropertyTags::GetTag (u32 nTagId, void *pTag, unsigned nTagSize, unsigned nRequestParmSize = 0)

	Makes a mailbox property call to the firmware with a single tag. ``nTagId`` is the tag identifier. The identifiers, used by Circle, are listed in the header file `circle/bcmpropertytags.h <https://github.com/rsta2/circle/blob/master/include/circle/bcmpropertytags.h>`_. ``pTag`` points to the tag structure and ``nTagSize`` is the size of this structure. This header file defines the tag structure for a number of mailbox property functions too. The parameter ``nRequestParmSize`` specifies the number of bytes in the tag structure, which are passed as input parameters to the firmware, where the ``TPropertyTag`` header does not count. This parameter may be zero for property tags, which do not pass input parameters to the firmware. ``GetTag()`` returns ``TRUE``, if the call succeeds.

.. cpp:function:: boolean CBcmPropertyTags::GetTags (void *pTags, unsigned nTagsSize)

	Makes a mailbox property call to the firmware with multiple tags at once. ``pTags`` points to the tags structure, which is a concatenation of multiple property tag structures. ``nTagsSize`` is the total size of this structure. ``GetTags()`` returns ``TRUE``, if the call succeeds.

Example
"""""""

The following code retrieves the current clock rate of the ARM CPU from the firmware with the ``GetTag()`` method:

.. code-block:: cpp

	#include <circle/bcmpropertytags.h>

	unsigned CMyClass::GetClockRate (void)
	{
		CBcmPropertyTags Tags;			// this class

		TPropertyTagClockRate TagClockRate;	// the tag structure

		TagClockRate.nClockId = CLOCK_ID_ARM;	// input parameter

		if (!Tags.GetTag (PROPTAG_GET_CLOCK_RATE,
				  &TagClockRate, sizeof TagClockRate,
				  sizeof TagClockRate.nClockId))
		{
			return 0;			// return 0 on failure
		}

		return TagClockRate.nRate;		// return the clock rate
	}

An example for using the ``GetTags()`` method is available in the `frame buffer driver <https://github.com/rsta2/circle/blob/master/lib/bcmframebuffer.cpp>`_.
