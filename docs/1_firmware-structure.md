## Taking a closer look inside the firmware

Before modifying anything, I wanted to understand what was actually inside these firmware files.

Both cameras use firmware distributed as "FWHC900A.bin", but that does not necessarily mean the files are interchangeable.

I started by comparing two versions:

| Firmware | Version | Size |
|----------|---------|------|
| SMART 4G | Smart-LT_20240729 | 31,721,988 bytes |
| PRO 4G | 4G ProA_20230221 | 30,910,652 bytes |

The SMART firmware was my reference because I knew it worked with my camera's hardware.

The PRO firmware was the one I wanted to adapt.

## What hardware are we dealing with here?

Looking through the firmware revealed a few useful details:

- **SoC:** Novatek NT96670
- **Architecture:** MIPS32, little-endian
- **Operating system:** Embedded Linux 4.1.0
- **Firmware platform:** HC930
- **Filesystem:** UBIFS

This was encouraging because both firmware versions appeared to share much of their underlying architecture.

However, similar hardware does not automatically mean compatible firmware, as I would soon discover.

## Firmware structure

The files are not just raw executable binaries.

They use Novatek's "NVTPACK_FW_HDR2" firmware format.

The format includes a header containing information about the firmware and the locations of its different partitions.

The firmware images I examined contained seven partitions:

| Partition ID | Contents |
|--------------|----------|
| 1 | Firmware/build configuration |
| 2 | Additional configuration and metadata |
| 3 | Camera application and control code |
| 4 | Additional embedded executable data |
| 6 | Linux kernel image |
| 7 | Linux root filesystem |
| 9 | Additional camera application data |

The partition IDs are not consecutive, which initially looks a little bit strange, but that is simply how these firmware packages are organized

## Extracting the Linux filesystem

Partition 7 was particularly interesting.

It contains a UBI image holding the Linux filesystem.

In the SMART firmware, the partition starts at file offset "0x009CD6D4".

The first 64 bytes form a checksum wrapper, meaning the actual UBI data begins at:
"0x009CD714"

The UBI eraseblock size is 131,072 bytes.

After extracting the filesystem, it became possible to examine the camera's Linux applications, configuration files, libraries, and startup scripts.

Some particularly interesting files included:

- "/bin/vk4g" - Camera's cellular communication application.
- "/etc/profile_prjcfg" - Hardware and project configuration.
- "/bin/nvtrtspd" - RTSP streaming server executable.
- "/usr/bin/rtspd" - Another RTSP-related executable.

Finding RTSP-related software was also interesting because it suggested that the firmware had more capabilities than those exposed through the original application.

Of course, having an executable in the filesystem does not necessarily mean the feature is enabled.

## What about the LCD?

This was the important part.

The two firmware versions used different display configurations.

SMART 4G:

"Disp_IF8B_LCD1_LS23P02T"

PRO 4G:

"Disp_IF8B_LCD1_ALT020ESLN_J2C"

This gave me my first concrete explanation for the display incompatibility that I later discovered.

However, simply changing the configuration string was not enough.

The firmware also contained compiled initialization code that sends commands to the LCD controller.

That would later turn out to be the more difficult part of the modification.
