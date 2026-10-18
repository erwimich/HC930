# LCD Initialization

The first thing I noticed when comparing the firmware was that both versions used the same Linux display drivers.

The files "nvt_disp.ko", "libnvtdisp.so" and "nvtdisp_test" were identical, so replacing those was not going to solve anything.

The actual difference had to be somewhere else.

## Finding the initialization sequences

I started looking through the camera-control firmware for references to LCD-related functions and configuration data.

One of the functions present in both versions was "DrvLCD_TurnOn".

Interestingly, the corresponding code was identical. This suggested that the problem was not necessarily the function responsible for enabling the display, but rather the initialization data supplied to it.

Eventually, I found two sets of LCD initialization tables, one in partition 3 and another in partition 9.

Both firmware versions contained these tables, but their contents were different.

## Table locations

The following offsets were identified in the extracted firmware partitions:

| Firmware | Partition | Table offset | Entries |
|----------|-----------|--------------|---------|
| SMART 4G | 3 | "0x16A228" | 73 |
| SMART 4G | 9 | "0x80742C" | 73 |
| PRO 4G | 3 | "0x16A25C" | 71 |
| PRO 4G | 9 | "0x82712C" | 66 |

These offsets are relative to the beginning of each partition, not the complete firmware image.

The two initialization tables in the SMART firmware were identical, while the PRO firmware contained two different sequences.

This was the first indication that I had found something more useful than another LCD configuration string.

## Understanding the data

The tables consist of pairs of 32-bit values stored in little-endian format.

The first value identifies the type of operation, while the second contains a command, parameter or delay value.

Four different operation types:

| Type | Value | Meaning |
|------|-------|---------|
| Command | "0x01000000" | Send LCD command |
| Data | "0x02000000" | Send command parameter |
| Delay | "0x80000000" | Wait before continuing |
| Control | "0x40000000" | Table control information |

These meanings were inferred from the binary data and the command sequences. They are not taken from official Novatek documentation.

For example, the beginning of the SMART initialization sequence contains:

```text
0x01000000  0x00000011
0x80000000  0x00000050
0x01000000  0x000000B9
0x02000000  0x000000FF
0x02000000  0x00000093
0x02000000  0x00000042
```

This can be interpreted as:

```text
CMD    0x11
DELAY  80 ms
CMD    0xB9
DATA   0xFF
DATA   0x93
DATA   0x42
```

The original PRO firmware uses different values and delays, which helps explain why its initialization sequence does not work correctly with the LS23P02T panel.

## Finding the references

Locating the tables was only half the problem.

The firmware also needed to know where to find them.

Each table has an associated descriptor containing a pointer to its data and the number of entries.

For example, in the original PRO firmware partition 3:

```text
Table header:      0x16A25C
Descriptor:        0x16A578
Runtime pointer:   0x0286A264
Entry count:       71
```

And in partition 9:

```text
Table header:      0x82712C
Descriptor:        0x827420
Runtime pointer:   0x00C27134
Entry count:       66
```

The runtime pointers are not simply file offsets.

They are calculated using the memory address at which the corresponding executable image is loaded.

For example, partition 3 uses a base address of "0x02700000".

Its original table pointer can therefore be calculated as:

```text
0x02700000  Image base
0x0016A25C  Table header offset
0x00000008  Header size
-------------------------------
0x0286A264  Table data address
```

This also gave me a way to verify that the descriptors actually referenced the initialization tables found before.

## Why copying the tables was not enough

The working SMART initialization tables are larger than their PRO counterparts.

Each entry occupies eight bytes, and each table also has an eight-byte header.

The sizes work out as follows:

| Firmware | Partition | Table size |
|----------|-----------|------------|
| SMART 4G | 3 | 592 bytes |
| SMART 4G | 9 | 592 bytes |
| PRO 4G | 3 | 576 bytes |
| PRO 4G | 9 | 536 bytes |

This presented a problem.

The replacement table needed an additional 16 bytes in partition 3 and 56 bytes in partition 9.

Simply copying the SMART tables over the existing PRO tables would overwrite the data following them.

Instead, the replacement tables would have to be relocated, and the references to them updated.

That meant modifying both the table descriptors and the MIPS instructions that construct their addresses.

I also had to change the entry counts:

```text
Partition 3: 71 -> 73
Partition 9: 66 -> 73
```

At this point, I had identified the initialization data and understood how the firmware accessed it.

What I had not figured out yet was where the replacement tables could safely be placed.

My first attempt at relocating them failed and produced another completely white screen.

The eventual solution required finding a suitable location inside the existing executable images, rather than simply appending more data to them.
