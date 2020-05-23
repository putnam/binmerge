# binmerge

Source code available at: https://github.com/putnam/binmerge

Tool to merge multiple bin/cue tracks into one.

Sometimes people rip discs in such a way that they have a separate bin file for every track. One example that I know of is the Redump project, specifically for the Playstation 1 or PSX.

For example:

```
FILE "Big Buddy (Track 01).bin" BINARY
  TRACK 01 MODE2/2352
    INDEX 01 00:00:00
FILE "Big Buddy (Track 02).bin" BINARY
  TRACK 02 AUDIO
    INDEX 00 00:00:00
    INDEX 01 00:02:00
FILE "Big Buddy (Track 03).bin" BINARY
  TRACK 03 AUDIO
    INDEX 00 00:00:00
    INDEX 01 00:02:00
FILE "Big Buddy (Track 04).bin" BINARY
  TRACK 04 AUDIO
    INDEX 00 00:00:00
    INDEX 01 00:02:00
FILE "Big Buddy (Track 05).bin" BINARY
  TRACK 05 AUDIO
    INDEX 00 00:00:00
    INDEX 01 00:02:00
...
```

Some incomplete software cannot read this style of disc image, because they only know how to work with a single bin file.

This script reads a cuesheet and series of bin files, and generates a new merged bin and cuesheet. It is completely non-destructive; it will not touch your existing files.

Here is the above example after being procesed by binmerge:
```
FILE "Big Buddy.bin" BINARY
  TRACK 01 MODE2/2352
    INDEX 01 00:00:00
  TRACK 02 AUDIO
    INDEX 00 32:26:61
    INDEX 01 32:28:61
  TRACK 03 AUDIO
    INDEX 00 35:43:29
    INDEX 01 35:45:29
  TRACK 04 AUDIO
    INDEX 00 36:47:27
    INDEX 01 36:49:27
  TRACK 05 AUDIO
    INDEX 00 38:50:66
    INDEX 01 38:52:66
```

If you want to return to the split bin format you can instead pass a merged cue file with the --split parameter.

This is useful for example to verify PSX split bin files against the Redump project's PSX DAT file.

Users of MAME's chdman tool may also find this option useful as chdman returns the merged cue format when decompressing.

Have fun!
