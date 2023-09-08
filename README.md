# binmerge

Source code available at: https://github.com/putnam/binmerge

Tool to merge multiple bin/cue tracks into one.

Sometimes discs are ripped in such a way that they have a separate bin file for every track. One example that I know of is the Redump project, specifically for the Playstation 1 or PSX.

Here is a cuesheet for the imaginary PSX game "Big Buddy". You can see it refers to several individual bin files, one for each track:

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

Some software cannot read this style of disc image, because they only know how to work with a single bin file or are unable to properly parse cuesheets according to the standard.

`binmerge` reads a cuesheet and its associated series of bin files and generates a new, single merged bin file and cuesheet. It is completely non-destructive; it will not touch your existing files.

Here is the above example after being processed by binmerge:
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

`binmerge` also supports reversing the process if you deleted the original files to save space. If you want to return to the split bin format you can instead pass a merged cue file with the `--split` parameter. However, for systems that have metadata tags (Dreamcast), these tags are currently not preserved by binmerge and will be missing. Complete cuesheet packs are available to download on Redump's site.

Have fun!
