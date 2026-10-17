# HC930

Firmware experiments and reverse engineering of cameras based on the HC930 platform.

## Overview
I had a trail camera running the SMART 4G firmware, which relies on Tuya's cloud services. I was not particularly satisfied with that, so I started looking into whether I could replace its firmware with an older, non-Tuya version.

The problem was that although the camera appeared to use almost identical physical hardware, they had different LCD controllers.

Flashing the older PRO 4G firmware resulted in a completely white screen. The camera itself appeared to be running, but the display was unusable.

Since the original firmware could still be restored, I decided to investigate what was different and whether I could make the older firmware work with my display.

## What was changed?

The original PRO 4G firmware was modified to support the LS23P02T LCD controller used in my camera.

## Project Objectives

- Extracting and comparing the firmware images.
- Identifying the LCD initialization sequences.
- Comparing the display related code between firmware versions.
- Modifying initialization tables and their references.
- Finding out the hard way that memory placement matters.
- Rebuilding the firmware and testing it on the camera.

The resulting firmware is still based on the original PRO 4G version, rather than the SMART firmware.

## Firmware Versions

| Firmware | Version | LCD Controller |
|----------|---------|----------------|
| Original SMART 4G | Smart-LT_20240729 | LS23P02T |
| Original PRO 4G | 4G ProA_20230221 | ALT020ESLN_J2C |
| Modified PRO 4G | 4G ProA_20230221 | LS23P02T |

## Does it even work?

Yes, at least the parts I was trying to fix.

The modified firmware boots, the LCD works correctly, and the camera can take photographs and record video.

The first few attempts were not particularly successful, including one that left me staring at a blank white screen again.

I will document those experiments here as well, since understanding why something did not work is probably just as useful as the final result.

## Documentation

I will be adding the firmware analysis, modification scripts, binary comparisons, and notes from the different experiments.

Hopefully, some of this will be useful to someone else dealing with these HC930-based cameras.

## Disclaimer

This is an experimental project, not an official firmware release. The research and modifications documented here are provided for educational purposes.

Modifying firmware can render hardware unusable. If you decide to try any of this yourself, make sure you have a way to restore the original firmware.
