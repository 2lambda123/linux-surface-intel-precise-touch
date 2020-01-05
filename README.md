# Intel Precise Touch & Stylus

Linux kernel driver for Intels IPTS touchscreen technology. With IPTS, the
Intel Management Engine acts as an interface for a touch controller, that
returns either raw capacitive touch data (for multitouch mode), or HID report
data (for singletouch mode). This data is processed outside of the ME (either
on the GPU or the driver directly) and then relayed into the HID subsystem of
the operating system.

### Original IPTS driver
The original driver for IPTS was released by Intel in late 2016
(https://github.com/ipts-linux-org/ipts-linux-new). It uses GuC submission
to process raw touch input data on the GPU, using OpenCL firmware extracted
from Windows.

An updated version of the driver, with support for linux 5.3 can be found here:
https://github.com/linux-surface/kernel/tree/v5.3-surface-devel/drivers/misc/ipts

### This driver
This driver is loosely based on the original implementation. The main difference
is, that due to changes in the i915 graphics driver we cannot easily use GuC
submission anymore on kernel 5.4+. This driver is not using GuC submission,
instead we are reverse engineering the raw touch data, and implementing our
own parser for it directly in the driver. Other changes are a significant
removal of unused and dead code compared to the driver provided by Intel.

### Working
* Stylus input, including tild (tested on SB2, should work on other devices)

### Not working
* Finger touch input
* Support for IPTS on gen7

### Building (in-tree)
* Add this repository as `drivers/input/touchscreen/ipts`
* Add `source "drivers/input/touchscreen/ipts/Kconfig"` to
  `drivers/input/touchscreen/Kconfig`
* Add `obj-y += ipts/` to `drivers/input/touchscreen/Makefile`

### Building (out-of-tree)
* Clone this repository and `cd` into it
* Make sure you applied the patches from `patches/` to your kernel
* Run `make all`
* Run `sudo insmod ipts.ko`

### Building (DKMS)
* Clone this repository and `cd` into it
* Make sure you applied the patches from `patches/` to your kernel
* Run `sudo make dkms-install`
* Reboot

### Kconfig options
* `CONFIG_TOUCHSCREEN_IPTS=[y/m/n]`
* `CONFIG_TOUCHSCREEN_IPTS_DEBUG=[y/n]`

When building out-of-tree, `CONFIG_TOUCHSCREEN_IPTS_DEBUG` can be enabled by
passing `DEBUG=y` to the `make all` command.

Enabling the debug option will cause the driver to dump the touch data buffers
into dmesg, encoded as hexadecimal. For proper reverse engineering, the
resulting HID data is required though, which can only be fetched using the old
IPTS driver on a kernel that supports it (5.3 or lower).
