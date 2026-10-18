# Firmware Patching

After identifying the LCD initialization tables, the next step was to transplant them from the SMART firmware into the PRO firmware.

The goal was to preserve the original PRO software while making its display initialization compatible with the LS23P02T controller.

## First attempt "White screen"

Initially, I appended the replacement LCD tables to the executable partitions and updated their references.

The firmware passed the checksum checks, but the camera displayed nothing except a white screen.

The likely problem was memory placement. The additional data could overlap memory used during startup or fall outside the region loaded by the firmware.

## Second attempt "Working display"

Instead of extending the executable partitions, I placed the replacement tables inside existing "0xFF" padding at offset "0x700".

Both partitions contained sufficient space for the 592-byte initialization table.

The following changes were made:

| Modification | Partition 3 | Partition 9 |
|---|---|---|
| New table offset | "0x700" | "0x700" |
| New table address | "0x02700700" | "0x00400700" |
| Table entries | 71 → 73 | 66 → 73 |
| LCD parameter | "0x6B4" → "0x6D6" | "0x6B4" → "0x6D6" |

I also updated the MIPS instructions referencing the tables, corrected their descriptors and changed the Linux LCD configuration to "LS23P02T".

Finally, I recalculated the firmware checksums.

## Result

The modified firmware retained the original PRO firmware size of **30,910,652 bytes**, with 1,227 bytes changed.

This time, flashing was successful. The LCD initialized correctly, the menus and live preview worked and the camera could take photographs and record video.

The final firmware retains the original version identifier "4G ProA_20230221".

**SHA-256:** "032c0ecf6fad02c2c7c880f32e16f3f46ea709e35813968f3953ffc39ef87e32"

The successful result confirms that the combined modifications work on my hardware, although it does not establish which individual change resolved the original display problem.
