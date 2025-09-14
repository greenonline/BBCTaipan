# Using BBC floppy disk images

## Links

 - [Save a program to disk](https://stardot.org.uk/forums/viewtopic.php?t=26904)
 - [How do I format a floppy disk in a BBC Master 128?](https://www.stardot.org.uk/forums/viewtopic.php?t=28549) - good
   - [Acorn Disk System UGI2.pdf](https://chrisacorns.computinghistory.org.uk/docs/Acorn/Manuals/Acorn_DiscSystemUGI2.pdf)
 - [BASIC disk format](https://stardot.org.uk/forums/viewtopic.php?t=14295&sid=97d4cb843cd0e868d83d794f310f02f3) - meh
 - [41 Operating system statements](https://central.kaserver5.org/Kasoft/Typeset/BBC/Ch41.html)
 - [Hardware library](https://8bs.com/othrdnld/manuals/hardwarebbcseries.shtml)
   - [Opus DFS Manual.rtf](https://8bs.com/othrdnld/manuals/hardware/OpusDiskManualCUC.zip)

### More disk stuff

 - [Writing ssd disk images on a BBC Master](https://stardot.org.uk/forums/viewtopic.php?t=20642)
 - [Getting .SSD or .DSD images onto real floppies.](https://stardot.org.uk/forums/viewtopic.php?t=16780)

## Process

1. Get a disk image from here, [BBC MICRO SOFTWARE ARCHIVE - DISK IMAGES](https://www.stairwaytohell.com/bbc/index.html?page=diskimages), i.e. the [TheHobbit-universaldiscversion.zip](https://www.stairwaytohell.com/bbc/archive/diskimages/MelbourneHouse/TheHobbit-universaldiscversion.zip)
2. Unzip the ZIP file
3. Mount in BeebEm
4. View disk contents, with `*CAT`
5. Erase all of the HOB* files, to make some space, using `*DELETE`
6. Ummount
7. Duplicate and rename as TAIPAN.SSD
8. Mount in BeebEm
9. Paste in the BBC BASIC version of Taipan
10. Save Taipan, using `SAVE  ":0.$.TAIPAN"`
11. Quit BeebEm
12. Relaunch BeebEm
13. Load the BAIC code, using `LOAD  ":0.$.TAIPAN"`, or just `LOAD"TAIPAN"`



### Formatting

From [How do I format a floppy disk in a BBC Master 128?](https://www.stardot.org.uk/forums/viewtopic.php?t=28549)

```none
*DISC
*FORM80 0
or
*FORM40 0
```


## `!BOOT`

```none
10 CHAIN"TAIPAN"
```

Then `SAVE  ":0.$.!BOOT"`

Then `*OPT 4 3`

