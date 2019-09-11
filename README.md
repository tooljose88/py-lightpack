py-lightpack
============

This is a Python library for controlling [Lightpack] custom API from my repo, with load and unload plugins options

Documentation
-------------

See the code or `pydoc lightpack` for full documentation.

Migrating from the official library
-----------------------------------

There are a number of breaking changes from the original library, though code 
should not be difficult to migrate. The changes include the following:

- Support API up to v2.2 (from [Lightpack Fork](https://github.com/psieg/Lightpack)).
- The class name is now in studly case, so use `lightpack.Lightpack()` instead 
  of `lightpack.lightpack()`.
- The constructor now takes named arguments rather than expecting them in a 
  particular order, and the `apikey` argument has been renamed to `api_key`.
- `setSmooth` has been renamed to `setSmoothness`.
- `getAPIStatus` has been renamed to `getApiStatus`.
- Colours are now passed to `setColour` and friends as a single tuple of red, 
  green and blue values rather than separate arguments for each. So use 
  `lp.setColourToAll((10, 255, 128))` rather than `lp.setColourToAll(10, 255, 
  128)`.
- When failing to connect (or when authentication fails during connection) there 
  is now a much more reliable `CannotConnectError`.
- Method calls now raise a `CommandFailedError` on failure where before they 
  were silent.
- Socket connection is buffered to prevent misreadings.
- All API commands are implemented.

Spellings
---------

Methods with the British spellings "colour" now exist, but the American "color" 
spellings are still supported.

Usage example
-------------

```python
from __future__ import print_function

import lightpack
from time import sleep
import sys

# Configuration
# host = 'localhost' # (default)
# port = 3636 # (default)
led_map = [ # Optional aliases for the LEDs in order
	'bottom-right',
	'right-bottom',
	'right-top',
	'top-far-right',
	'top-right',
	'top-left',
	'top-far-left',
	'left-top',
	'left-bottom',
	'bottom-left',
]
# api_key = '{secret-code}' # Default is None

# Connect to the Lightpack API
lp = lightpack.Lightpack(led_map=led_map)
try:
	lp.connect()
except lightpack.CannotConnectError as e:
	print(repr(e))
	sys.exit(1)

# Read the current states
print("Status:           %s" % lp.getStatus())
print("API status:       %s" % lp.getApiStatus())
print("API version:      %s" % lp.getApiVersion())
print("Locked:           %s" % lp.getLockStatus())

print("Devices possible: %s" % ', '.join(lp.getDevices()))
print("Device:           %s" % lp.getDevice())
print("LED count:        %d" % lp.getCountLeds())
print("LED max:          %d" % lp.getMaxLeds())
print("Profiles:         %s" % ', '.join(lp.getProfiles()))
print("Screen size:      %s" % (lp.getScreenSize(),))
print("Monitor 0 size:   %s" % (lp.getMonitorSize(0),))
print("LED 0 size:       %s" % (lp.getLedSizes()[0],))

print("FPS:              %d" % lp.getFps())
print("Mode:             %s" % lp.getMode())
print("Profile:          %s" % lp.getProfile())
print("Colour average:   %s" % lp.getColourAverage())
if lp.getApiVersion() >= '1.5':
	print("Brightness:       %d" % lp.getBrightness())
	print("Smoothness:       %s" % lp.getSmoothness())
	print("Gamma:            %s" % lp.getGamma())
if lp.getApiVersion() >= '2.1':
	print("SoundViz liquid:  %s" % lp.getSoundVizLiquid())
	print("SoundViz colours: %s" % (lp.getSoundVizColours(),))
if lp.getApiVersion() >= '2.2':
	print("Persistence:      %s" % lp.getPersistence())

# Lock the Lightpack so we can make changes
lp.lock()

# Flash green three times
for i in range(3):
	# The American spellings such as setColorToAll are available as aliases
	lp.setColourToAll((0, 255, 0))
	sleep(0.2)
	lp.setColourToAll((0, 0, 0))
	sleep(0.2)

# Set top right light to yellow
# The Colour class is optional
from colour import Colour
lp.setColour('top-right', Colour('yellow'))

sleep(1)

# Set left bottom and left right lights to two other colours
lp.setColours(('left-bottom', Colour('red')), ('left-top', Colour('goldenrod')))

sleep(1)

# Unlock to release control (the disconnect method actually calls this 
# automatically, but it is often useful on its own so is here for informational 
# purposes)
lp.unlock()

# Disconnect
lp.disconnect()
```
