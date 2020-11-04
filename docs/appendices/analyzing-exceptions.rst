.. _analyzing-exceptions:

Analyzing exceptions
~~~~~~~~~~~~~~~~~~~~

This appendix explains, how system abort exceptions can be analyzed. The output from the program in section :ref:`a-more-complex-program` is used for this purpose. It looks like this:

.. code-block:: none

	logger: Circle 43.1 started on Raspberry Pi Zero W
	00:00:01.00 timer: SpeedFactor is 1.00
	00:00:01.00 kernel: An exception will occur after 15 seconds from now
	00:00:02.00 kernel: Time is 2
	00:00:03.00 kernel: Time is 3
	00:00:04.00 kernel: Time is 4
	...
	00:00:14.00 kernel: Time is 14
	00:00:15.00 kernel: Time is 15
	00:00:16.00 except: stack[7] is 0xEFBC
	00:00:16.00 except: stack[8] is 0xF04C
	00:00:16.00 except: stack[11] is 0x11304
	00:00:16.00 except: stack[13] is 0x113C4
	00:00:16.00 except: stack[23] is 0x11408
	00:00:16.00 except: stack[25] is 0x108E4
	00:00:16.00 except: stack[31] is 0xE834
	00:00:16.00 except: Prefetch abort (PC 0x500000, FSR 0xD, FAR 0x500000,
		SP 0x237F80, LR 0xEE7C, PSR 0x20000192)

If you want to detect the instruction, which caused the exception, you can open the file *kernel\*.lst* and search for the address in *PC* (Program Counter). Because this is an invalid address outside the kernel image, you will not find it here, but *LR* (Link Register) specifies the address, from where ``TimerHandler()`` had been called (0xEE7C). The respective address is located at the beginning of a line in *kernel\*.lst* with a trailing colon:

.. code-block:: none

        0000edd0 <CTimer::PollKernelTimers()>:
            edd0:	e92d41f0 	push	{r4, r5, r6, r7, r8, lr}
        ...
            ee64:	e3530000 	cmp	r3, #0
            ee68:	0a000011 	beq	eeb4 <CTimer::PollKernelTimers()+0xe4>
            ee6c:	e1a00004 	mov	r0, r4
            ee70:	e5942010 	ldr	r2, [r4, #16]
            ee74:	e594100c 	ldr	r1, [r4, #12]
            ee78:	e12fff33 	blx	r3
        ==> ee7c:	e1a00004 	mov	r0, r4
            ee80:	e3a01014 	mov	r1, #20
        ...

Thus ``TimerHandler()`` had been called by the instruction "blx r3", the preceeding instruction of the given address.

The several listed addresses from the *stack[]* allow to do a backtrace, but not every shown address needs to be valid. Just search for the addresses in the *kernel\*.lst* file, starting with the first one, and you will get the information, which function has been called from which other function (inside-out).
