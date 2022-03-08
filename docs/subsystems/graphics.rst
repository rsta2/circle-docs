Graphics
~~~~~~~~

Circle provides several options for implementing graphical user interfaces (GUI) and for generating pixel and vector graphics on an attached HDMI or composite TV display. These options are described in this section.

C2DGraphics
^^^^^^^^^^^

The class ``C2DGraphics`` is part of the Circle base library and can be used to generate pixel graphics on a frame buffer, which is provided by the class :cpp:class:`CBcmFrameBuffer`.

.. code-block:: cpp

	#include <circle/2dgraphics.h>

.. cpp:class:: C2DGraphics

	This class is a software graphics library with VSync and hardware-accelerated double buffering.

.. cpp:function:: C2DGraphics::C2DGraphics (unsigned nWidth, unsigned nHeight, boolean bVSync = TRUE, unsigned nDisplay = 0)

	Creates on instance of this class. ``nWidth`` is the screen width in pixels (0 to detect). ``nHeight`` is the screen height in pixels (0 to detect). Set ``bVSync`` to ``TRUE`` to enable VSync and HW double buffering. ``nDisplay`` is the zero-based display number (for Raspberry Pi 4).

.. cpp:function:: boolean C2DGraphics::Initialize (void)

	Initializes the screen. Returns ``TRUE`` on success.

.. cpp:function:: unsigned C2DGraphics::GetWidth (void) const

	Returns the screen width in pixels.

.. cpp:function:: unsigned C2DGraphics::GetHeight (void) const

	Returns the screen height in pixels.

.. cpp:function:: void C2DGraphics::ClearScreen (TScreenColor Color)

	Clears the screen. ``Color`` is the color used to clear the screen (see :cpp:func:`CScreenDevice::SetPixel()`).

.. cpp:function:: void C2DGraphics::DrawRect (unsigned nX, unsigned nY, unsigned nWidth, unsigned nHeight, TScreenColor Color)

	Draws a filled rectangle. ``nX`` is the start X coordinate. ``nY`` is the start Y coordinate. ``nWidth`` is the rectangle width. ``nHeight`` is the rectangle height. ``Color`` is the rectangle color.

.. cpp:function:: void C2DGraphics::DrawRectOutline (unsigned nX, unsigned nY, unsigned nWidth, unsigned nHeight, TScreenColor Color)

	Draws an unfilled rectangle (inner outline). ``nX`` is the start X coordinate. ``nY`` is the start Y coordinate. ``nWidth`` is the rectangle width. ``nHeight`` is the rectangle height. ``Color`` is the rectangle color.

.. cpp:function:: void C2DGraphics::DrawLine (unsigned nX1, unsigned nY1, unsigned nX2, unsigned nY2, TScreenColor Color)

	Draws a line. ``nX1`` is the start position X coordinate. ``nY1`` is the start position Y coordinate. ``nX2`` is the end position X coordinate. ``nY2`` is the end position Y coordinate. ``Color`` is the line color.

.. cpp:function:: void C2DGraphics::DrawCircle (unsigned nX, unsigned nY, unsigned nRadius, TScreenColor Color)

	Draws a filled circle. ``nX`` is the circle X coordinate. ``nY`` is the circle Y coordinate. ``nRadius`` is the circle radius. ``Color`` is the circle color.

.. cpp:function:: void C2DGraphics::DrawCircleOutline (unsigned nX, unsigned nY, unsigned nRadius, TScreenColor Color)

	Draws an unfilled circle (inner outline). ``nX`` is the circle X coordinate. ``nY`` is the circle Y coordinate. ``nRadius`` is the circle radius. ``Color`` is the circle color.

.. cpp:function:: void C2DGraphics::DrawImage (unsigned nX, unsigned nY, unsigned nWidth, unsigned nHeight, TScreenColor *PixelBuffer)

	Draws an image from a pixel buffer. ``nX`` is the image X coordinate. ``nY`` is the image Y coordinate. ``nWidth`` is the image width. ``nHeight`` is the image height. ``PixelBuffer`` is a pointer to the pixels.

.. cpp:function:: void C2DGraphics::DrawImageTransparent (unsigned nX, unsigned nY, unsigned nWidth, unsigned nHeight, TScreenColor *PixelBuffer, TScreenColor TransparentColor)

	Draws an image from a pixel buffer with transparent color. ``nX`` is the image X coordinate. ``nY`` is the image Y coordinate. ``nWidth`` is the image width. ``nHeight`` is the image height. ``PixelBuffer`` is a pointer to the pixels. ``TransparentColor`` is the color to use for transparency.

