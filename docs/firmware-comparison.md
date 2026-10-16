# Comparing the firmware versions

Once I had a better understanding of the firmware structure, the next step was to compare the SMART 4G and PRO 4G firmware.

The goal was fairly simple: find out how much of the hardware was actually different and whether it would be possible to adapt the older firmware instead of rewriting anything from scratch.

## The two firmware versions

| Property | SMART 4G | PRO 4G |
|----------|----------|--------|
| Version | Smart-LT_20240729 | 4G ProA_20230221 |
| Size | 31,721,988 bytes | 30,910,652 bytes |
| Platform | HC930 | HC930 |
| Processor | Novatek NT96670 | Novatek NT96670 |
| Image sensor | Sony IMX335 | Sony IMX335 |
| Linux kernel | 4.1.0 | 4.1.0 |
| LCD controller | LS23P02T | ALT020ESLN_J2C |

At first glance, this was encouraging.

Both firmware versions appeared to be built around the same processor, image sensor and operating system.

The most obvious hardware difference was the LCD controller.

## Comparing the Linux filesystems

After extracting the Linux root filesystems from both firmware images, I compared their contents.

Out of 257 files extracted from the older PRO firmware, 242 had identical counterparts in the SMART firmware.

That was quite a bit more similarity than I had expected.

Several important components were identical, including the Linux display driver and its supporting libraries.

For example:

- "nvt_disp.ko"
- "libnvtdisp.so"
- "nvtdisp_test"

This suggested that the display problem was probably not caused by the generic Linux display driver itself.

Instead, the difference was likely somewhere in the hardware specific initialization code.

## The LCD configuration

The SMART firmware contained the following configuration:

"LCD=Disp_IF8B_LCD1_LS23P02T"

The PRO firmware used:

"LCD=Disp_IF8B_LCD1_ALT020ESLN_J2C"

These identifiers correspond to different LCD configurations.

Although both cameras use a similar display interface, the actual initialization commands are not necessarily interchangeable.

Different LCD controllers can require different command sequences, delays and operating parameters.

Simply using the wrong sequence can leave the display powered on without producing a usable image.

## The first cross-flash

Before attempting any modifications, I tried flashing the original PRO 4G firmware onto my SMART 4G camera.

The firmware was accepted and installed, but the result was a completely white LCD.

There was no startup logo, no menu and no camera preview.

Fortunately, reinstalling the original SMART firmware restored the display.

This was an important observation.

It suggested that the older firmware was sufficiently compatible to pass the update process, but something in its display initialization was incompatible with my camera.

Of course, accepting a firmware update does not prove that every part of the firmware is functioning correctly.

## Where to look next?

Initially, changing the LCD configuration seemed like the obvious solution.

Unfortunately, the LCD controller name was only part of the problem.

The camera also uses compiled initialization routines containing the actual commands sent to the display.

Those routines were located in the camera-control firmware rather than the generic Linux display driver.

This meant that fixing the display would require modifying the firmware at the binary level.

And that turned out to be considerably more interesting than changing a configuration string.