.. cpp:function:: void C2DGraphics::DrawImageRect (unsigned nX, unsigned nY, unsigned nWidth, unsigned nHeight, unsigned nSourceX, unsigned nSourceY, TScreenColor *PixelBuffer)

	Draws an area of an image from a pixel buffer. ``nX`` is the image X coordinate. ``nY`` is the image Y coordinate. ``nWidth`` is the image width. ``nHeight`` is the image height. ``nSourceX`` is the source X coordinate in the pixel buffer. ``nSourceY`` is the source Y coordinate in the pixel buffer. ``PixelBuffer`` is a pointer to the pixels.

.. cpp:function:: void C2DGraphics::DrawImageRectTransparent (unsigned nX, unsigned nY, unsigned nWidth, unsigned nHeight, unsigned nSourceX, unsigned nSourceY, unsigned nSourceWidth, unsigned nSourceHeight, TScreenColor *PixelBuffer, TScreenColor TransparentColor)

	Draws an area of an image from a pixel buffer with transparent color. ``nX`` is the image X coordinate. ``nY`` is the image Y coordinate. ``nWidth`` is the image width. ``nHeight`` is the image height. ``nSourceX`` is the source X coordinate in the pixel buffer. ``nSourceY`` is the source Y coordinate in the pixel buffer. ``nSourceWidth`` is the source image width. ``nSourceHeight`` is the source image height. ``PixelBuffer`` is a pointer to the pixels. ``TransparentColor`` is the color to use for transparency.

.. cpp:function:: void C2DGraphics::DrawPixel (unsigned nX, unsigned nY, TScreenColor Color)

	Draws a single pixel. ``nX`` is the pixel X coordinate. ``nY`` is the pixel Y coordinate. ``Color`` is the pixel color.

.. note::

	If you need to draw a lot of pixels, consider using :cpp:func:`C2DGraphics::GetBuffer()` for better speed.

.. cpp:function:: TScreenColor *C2DGraphics::GetBuffer (void)

	Gets raw access to the drawing buffer. Returns a pointer to the buffer.

.. cpp:function:: void C2DGraphics::UpdateDisplay (void)

	Once everything has been drawn, updates the display to show the contents on screen. If VSync is enabled, this method is blocking until the screen refresh signal is received (every 16ms for 60 FPS refresh rate).

LVGL
^^^^

The `Light and Versatile Graphics Library <https://lvgl.io/>`_ (LVGL) v8.2.0 can be used with Circle. This library provides an API, which is based on the C language. See the `LVGL documentation <https://docs.lvgl.io/8.2/>`_ for details.

.. code-block:: cpp

	#include <lvgl/lvgl.h>

.. cpp:class:: CLVGL

	This class is a wrapper for LVGL and has to be instantiated to use this graphics library. The wrapper class supports USB mouse or touchscreen input.

.. cpp:function:: CLVGL::CLVGL (CScreenDevice *pScreen, CInterruptSystem *pInterrupt)
.. cpp:function:: CLVGL::CLVGL (CBcmFrameBuffer *pFrameBuffer, CInterruptSystem *pInterrupt)

	Create an instance of this class. ``pScreen`` or ``pFrameBuffer`` reference the display to be used. ``pInterrupt`` is a pointer to the system interrupt object.

.. cpp:function:: boolean CLVGL::Initialize (void)

	Initializes to LVGL support. Returns ``TRUE`` on success.

.. cpp:function:: void CLVGL::Update (boolean bPlugAndPlayUpdated = FALSE)

	Updates the display. This has to be called continuously from the application main loop at ``TASK_LEVEL``. ``bPlugAndPlayUpdated`` must be set to ``TRUE``, if the application supports USB plug-and-play and :cpp:func:`CUSBHostController::UpdatePlugAndPlay()` returned ``TRUE`` too.

µGUI
^^^^

The `µGUI library <http://embeddedlightning.com/ugui/>`_ can be used with Circle. This library provides an API, which is based on the C language. Download the `Reference Guide <http://embeddedlightning.com/download/reference-guide/>`_ for details.

.. code-block:: cpp

	#include <ugui/uguicpp.h>

.. cpp:class:: CUGUI

	This class is a wrapper for µGUI and has to be instantiated to use this graphics library. The wrapper class supports USB mouse or touchscreen input.

.. cpp:function:: CUGUI::CUGUI (CScreenDevice *pScreen)

	Creates an instance of this class. ``pScreen`` references the display to be used.

.. cpp:function:: boolean CUGUI::Initialize (void)

	Initializes to µGUI support. Returns ``TRUE`` on success.

.. cpp:function:: void CUGUI::Update (boolean bPlugAndPlayUpdated = FALSE)

	Updates the display. This has to be called continuously from the application main loop at ``TASK_LEVEL``. ``bPlugAndPlayUpdated`` must be set to ``TRUE``, if the application supports USB plug-and-play and :cpp:func:`CUSBHostController::UpdatePlugAndPlay()` returned ``TRUE`` too.

Accelerated graphics
^^^^^^^^^^^^^^^^^^^^

The accelerated graphics support is described in the :ref:`VC4` subsystem section.
