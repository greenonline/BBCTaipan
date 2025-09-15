## Preamble

Revisiting the age old problem of graphics, namely the implementation of AppleSoft BASIC `INVERSE`, and the vertical half block character, missing from the Apple II character set, yet present on the TRS80, in the form of `CHR$(133)` when drawing the goods table.

We require a 40 column screen mode, as that is the screen size used by the Apple II. That meeans modes 1, 4, 6, and 7. Modes 1 and 4 give `Bad MODE` errors, and `MODE 6` gives a memory error when dimensioning arrays. So the 40x25 `MODE 7` *seems* to be the natural default, yet it isn't without its own drawbacks.

On this page I wonder through `MODE 7` graphics characters, I try plain text blocks, dabble with colour, wrestle with the control character in the first column at TAB 0, and finally try to resort to using modes 1, 4 and 6, only to be forced to return to `MODE 7`.

I probably went over all of this two years ago in the winter of 2023, in my blog [Taipan for BBC BASIC](https://gr33nonline.wordpress.com/2023/12/12/taipan-for-bbc-basic/), whilst coming up with the inital port. However, I've forgotten it all and can't be bothered to re-read my notes.  Previously, I had had my fill with the whole port, from Apple to BBC, so that by the time the graphics were to be considered, I had started to lose interest and enthusiasm. This time though, in late 2025, I have tried to be a lot more thorough, and explored the various possiblities.

The main screen should look like this screenshot from the Apple II, even though it *is* missing the vertical half bar characters (`CHR$(133)`), as they don't exist on the Apple II:

[![Apple II screenshot of Taipan][1]][1]

However, there are one column indents in `MODE 7` that can not be avoided, when emulating the handy `INVERSE` function of the Apple II – the BBC has no inverse command, so to speak, in `MODE 7`, it doesn't exist! So you end up with either a 39 column display, like this:

[![BBCTaipan_MODE7_MainScreen_Asymmetrical_GoodsStatsTempleMarketFixed][2]][2]

 or an asymmetrical display, like this:

[![Asymmetrical MODE 7 screenshot of Taipan][3]][3]

## See also

 - Meh

## Links

 - [Character editor](https://stardot.org.uk/forums/viewtopic.php?t=19556)
   - [DefChar font editor](https://mdfs.net/Apps/Font/)
 - [BBC BASIC for Windows - MODE 7 - Teletext](https://www.bbcbasic.co.uk/bbcwin/manual/bbcwinh.html) - Very useful
 - [Mode 7 teleetxt graphics question](https://www.stardot.org.uk/forums/viewtopic.php?t=21572)
 - [Introduction to VDU commands - VDU code summary](https://www.bbcbasic.co.uk/bbcwin/manual/bbcwin8.html)
 - [BBC BASIC for SDL 2.0 - BBC BASIC for Windows - Graphics and Colours](https://www.bbcbasic.co.uk/bbcwin/manual/bbcwin3.html) for the list of resolutions.


## Notes

```none
5 REM FROM https://www.bbcbasic.co.uk/bbcwin/manual/bbcwinh.html
10 MODE 7
30 PRINT "A red box:"CHR$145;CHR$247;CHR$251
40 PRINT CHR$151;CHR$255;CHR$255;" MARKET PRICES "CHR$151;CHR$255;CHR$255
50 PRINT CHR$151;:FOR N=1 TO 30: PRINT CHR$255;: NEXT N
60 PRINT CHR$255: REM WITHOUT THIS A CHR$190 IS PRINTED AT END OF THE LINE


70 PRINT CHR$151;:FOR N=1 TO 30: PRINT CHR$255;: NEXT N: PRINT CHR$255
```

[![BBCTaipan_MODE7_FirstTestExample][4]][4]

But there will be an issue when the loction name is introduced, as the line length will vary, and it will be necessary to calaculate how many blocks to print, in order to fill the line correctly.

It would be easier to print a line of screen width of blocks and then overlay the title on them.


```none
10 MODE 7
50 PRINT CHR$151;:FOR N=1 TO 40: PRINT CHR$255;: NEXT N: PRINT CHR$255
```

Note that the first character of the line is not a block, as this is the control character. Therefore, it may be necessary to indent all text by one, so that there are no obvious misalignment issues.

[![BBCTaipan_MODE7_FirstTestExample_Overspill][5]][5]

Using 40 there are two characters overspilling on to the following line, so use 38 instead:

```none
10 MODE 7
50 PRINT TAB(0,13);CHR$151;:FOR N=1 TO 38: PRINT CHR$255;: NEXT N: PRINT CHR$255
```

[![BBCTaipan_MODE7_FirstTestExample_NoOverspill][6]][6]

Now to overlay the text

```none
60 PRINT TAB(8,13); " HONGKONG MARKET PRICES "
```

Looks OK, good even.

[![BBCTaipan_MODE7_FirstTestExample_MarketPrices][7]][7]


Without the control char, white blocks (with a black border) are printed - even in the first column

```none
10 MODE 7
50 PRINT TAB(0,13);:FOR N=1 TO 39: PRINT CHR$255;: NEXT N: PRINT CHR$255
60 PRINT TAB(8,13); " HONGKONG MARKET PRICES "
```

Whilst a little "blocky", it looks OK, is more symetrical, and doesn't require the (non-printing) colour control character, nor shifting everything by one to the right.

[![BBCTaipan_MODE7_FirstTestExample_MarketPrices_TextBlocks][8]][8]  

### Separated graphics - ugly

Both look the same, seperated

```none
10 MODE 7
50 PRINT TAB(0,13);CHR$154;:FOR N=1 TO 38: PRINT CHR$255;: NEXT N: PRINT CHR$255
60 PRINT TAB(8,13); " HONGKONG MARKET PRICES "
70 PRINT TAB(0,14);CHR$153;:FOR N=1 TO 38: PRINT CHR$255;: NEXT N: PRINT CHR$255
80 PRINT TAB(8,14); " HONGKONG MARKET PRICES "
```

[![BBCTaipan_MODE7_SeparatedGraphics#1][9]][9]

You need the block graphics colour control character, but then the first line blocks look very strange – as six minor blocks, a.k.a. *sixels*. This example shows the difference(s):

```none
10 MODE 7
50 PRINT TAB(0,13);CHR$151;CHR$154;:FOR N=1 TO 38: PRINT CHR$255;: NEXT N: PRINT CHR$255
60 PRINT TAB(8,13); " HONGKONG MARKET PRICES "
70 PRINT TAB(0,14);CHR$151;CHR$153;:FOR N=1 TO 38: PRINT CHR$255;: NEXT N: PRINT CHR$255
80 PRINT TAB(8,14); " HONGKONG MARKET PRICES "
89 REM WITHOUT COLOUR GRAPHICS CONTROL CHARACTER
90 PRINT TAB(0,15);CHR$154;:FOR N=1 TO 38: PRINT CHR$255;: NEXT N: PRINT CHR$255
95 PRINT TAB(8,15); " HONGKONG MARKET PRICES "
```

[![BBCTaipan_MODE7_SeparatedGraphics1][10]][10]

Commands:

```none
Turn on:  CHR$154
Turn off: CHR$153
```

### Held graphics

Use this to avoid the space character at the start of the line

```none
10 MODE 7
50 PRINT TAB(0,13);CHR$151;:FOR N=1 TO 38: PRINT CHR$255;: NEXT N: PRINT CHR$255
60 PRINT TAB(0,14);CHR$158;CHR$151;:FOR N=1 TO 38: PRINT CHR$255;: NEXT N: PRINT CHR$255
```
Does not *seem to* work, still with control characters as spaces. Investigated further in **More hold characters (`CHR$158`)** below.

[![BBCTaipan_MODE7_HeldGraphics][11]][11]

Commands:

```none
Turn on:  CHR$158
Turn off: CHR$159
```

#### Another held graphics example

From [this answer](https://stackoverflow.com/a/42572444/4424636) to [BBC Basic: Inserting a control character without occupying space in Mode 7](https://stackoverflow.com/q/38662354/4424636), which leads to [Teletext Mode - Placing blocks of colour next to each other](http://www.riscos.com/support/developers/bbcbasic/part2/teletext.html)

```none
PRINT CHR$(145)CHR$(152)CHR$(255)CHR$(158)CHR$(146) CHR$(147)CHR$(159)
```

The control characters are as follows:

```none
red;conceal;block;held_on;green yellow;held_off
```

[![BBCTaipan_MODE7_AnotherHeldGraphicsExample][12]][12]
  
### The Taipan main goods held table

#### Using normal text block

The original barebones code:

```none
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);G$(I):PRINT TAB(10,4+I); CHR$ (124);: Q=SG(I):GOSUB 1330:PRINT TAB(25,4+I); CHR$ (124);: Q = GG(I):GOSUB 1330: NEXT I:INVERSE=0:PRINT A$:NORMAL=0: RETURN
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);G$(I):PRINT TAB(10,4+I); CHR$ (133);: Q=SG(I):GOSUB 1330:PRINT TAB(25,4+I); CHR$ (133);: Q = GG(I):GOSUB 1330: NEXT I:INVERSE=0:PRINT A$:NORMAL=0: RETURN
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);G$(I):PRINT TAB(10,4+I); : Q=SG(I):GOSUB 1330:PRINT TAB(25,4+I); : Q = GG(I):GOSUB 1330: NEXT I:INVERSE=0:PRINT A$:NORMAL=0: RETURN
```

[![BBCTaipan_MODE7_GoodsHeld_Plain][13]][13]

Not sure where the `CHR$124` comes from, TODO Check Blog.

Anyway, reinstating the `CHR$133`, but as `CHR$181`/`CHR$234`, so changing

```none
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);G$(I):PRINT TAB(10,4+I); : Q=SG(I):GOSUB 1330:PRINT TAB(25,4+I); : Q = GG(I):GOSUB 1330: NEXT I:INVERSE=0:PRINT A$:NORMAL=0: RETURN
```

to

```none
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);G$(I):PRINT TAB(10,4+I); CHR$181; : Q=SG(I):GOSUB 1330:PRINT TAB(25,4+I); CHR$181;: Q = GG(I):GOSUB 1330: NEXT I:INVERSE=0:PRINT A$:NORMAL=0: RETURN
```

[![BBCTaipan_MODE7_GoodsHeld_BadFive][14]][14]

But this doesn't work. Why? Here is an MRE which just prints "555"

```none
5 MODE 7
10 PRINT CHR$181;CHR$181;CHR$181
```

[![BBCTaipan_MODE7_GoodsHeld_BadMRE_1][15]][15]

Another MRE, reverting to one of the original examples above, also just prints a row of "5" characters.

```none
10 MODE 7
50 PRINT TAB(0,13);:FOR N=1 TO 39: PRINT CHR$181;: NEXT N: PRINT CHR$255
60 PRINT TAB(8,13); " HONGKONG MARKET PRICES "
```

[![BBCTaipan_MODE7_GoodsHeld_BadMRE_2][16]][16]

So, it can be deduced that 255 is a block in "NORMAL" mode, but the other block characters are *not* available in "normal" mode - a colour control character is required (which brings the asymmetry back). See **Character set** section below:

```none
10 MODE 7
50 PRINT TAB(0,13);CHR$151;:FOR N=1 TO 39: PRINT CHR$181;: NEXT N: PRINT CHR$255
60 PRINT TAB(8,13); " HONGKONG MARKET PRICES "
```

[![BBCTaipan_MODE7_GoodsHeld_BadMRE_3][17]][17]

So, using the "normal mode" outlined blocks (`CHR$255`) in normal mode instead of the thick pipe:

```none
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);G$(I):PRINT TAB(10,4+I); CHR$255; : Q=SG(I):GOSUB 1330:PRINT TAB(25,4+I); CHR$255;: Q = GG(I):GOSUB 1330: NEXT I:INVERSE=0:PRINT A$:NORMAL=0: RETURN
```

[![BBCTaipan_MODE7_GoodsHeld_TextBlocks][18]][18]

It just looks awful.

or with the bar as well

```none
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);G$(I):PRINT TAB(10,4+I); CHR$255; : Q=SG(I):GOSUB 1330:PRINT TAB(25,4+I); CHR$255;: Q = GG(I):GOSUB 1330: NEXT I:FOR N=1 TO 39: PRINT CHR$255;: NEXT N: RETURN
```

[![BBCTaipan_MODE7_GoodsHeld_TextBlocksBar][19]][19]

Not *so* bad, although the columns are badly spaced.


#### Using graphics blocks

This will render some columns unuseable in the table, so very long numbers, and the (main display display of the) Bank hack, may not be displayed properly.

##### Using the graphics blocks (`CHR$255`) in normal mode instead of the thick pipe

```none
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);G$(I):PRINT TAB(10,4+I); CHR$151;CHR$255; : Q=SG(I):GOSUB 1330:PRINT TAB(25,4+I); CHR$151;CHR$255;: Q = GG(I):GOSUB 1330: NEXT I:PRINT CHR$151;:FOR N=1 TO 39: PRINT CHR$255;: NEXT N: RETURN
```

[![BBCTaipan_MODE7_GoodsHeld_TextBlocksBar_Bad][20]][20]

Meh. For some reason there is a `CHR$176` printed, which must be the "0". Need to return to text mode `CHR$135`:

```none
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);G$(I):PRINT TAB(10,4+I); CHR$151;CHR$255;CHR$135; : Q=SG(I):GOSUB 1330:PRINT TAB(25,4+I); CHR$151;CHR$255;CHR$135;: Q = GG(I):GOSUB 1330: NEXT I:PRINT CHR$151;:FOR N=1 TO 39: PRINT CHR$255;: NEXT N: RETURN
```

Too long by just 4 characters! Use `P.` instead of PRINT, for the bottom bar:

```none
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);G$(I):PRINT TAB(10,4+I); CHR$151;CHR$255;CHR$135; : Q=SG(I):GOSUB 1330:PRINT TAB(25,4+I); CHR$151;CHR$255;CHR$135;: Q = GG(I):GOSUB 1330: NEXT I:P. CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N: RETURN
```

[![BBCTaipan_MODE7_GoodsHeld_TextBlocksBar_Good][21]][21]

Better.

##### Using the graphics thick pipe (`CHR$181`/`CHR$234`)

```none
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);G$(I):PRINT TAB(10,4+I); CHR$151;CHR$181;CHR$135; : Q=SG(I):GOSUB 1330:PRINT TAB(25,4+I); CHR$151;CHR$181;CHR$135;: Q = GG(I):GOSUB 1330: NEXT I:P. CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N: RETURN
```

[![BBCTaipan_MODE7_GoodsHeld_TextBlocksBar_Pipe][22]][22]

Much better. 

Adding in the upper white bar, for the table headings

```none
141 P. TAB(0,3); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3); "GOODS     ABOARD SHIP    HONGKONG GODOWN"
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);G$(I):PRINT TAB(10,4+I); CHR$151;CHR$181;CHR$135; : Q=SG(I):GOSUB 1330:PRINT TAB(25,4+I); CHR$151;CHR$181;CHR$135;: Q = GG(I):GOSUB 1330: NEXT I:P. CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N: RETURN
```

[![BBCTaipan_MODE7_GoodsHeld_TextBlocksTwoBar_Pipe_Bad][23]][23]

But the table headings wipe out the white bar! The heading string will need to be broken up and the words printed individually. However, without inverse chracters, it looks a bit crap. Also, the inability to have the first character of any line as a block character (as it is taken up by a control character) is pretty rubbish.

```none
141 P. TAB(0,3); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3); "GOODS"; TAB(10,3);"ABOARD SHIP";TAB(25,3);"HONGKONG GODOWN"
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);G$(I):PRINT TAB(10,4+I); CHR$151;CHR$181;CHR$135; : Q=SG(I):GOSUB 1330:PRINT TAB(25,4+I); CHR$151;CHR$181;CHR$135;: Q = GG(I):GOSUB 1330: NEXT I:P. CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N: RETURN
```

[![BBCTaipan_MODE7_GoodsHeld_TextBlocksTwoBar_Pipe_Bad2][24]][24]

but now the bits of *solid* bar that show through are converted to text mode blocks. Need to prefix with `CHR$151`, which will indent the goods heading

```none
141 P. TAB(0,3); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3);CHR$151; "GOODS"; TAB(10,3);"ABOARD SHIP";TAB(25,3);"HONGKONG GODOWN"
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);G$(I):PRINT TAB(10,4+I); CHR$151;CHR$181;CHR$135; : Q=SG(I):GOSUB 1330:PRINT TAB(25,4+I); CHR$151;CHR$181;CHR$135;: Q = GG(I):GOSUB 1330: NEXT I:P. CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N: RETURN
```

[![BBCTaipan_MODE7_GoodsHeld_TextBlocksTwoBar_Pipe_Better][25]][25]

Better. Still need to shift the pipe columns left by two characters, whilst leaving the numbers/value where they are. Add extra `TAB()` locators:

```none
141 P. TAB(0,3); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3);CHR$151; "GOODS"; TAB(10,3);"ABOARD SHIP";TAB(25,3);"HONGKONG GODOWN"
150 FORI=0TO5:P.TAB(0,4+I);G$(I):P.TAB(7,4+I);CHR$151;CHR$181;CHR$135:P.TAB(10,4+I);: Q=SG(I):GOSUB 1330:P.TAB(22,4+I);CHR$151;CHR$181;CHR$135:P.TAB(25,4+I);:Q=GG(I):GOSUB 1330:NEXTI:P.CHR$151;:FOR N=1TO39:P.CHR$255;:NEXTN:RETURN
```

[![BBCTaipan_MODE7_GoodsHeld_TextBlocksTwoBar_Pipe_Shift][26]][26]

Note: The line 150 is getting too long again, reaching maximum capacity. Is it possible to split? Yes, lines 151-154 available.

Looks Ok, not bad at all.

Need to fix the missing indent on the right hand side of lower bar:

```none
141 P. TAB(0,3); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3);CHR$151; "GOODS"; TAB(10,3);"ABOARD SHIP";TAB(25,3);"HONGKONG GODOWN0"
150 FORI=0TO5:P.TAB(0,4+I);G$(I):P.TAB(7,4+I);CHR$151;CHR$181;CHR$135:P.TAB(10,4+I);: Q=SG(I):GOSUB 1330:P.TAB(22,4+I);CHR$151;CHR$181;CHR$135:P.TAB(25,4+I);:Q=GG(I):GOSUB 1330:NEXTI:P.CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:RETURN
```

[![BBCTaipan_MODE7_GoodsHeld_TextBlocksTwoBar_Pipe_RightIndent][27]][27]

Need to shift the values right by, at least, two columns:

```none
141 P. TAB(0,3); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3);CHR$151; "GOODS"; TAB(10,3);"ABOARD SHIP";TAB(25,3);"HONGKONG GODOWN"
150 FORI=0TO5:P.TAB(0,4+I);G$(I):P.TAB(7,4+I);CHR$151;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330:P.TAB(22,4+I);CHR$151;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:NEXTI:P.CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:RETURN
```

[![BBCTaipan_MODE7_GoodsHeld_TextBlocksTwoBar_Pipe_ShiftAgain][28]][28]

Looks a bit better. Maybe add more BLOCKS for the space characters in "ABROAD SHIP" and "HONGKONG GODOWN":

```none
141 P. TAB(0,3); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3);CHR$151; "GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(25,3);"HONGKONG";TAB(34,3);"GODOWN"
150 FORI=0TO5:P.TAB(0,4+I);G$(I):P.TAB(7,4+I);CHR$151;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330:P.TAB(22,4+I);CHR$151;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:NEXTI:P.CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:RETURN
```

[![BBCTaipan_MODE7_GoodsHeld_TextBlocksTwoBar_Pipe_MoreBlocks][29]][29]

Meh.

TODO: Need to add blocks around the market prices heading text, to make it consistent.

### Market prices

Need to add inverse (or colour) to line 220:

```none
220 GOSUB 790: GOSUB 1340:INVERSE=0:PRINT TAB(7,10); " ";L$(L);" MARKET PRICES ":NORMAL=0:PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(0,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);
```

[![BBCTaipan_MODE7_MarketPrices_Plain][30]][30]

#### Using normal text block

```none
220 GOSUB 790: GOSUB 1340:P. TAB(0,10);:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(7,10); " ";L$(L);" MARKET PRICES ":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(0,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);
```

[![BBCTaipan_MODE7_MarketPrices_TextBlock][31]][31]

#### Using graphics blocks


```none
220 GOSUB 790: GOSUB 1340:P. TAB(0,10); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(7,10); " ";L$(L);" MARKET PRICES ":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(0,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);
```

[![BBCTaipan_MODE7_MarketPrices_GraphicsBlock][32]][32]

Not too bad looking, considering. But the indent on the left hand side is just weird.

### Character set

```none
10 FOR N=33 TO 255: PRINT CHR$(N);: NEXT N
10 FOR N=33 TO 255
20 PRINT CHR$(N);
30 NEXT N
```

[![BBCTaipan_MODE7_CharacterSet_1][100]][100]

```none
10 FOR N=128 TO 175: PRINT CHR$(N);: NEXT N
10 FOR N=120 TO 185: PRINT CHR$(N);: NEXT N
10 FOR N=110 TO 185: PRINT CHR$(N);: NEXT N
```

[![BBCTaipan_MODE7_CharacterSet_2][101]][101]


```none
10 FOR N=33 TO 255: PRINT CHR$(N);: NEXT N
15 PRINT
20 FOR N=113 TO 195: PRINT CHR$(N);: NEXT N: REM PRINTS UNTIL "C" on the next line
```

[![BBCTaipan_MODE7_CharacterSet_3][102]][102]


```none
20 FOR N=113 TO 150: PRINT CHR$(N);: NEXT N: REM Gives blue block
20 FOR N=113 TO 155: PRINT CHR$(N);: NEXT N: REM Gives blank line
20 FOR N=113 TO 156: PRINT CHR$(N);: NEXT N: REM Gives blank line
20 FOR N=113 TO 157: PRINT CHR$(N);: NEXT N: REM Minimal repro of white bar
```

[![BBCTaipan_MODE7_CharacterSet_4][103]][103]

### Reproducing the splash screen

The original bare bones version:

```none
10 HOME=0:VDU12:A$ = "                                        ":W$ = "ELDER BROTHER WU":LY$ = "LI YUEN":YS$ = "YANATO & SMYTHE":TC$ = "O, S, T, A, P, OR R"
11 B$="                                       "
20 INVERSE=0:PRINT TAB(0,3); A$:PRINT TAB(12); "T A I P A N:":NORMAL=0: PRINT TAB(12);"_________________"
21 SPEED = 100:PRINT TAB(13,8);"A  G A M E  I N":PRINT TAB( 14);"C O N T E X T": PRINT:PRINT TAB( 13);"HAYDEN BOOK CO."
22 SPEED = 255:INVERSE=0:PRINT TAB(0,14); A$:NORMAL=0
```

[![BBCTaipan_MODE7_SplashScreenReproduction_Plain][104]][104]

becomes

```none
10 HOME=0:VDU12:A$ = "                                        ":W$ = "ELDER BROTHER WU":LY$ = "LI YUEN":YS$ = "YANATO & SMYTHE":TC$ = "O, S, T, A, P, OR R"
11 B$="                                       "
20 PRINT TAB(0,3): FOR N=1 TO 40: PRINT CHR$255;:NEXT N: PRINT TAB(12); "T A I P A N:": PRINT TAB(12);"_________________"
21 SPEED = 100:PRINT TAB(13,8);"A  G A M E  I N":PRINT TAB( 14);"C O N T E X T": PRINT:PRINT TAB( 13);"HAYDEN BOOK CO."
22 SPEED = 255:PRINT TAB(0,14);:FOR N=1 TO 40: PRINT CHR$255;:NEXT N
```

[![BBCTaipan_MODE7_SplashScreenReproduction_TextBlocks][105]][105]

or, using the graphics characters (and indenting the bar on left (due to control character ) and on the right (to ensure symmetry)

```none
10 HOME=0:VDU12:A$ = "                                        ":W$ = "ELDER BROTHER WU":LY$ = "LI YUEN":YS$ = "YANATO & SMYTHE":TC$ = "O, S, T, A, P, OR R"
11 B$="                                       "
20 PRINT TAB(0,3);CHR$151;:FOR N=1 TO 37: PRINT CHR$255;: NEXT N: PRINT CHR$255:PRINT: PRINT TAB(12); "T A I P A N:": PRINT TAB(12);"_________________"
21 SPEED = 100:PRINT TAB(13,8);"A  G A M E  I N":PRINT TAB( 14);"C O N T E X T": PRINT:PRINT TAB( 13);"HAYDEN BOOK CO."
22 SPEED = 255:PRINT TAB(0,14);CHR$151;:FOR N=1 TO 37: PRINT CHR$255;: NEXT N: PRINT CHR$255
```

[![BBCTaipan_MODE7_SplashScreenReproduction_GraphicsBlocks_Bad][106]][106]

But the `<` and the `>` in the "PRESS < SPACEBAR > " message get corrupted, so to fix:

```none
590 PRINT TAB(8,14);CHR$135"PRESS <";:FLASH=1:PRINT CHR$136"SPACEBAR"CHR$137;:NORMAL=1:PRINT "> TO START"; : GOSUB 60:IF X$ = " " THEN RETURN
```

[![BBCTaipan_MODE7_SplashScreenReproduction_GraphicsBlocks_Bad2][107]][107]

But now the last three blocks are text mode blocks, so add a return to graphics mode

```none
590 PRINT TAB(8,14);CHR$135"PRESS <";:FLASH=1:PRINT CHR$136"SPACEBAR"CHR$137;:NORMAL=1:PRINT "> TO START";CHR$151 : GOSUB 60:IF X$ = " " THEN RETURN
```

[![BBCTaipan_MODE7_SplashScreenReproduction_GraphicsBlocks_OK][108]][108]

Meh, looks ok, but not centered. See **Splash screen seems a bit off center** section below, for centering.

### Using colour (for the table)

From page 129 of the BBC User Guide. v1.00, blue on yellow

```none
10 MODE 7
20 PRINT CHR$(131);CHR$(157);CHR$(132);"BLUE LETTERS ON YELLOW"
```

[![BBCTaipan_MODE7_BlueOnYellow][109]][109]

This gives a "bar" with inverted text. However, there is still:

 - The indented column, of the background, on the left, due to the control character
 - The text starts indented by three characters

Would the colour of the whole UI text need to change?

#### Right shift by three characters

There is another issue:

```none
10 MODE 7
20 PRINT TAB(0,3);CHR$(131);CHR$(157);CHR$(132); "GOODS     ABOARD SHIP    HONGKONG GODOWN"
```

[![BBCTaipan_MODE7_BlueOnYellow_ThreeShiftRight][110]][110]

Notes:

 - The heading text is indented and shifted *three* (!) characters to the right, so "OWN" is on the next line..! This is due to the three control characters. 
 - The background is only indented by one

It could be better to keep white characters and use a blue background? But then it isn't inverted. Maybe use green on black (like a monochrome Apple II) and then can invert with black on green. But then, still three/two?<sup>1</sup>  characters will be needed, and the whole screen will have to be shifted right by three/two?<sup>1</sup>  characters, such that the origin is at (2,0).

<sup>1</sup> two?  Blackground is still black most of the time so, one character for green. Except when inverted, then it will be three charactes. This is a nonsense system to work with!

Maybe use line graphics instead? See **Using line graphic (not blocks)** below.

#### Blue and Yellow table

Making the whole table blue with yellow background. Line 150 will need to be split as it is already at maximum capacity. Line 141 can be reverted back to the original line format:

```none
141 PRINT TAB(0,3);CHR$(131);CHR$(157);CHR$(132); "GOODS     ABOARD SHIP    HONGKONG GODOWN"
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);CHR$131;G$(I):P.TAB(7,4+I);CHR$147;CHR$181;CHR$131:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330:P.TAB(22,4+I);CHR$147;CHR$181;CHR$131:PRINT TAB(27,4+I);:Q=GG(I):GOSUB 1330:NEXTI
```

[![BBCTaipan_MODE7_BlueOnYellow_MainTable][111]][111]

Should the goods and values still be in white?

```none
141 PRINT TAB(0,3);CHR$(131);CHR$(157);CHR$(132); "GOODS     ABOARD SHIP    HONGKONG GODOWN"
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);CHR$135;G$(I):P.TAB(7,4+I);CHR$147;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330:P.TAB(22,4+I);CHR$147;CHR$181;CHR$135:PRINT TAB(27,4+I);:Q=GG(I):GOSUB 1330:NEXTI
```

[![BBCTaipan_MODE7_BlueOnYellow_MainTable_GoodsWhite][112]][112]

#### Green on Black table (like Apple II monochrome)

Not possible as black text is not allowed, so use blue instead


```none
141 PRINT TAB(0,3);CHR$(130);CHR$(157);CHR$(132);"GOODS     ABOARD SHIP    HONGKONG GODOWN"
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);CHR$130;G$(I):P.TAB(7,4+I);CHR$146;CHR$181;CHR$130:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330:P.TAB(22,4+I);CHR$146;CHR$181;CHR$130:PRINT TAB(27,4+I);:Q=GG(I):GOSUB 1330:NEXTI
```

[![BBCTaipan_MODE7_GreenOnBlack_MainTable][113]][113]

Should the goods and values still be in white?


```none
141 PRINT TAB(0,3);CHR$(130);CHR$(157);CHR$(132);"GOODS     ABOARD SHIP    HONGKONG GODOWN"
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);CHR$135;G$(I):P.TAB(7,4+I);CHR$146;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330:P.TAB(22,4+I);CHR$146;CHR$181;CHR$135:PRINT TAB(27,4+I);:Q=GG(I):GOSUB 1330:NEXTI
```

[![BBCTaipan_MODE7_GreenOnBlack_MainTable_GoodsWhite][114]][114]

### Using colour (for the Market Prices)

#### Blue and Yellow market prices

```none
220 GOSUB 790: GOSUB 1340:PRINT TAB(7,10);CHR$(131);CHR$(157);CHR$(132);" ";L$(L);" MARKET PRICES ":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(0,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);
```

[![BBCTaipan_MODE7_BlueOnYellow_MarketPrices_Bad][115]][115]

Looks better with prefix spaces

```none
220 GOSUB 790: GOSUB 1340:PRINT TAB(0,10);CHR$(131);CHR$(157);CHR$(132);"        ";L$(L);" MARKET PRICES ":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(0,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);
```

[![BBCTaipan_MODE7_BlueOnYellow_MarketPrices_PrefixSpaces][116]][116]

#### Green on Black (like Apple II monochrome) market prices

Not possible as black text is not allowed

```none
220 GOSUB 790: GOSUB 1340:PRINT TAB(0,10);CHR$(130);CHR$(157);CHR$(128);"        ";L$(L);" MARKET PRICES ":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(0,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);
```

[![BBCTaipan_MODE7_GreenOnBlack_MarketPrices_Bad][117]][117]

So use blue?

```none
220 GOSUB 790: GOSUB 1340:PRINT TAB(0,10);CHR$(130);CHR$(157);CHR$(132);"        ";L$(L);" MARKET PRICES ":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(0,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);
```

[![BBCTaipan_MODE7_GreenOnBlack_MarketPrices][118]][118]

Not too bad - still left hand indent of one character

### Using line graphics (not blocks)

TODO: Fill this



## Interim test UI patches

### Black and White

```none
10 HOME=0:VDU12:A$ = "                                        ":W$ = "ELDER BROTHER WU":LY$ = "LI YUEN":YS$ = "YANATO & SMYTHE":TC$ = "O, S, T, A, P, OR R"
11 B$="                                       "
20 PRINT TAB(0,3);CHR$151;:FOR N=1 TO 37: PRINT CHR$255;: NEXT N: PRINT CHR$255:PRINT: PRINT TAB(12); "T A I P A N:": PRINT TAB(12);"_________________"
21 SPEED = 100:PRINT TAB(13,8);"A  G A M E  I N":PRINT TAB( 14);"C O N T E X T": PRINT:PRINT TAB( 13);"HAYDEN BOOK CO."
22 SPEED = 255:PRINT TAB(0,14);CHR$151;:FOR N=1 TO 37: PRINT CHR$255;: NEXT N: PRINT CHR$255

141 P. TAB(0,3); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3);CHR$151; "GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(25,3);"HONGKONG";TAB(34,3);"GODOWN"
150 FORI=0TO5:P.TAB(0,4+I);G$(I):P.TAB(7,4+I);CHR$151;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330:P.TAB(22,4+I);CHR$151;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:NEXTI:P.CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:RETURN

220 GOSUB 790: GOSUB 1340:P. TAB(0,10); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(7,10); " ";L$(L);" MARKET PRICES ":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(0,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);

590 PRINT TAB(8,14);CHR$135"PRESS <";:FLASH=1:PRINT CHR$136"SPACEBAR"CHR$137;:NORMAL=1:PRINT "> TO START";CHR$151 : GOSUB 60:IF X$ = " " THEN RETURN
```

TODO: Query: Line 141 - Should the second CHR$151 actually be CHR$135?

### Colour

#### Blue on Yellow

```none
141 PRINT TAB(0,3);CHR$(131);CHR$(157);CHR$(132); "GOODS     ABOARD SHIP    HONGKONG GODOWN"
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);CHR$131;G$(I):P.TAB(7,4+I);CHR$147;CHR$181;CHR$131:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330:P.TAB(22,4+I);CHR$147;CHR$181;CHR$131:PRINT TAB(27,4+I);:Q=GG(I):GOSUB 1330:NEXTI

220 GOSUB 790: GOSUB 1340:PRINT TAB(0,10);CHR$(131);CHR$(157);CHR$(132);"        ";L$(L);" MARKET PRICES ":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(0,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);
```

##### Goods/values in white text

```none
141 PRINT TAB(0,3);CHR$(131);CHR$(157);CHR$(132); "GOODS     ABOARD SHIP    HONGKONG GODOWN"
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);CHR$135;G$(I):P.TAB(7,4+I);CHR$147;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330:P.TAB(22,4+I);CHR$147;CHR$181;CHR$135:PRINT TAB(27,4+I);:Q=GG(I):GOSUB 1330:NEXTI

220 GOSUB 790: GOSUB 1340:PRINT TAB(0,10);CHR$(131);CHR$(157);CHR$(132);"        ";L$(L);" MARKET PRICES ":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(0,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);
```

#### Green/Blue

```none
141 PRINT TAB(0,3);CHR$(130);CHR$(157);CHR$(132);"GOODS     ABOARD SHIP    HONGKONG GODOWN"
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);CHR$130;G$(I):P.TAB(7,4+I);CHR$146;CHR$181;CHR$130:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330:P.TAB(22,4+I);CHR$146;CHR$181;CHR$130:PRINT TAB(27,4+I);:Q=GG(I):GOSUB 1330:NEXTI

220 GOSUB 790: GOSUB 1340:PRINT TAB(0,10);CHR$(130);CHR$(157);CHR$(132);"        ";L$(L);" MARKET PRICES ":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(0,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);
```

##### Goods/values in white text

```none
141 PRINT TAB(0,3);CHR$(130);CHR$(157);CHR$(132);"GOODS     ABOARD SHIP    HONGKONG GODOWN"
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);CHR$135;G$(I):P.TAB(7,4+I);CHR$146;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330:P.TAB(22,4+I);CHR$146;CHR$181;CHR$135:PRINT TAB(27,4+I);:Q=GG(I):GOSUB 1330:NEXTI

220 GOSUB 790: GOSUB 1340:PRINT TAB(0,10);CHR$(130);CHR$(157);CHR$(132);"        ";L$(L);" MARKET PRICES ":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(0,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);
```

## More hold characters (`CHR$158`)

Maybe use the hold character to avoid blanks in column 0? See [Mode 7 teleetxt graphics question](https://www.stardot.org.uk/forums/viewtopic.php?t=21572).

As [this post](https://www.stardot.org.uk/forums/viewtopic.php?p=305507&sid=354c0a51ad31246e578ad68313e0134f#p305507) states, it would have been much easier to use 2 bytes per character, with the second byte as attributes. However, this was intended to be as lightweight as possible for broadcasting over the airwaves, so why double to bandwidth, just to make programmers' lives easier?

From the same post, 

```none
VDU 145,255, 158,146,255, 158,147,255, 158,148,255, 158,149,255, 158,150,255, 158,151,255,255,255 10,13
```
Note: There is *still* a black character at the start of the line.

[![BBCTaipan_MODE7_MoreHoldCharacters_Post1][950]][950]

Shouldn't there be an extra comma before the penultimate byte?
```none
VDU 145,255, 158,146,255, 158,147,255, 158,148,255, 158,149,255, 158,150,255, 158,151,255,255,255, 10,13
```

Meh, doesn't make much difference.

From [Introduction to VDU commands - VDU code summary](https://www.bbcbasic.co.uk/bbcwin/manual/bbcwin8.html), 10 is next line and 13 start of line

Making this multi-line

```none
5 CLS
10 VDU 145,255, 158,146,255, 158,147,255, 158,148,255, 158,149,255, 158,150,255, 158,151,255,255,255 10,13
20 VDU 145,255, 158,146,255, 158,147,255, 158,148,255, 158,149,255, 158,150,255, 158,151,255,255,255 10,13
30 PRINT
40 VDU145,255, 158,146,255, 158,147,255, 158,148,255, 158,149,255, 158,150,255, 158,151,255,255,255, 145,255, 158,146,255, 158,147,255, 158,148,255, 158,149,255, 158,150,255, 158,151,255,255,255, 145,255, 158,146,255, 158,147,255, 255,255
50 PRINT
60 VDU145,255, 158,146,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255
70 PRINT
80 VDU145,255, 158,146,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,145,255,255,255,255,255,255,255,255,255,255,255,255,255,255
85 PRINT
90 VDU145,255, 158,146,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255: REM One block spillover
100 PRINT
```

Note: There is *still* a black character at the start of the line. Line 60 shows that even if you overrun, then the blocks just get "outlined" – i.e. they turn into text blocks on the following line. The hold character does not work on the first character of the line. Line 80 shows that the last `145` is not displayed as a hold character, in column 0.

In order words, the hold character seems to need to be reset for *each* line. You can't set it on a preceding and expect it to work as the first character of the next line, unfortunately.

[![BBCTaipan_MODE7_MoreHoldCharacters_VDU][951]][951]

### MRE: Dummy shopping list table in Apple II and BBC

Apple II shopping list

```none
10 HOME
20 INVERSE: PRINT "                                        ": NORMAL
30 VTAB1:HTAB1:INVERSE: PRINT "SHOPPING LIST": NORMAL
40 PRINT "SAUSAGES"
50 PRINT "EGGS"
60 PRINT "SKIMMED MILK"
70 PRINT "MILK"
80 INVERSE: PRINT "                                        ": NORMAL
```

[![BBCTaipan_MODE7_AppleMREShoppingList][952]][952]

BBC shopping list

```none
5 MODE 7
10 CLS
20 P.CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN
30 PRINT TAB(0,0);CHR$151;"SHOPPING LIST"
40 PRINT "SAUSAGES"
50 PRINT "EGGS"
60 PRINT "SKIMMED MILK"
70 PRINT "MILK"
80 P.CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN: P.CHR$255
```

[![BBCTaipan_MODE7_BBCMREShoppingList][953]][953]

## A different MODE?

MODE 7 is clearly useless, as the first column is lost.

From page 50, BBC User Guide.

```none
10 MODE 5
20 VDU 24, 0; 0; 500; 1000;
30 VDU 28,10,20,19,5
40 COLOUR 129
50 COLOUR 2
60 GCOL 0,130
70 CLS: CLG
80 FOR N = 1 TO 1000
90 PRINT "LINE"; N
100 GCOL 0, RND(4)
110 DRAW RND(500), RND(1000)
120 NEXT N
```

[![BBCTaipan_MODE7_MODE5Example][966]][966]

As can be seen in the above image, the text in `MODE 5` is awful, barely useable. `MODE 4` (and 1 and 6) has text more similar to (but not the same as) `MODE 7` – Modes 1 (40x32), 4 (40x32) and 6 (40x25) are the only 40 column modes, apart from `MODE 7` (40x25). MODE 6 looks more clunky. The Apple II resolution is 40x24, which is most similar to the more clunky looking `MODE 6` (and prettier `MODE 7`).

See [BBC BASIC for SDL 2.0 - BBC BASIC for Windows - Graphics and Colours](https://www.bbcbasic.co.uk/bbcwin/manual/bbcwin3.html) for the list of resolutions.

### Equivalents of `NORMAL` and `INVERSE`

For modes 1-6:

 - White Text/Black Background: `COLOUR 3: COLOUR 128`
 - Black Text/White Background: `COLOUR 0: COLOUR 131`  <---- inverse

### The shopping list MRE

#### In `MODE 4`

```none
5 MODE 4: REM MODE 1, 4 or 6
10 CLS
20 COLOUR 0: COLOUR 131
30 PRINT TAB(0,0);"SHOPPING LIST"
40 COLOUR 3: COLOUR 128
50 PRINT "SAUSAGES"
60 PRINT "EGGS"
70 PRINT "SKIMMED MILK"
80 PRINT "MILK"
90 COLOUR 0: COLOUR 131: PRINT "                                        "
100 COLOUR 3: COLOUR 128
```

[![BBCTaipan_MODE7_BBCMREShoppingList_MODE4][967]][967]

And overlaying headings, by changing line 20

```none
5 MODE 4: REM MODE 1, 4 or 6
10 CLS
20 COLOUR 0: COLOUR 131: PRINT "                                        "
30 PRINT TAB(0,0);"SHOPPING LIST"
40 COLOUR 3: COLOUR 128
50 PRINT "SAUSAGES"
60 PRINT "EGGS"
70 PRINT "SKIMMED MILK"
80 PRINT "MILK"
90 COLOUR 0: COLOUR 131: PRINT "                                        "
100 COLOUR 3: COLOUR 128
```

[![BBCTaipan_MODE7_BBCMREShoppingList_MODE4_Overlay][968]][968]

or just not changing line 20, and centering the heading along with padding spaces in line 30:

```none
5 MODE 4: REM MODE 1, 4 or 6
10 CLS
20 COLOUR 0: COLOUR 131
30 PRINT TAB(0,0);      "              SHOPPING LIST             "
40 COLOUR 3: COLOUR 128
50 PRINT "SAUSAGES"
60 PRINT "EGGS"
70 PRINT "SKIMMED MILK"
80 PRINT "MILK"
90 COLOUR 0: COLOUR 131: PRINT "                                        "
100 COLOUR 3: COLOUR 128
```

[![BBCTaipan_MODE7_BBCMREShoppingList_MODE4_Center][969]][969]

The text font of `MODE 4` looks rather clunky, as does `MODE1` (there is no diserible difference really between the font of modes 1 and 4), and `MODE 6` is even more "*clunkier looking*" still, when compared to the font used in `MODE 7` – even though the 40x25 character resolution of `MODE 6` (and `MODE 7`) more closely matches the screen resolution of the Apple II, which is 40x24.

See [BBC BASIC for SDL 2.0 - BBC BASIC for Windows - Graphics and Colours](https://www.bbcbasic.co.uk/bbcwin/manual/bbcwin3.html) for the list of resolutions.

#### In `MODE 1`

[![BBCTaipan_MODE7_BBCMREShoppingList_MODE1_Center][970]][970]


#### In `MODE 6`

[![BBCTaipan_MODE7_BBCMREShoppingList_MODE6_Center][971]][971]

### Splash screen

However, another issue becomes painfully apparent, as soon as the `MODE` change, and other changes, are patched in:

```none
6 MODE 6
10 HOME=0:VDU12:A$ = "                                        ":W$ = "ELDER BROTHER WU":LY$ = "LI YUEN":YS$ = "YANATO & SMYTHE":TC$ = "O, S, T, A, P, OR R"
11 B$="                                       "

20 COLOUR 0: COLOUR 131:PRINT TAB(0,3); A$:PRINT TAB(12); "T A I P A N:":COLOUR 3: COLOUR 128: PRINT TAB(12);"_________________"
21 SPEED = 100:PRINT TAB(13,8);"A  G A M E  I N":PRINT TAB( 14);"C O N T E X T": PRINT:PRINT TAB( 13);"HAYDEN BOOK CO."
22 SPEED = 255:COLOUR 0: COLOUR 131:PRINT TAB(0,14); A$:COLOUR 3: COLOUR 128
```

[![BBCTaipan_MODE7_MODE6_DIM][970]][970]

Using `MODE 6` results in a `DIM space at line 30` error. Not enough memory! Both `MODE 1` and `MODE 4` give `Bad MODE` error. 

[![BBCTaipan_MODE7_MODE1_BadMODE][973]][973]

However, `MODE 6` uses the same amount of memory as `MODE 7`, which is 8 kB (as does `MODE 4`, whereas `MODE 1` uses 16 kB).

From [BBC BASIC memory usage](https://stardot.org.uk/forums/viewtopic.php?t=10309), `PRINT HIMEM-LOMEM` gives 9583, upon reset and loading from disk and 8559 after running the game.

See also [BASIC memory usage](https://beebwiki.mdfs.net/BASIC_memory_usage)

## 39 Columns: Stuck with `MODE 7` - and the indent

So it seems as if we are stuck with using `MODE 7`, as `MODE 1` and `MODE4` give `Bad MODE` errors and `MODE 6` has not enough memory to DIM an array. We are stuck with `MODE 7` and the missing column 0 – so maybe indent everything else that isn't graphics?

Using the code from **Interim test UI patches - Black and White**

```none
10 HOME=0:VDU12:A$ = "                                        ":W$ = "ELDER BROTHER WU":LY$ = "LI YUEN":YS$ = "YANATO & SMYTHE":TC$ = "O, S, T, A, P, OR R"
11 B$="                                       "
20 PRINT TAB(0,3);CHR$151;:FOR N=1 TO 37: PRINT CHR$255;: NEXT N: PRINT CHR$255:PRINT: PRINT TAB(12); "T A I P A N:": PRINT TAB(12);"_________________"
21 SPEED = 100:PRINT TAB(13,8);"A  G A M E  I N":PRINT TAB( 14);"C O N T E X T": PRINT:PRINT TAB( 13);"HAYDEN BOOK CO."
22 SPEED = 255:PRINT TAB(0,14);CHR$151;:FOR N=1 TO 37: PRINT CHR$255;: NEXT N: PRINT CHR$255

141 P. TAB(0,3); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3);CHR$151; "GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(25,3);"HONGKONG";TAB(34,3);"GODOWN"
150 FORI=0TO5:P.TAB(0,4+I);G$(I):P.TAB(7,4+I);CHR$151;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330:P.TAB(22,4+I);CHR$151;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:NEXTI:P.CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:RETURN

220 GOSUB 790: GOSUB 1340:P. TAB(0,10); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(7,10); " ";L$(L);" MARKET PRICES ":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(0,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);

590 PRINT TAB(8,14);CHR$135"PRESS <";:FLASH=1:PRINT CHR$136"SPACEBAR"CHR$137;:NORMAL=1:PRINT "> TO START";CHR$151 : GOSUB 60:IF X$ = " " THEN RETURN
```

and changing line 150 to indent the goods labels

```none
150 FORI=0TO5:P.TAB(1,4+I);G$(I):P.TAB(7,4+I);CHR$151;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330:P.TAB(22,4+I);CHR$151;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:NEXTI:P.CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:RETURN
```

[![BBCTaipan_MODE7_MainScreen_Asymmetrical_GoodsOnlyFixed][1000]][1000]

and also indenting the player's stats, above the table, in lines 130-140

```none
130 VDU12:PRINT TAB(1,0); "PORT ";L$(L);: PRINT TAB(27,0);M$(M);". ";DA+1;",";Y
140 INVERSE=0:PRINT TAB(1,1); "CASH ";: Q = C:GOSUB 1330:NORMAL=0:PRINT TAB(27,1); "GUNS ";:Q=G:GOSUB 1330: PRINT TAB(1,2); "DEBT ";: Q = D:GOSUB 1330:PRINT TAB(27,2); "HOLD ";: Q = SH:GOSUB 1330
```

[![BBCTaipan_MODE7_MainScreen_Asymmetrical_GoodsStatsFixed][1001]][1001]

Also, the temple donation

```none
830 IF C > 100 AND TR = 0AND L = 0 THEN GOSUB 1340:PRINT TAB(1,11);YS$;", LIEUTENANTS":PRINT "OF THE MARINER, ";LY$;", ASK IF": PRINT "YOU WILL DONATE ";:Q = DN:GOSUB 1330:PRINT "TO THE TEMPLE OF TIN HAU, THE"
```

but additional `TAB` are required, for all of the printed lines

```none
830 IF C > 100 AND TR = 0AND L = 0 THEN GOSUB 1340:PRINT TAB(1,11);YS$;", LIEUTENANTS":PRINT TAB(1,12);"OF THE MARINER, ";LY$;", ASK IF": PRINT TAB(1,13);"YOU WILL DONATE ";:Q = DN:GOSUB 1330:PRINT TAB(1,14);"TO THE TEMPLE OF TIN HAU, THE"
```

Too long! Shortening (and changing line 831 too):

```none
830 IF C > 100 AND TR = 0AND L = 0 THEN GOSUB 1340:P. TAB(1,11);YS$;", LIEUTENANTS":P. TAB(1,12);"OF THE MARINER, ";LY$;", ASK IF": P. TAB(1,13);"YOU WILL DONATE ";:Q = DN:GOSUB 1330:P. TAB(1,14);"TO THE TEMPLE OF TIN HAU, THE"
831 IF C > 100 AND TR = 0 AND L = 0THEN PRINT TAB(1,15);"SEA GODDESS. (Y) OR (N)"
```

Don't forget the Y/N response of Y&S!

```none
840 IF C > 0 AND TR = 0 AND L = 0 THEN GOSUB 60: IF X$ = "Y" THEN C = C - DN:TR = 1:GOSUB 130:GOSUB 1340:PRINT TAB(1,11);" ":PRINT " " ;YS$;" THANK": PRINT TAB(1,12);" YOU, AND DEPART.": PRINT:PRINT A$;
841 IF C > 0 AND TR = 0 AND L = 0THEN IF X$ <> "Y" THEN GOSUB 1340: PRINT TAB(1,11); " ":PRINT TAB(1,12);" ";YS$;" DEPART":PRINT TAB(1,13);"ABRUPTLY IN A CHILLY SILENCE.": PRINT A$;
```

[![BBCTaipan_MODE7_MainScreen_Asymmetrical_GoodsStatsTempleFixed][1002]][1002]

Also the market prices

```none
220 GOSUB 790: GOSUB 1340:INVERSE=0:PRINT TAB(7,10); " ";L$(L);" MARKET PRICES ":NORMAL=0:PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(1,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);

230 PRINT: PRINT TAB(1,17) " B)UY, S)ELL, L)EAVE, OR R)ETIRE?" 
```

[![BBCTaipan_MODE7_MainScreen_Asymmetrical_GoodsStatsTempleMarketFixed][1003]][1003]

Also the Lender

```none
```

Also the XXXX

```none
```

Even though, it is now symmetical, with the fully indented 39 column screen, the table in the main display *still* just looks ***awful***. This is mainly because it is not possible to invert the characters in the table headings/title.

There is no *real* need (except for one of consistency) to change the x coordinates of other messages, on other screens, if there is no inverse, nor graphics characters employed there.

### Splash screen seems a bit off center

Changing line 590
 
```none
590 PRINT TAB(8,14);CHR$135"PRESS <";:FLASH=1:PRINT CHR$136"SPACEBAR"CHR$137;:NORMAL=1:PRINT "> TO START";CHR$151 : GOSUB 60:IF X$ = " " THEN RETURN
```

[![BBCTaipan_MODE7_SplashNotCentered][1020]][1020]

And shifting left by two characters is much better:

```none
590 PRINT TAB(6,14);CHR$135"PRESS <";:FLASH=1:PRINT CHR$136"SPACEBAR"CHR$137;:NORMAL=1:PRINT "> TO START";CHR$151 : GOSUB 60:IF X$ = " " THEN RETURN
```

[![BBCTaipan_MODE7_SplashCentered][1021]][1021]

## Full UI patch

### Black and White - Indented

```none
10 HOME=0:VDU12:A$ = "                                        ":W$ = "ELDER BROTHER WU":LY$ = "LI YUEN":YS$ = "YANATO & SMYTHE":TC$ = "O, S, T, A, P, OR R"
11 B$="                                       "
20 PRINT TAB(0,3);CHR$151;:FOR N=1 TO 37: PRINT CHR$255;: NEXT N: PRINT CHR$255:PRINT: PRINT TAB(12); "T A I P A N:": PRINT TAB(12);"_________________"
21 SPEED = 100:PRINT TAB(13,8);"A  G A M E  I N":PRINT TAB( 14);"C O N T E X T": PRINT:PRINT TAB( 13);"HAYDEN BOOK CO."
22 SPEED = 255:PRINT TAB(0,14);CHR$151;:FOR N=1 TO 37: PRINT CHR$255;: NEXT N: PRINT CHR$255


130 VDU12:PRINT TAB(1,0); "PORT ";L$(L);: PRINT TAB(27,0);M$(M);". ";DA+1;",";Y
140 INVERSE=0:PRINT TAB(1,1); "CASH ";: Q = C:GOSUB 1330:NORMAL=0:PRINT TAB(27,1); "GUNS ";:Q=G:GOSUB 1330: PRINT TAB(1,2); "DEBT ";: Q = D:GOSUB 1330:PRINT TAB(27,2); "HOLD ";: Q = SH:GOSUB 1330
141 P. TAB(0,3); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3);CHR$151; "GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(25,3);"HONGKONG";TAB(34,3);"GODOWN"

150 FORI=0TO5:P.TAB(1,4+I);G$(I):P.TAB(7,4+I);CHR$151;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330:P.TAB(22,4+I);CHR$151;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:NEXTI:P.CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:RETURN


220 GOSUB 790: GOSUB 1340:INVERSE=0:PRINT TAB(7,10); " ";L$(L);" MARKET PRICES ":NORMAL=0:PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(1,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);

230 PRINT: PRINT TAB(1,17) " B)UY, S)ELL, L)EAVE, OR R)ETIRE?" 

590 PRINT TAB(6,14);CHR$135"PRESS <";:FLASH=1:PRINT CHR$136"SPACEBAR"CHR$137;:NORMAL=1:PRINT "> TO START";CHR$151 : GOSUB 60:IF X$ = " " THEN RETURN

830 IF C > 100 AND TR = 0AND L = 0 THEN GOSUB 1340:P. TAB(1,11);YS$;", LIEUTENANTS":P. TAB(1,12);"OF THE MARINER, ";LY$;", ASK IF": P. TAB(1,13);"YOU WILL DONATE ";:Q = DN:GOSUB 1330:P. TAB(1,14);"TO THE TEMPLE OF TIN HAU, THE"
831 IF C > 100 AND TR = 0 AND L = 0THEN PRINT TAB(1,15);"SEA GODDESS. (Y) OR (N)"

840 IF C > 0 AND TR = 0 AND L = 0 THEN GOSUB 60: IF X$ = "Y" THEN C = C - DN:TR = 1:GOSUB 130:GOSUB 1340:PRINT TAB(1,11);" ":PRINT " " ;YS$;" THANK": PRINT TAB(1,12);" YOU, AND DEPART.": PRINT:PRINT A$;
841 IF C > 0 AND TR = 0 AND L = 0THEN IF X$ <> "Y" THEN GOSUB 1340: PRINT TAB(1,11); " ":PRINT TAB(1,12);" ";YS$;" DEPART":PRINT TAB(1,13);"ABRUPTLY IN A CHILLY SILENCE.": PRINT A$;
```

 
This is the end, my friend

  [1]: https://i.sstatic.net/wiF5k7WY.png
  [2]: mode7images/BBCTaipan_MODE7_MainScreen_Asymmetrical_GoodsStatsTempleMarketFixed.png
  [3]: mode7images/BBCTaipan_MODE7_MainScreen_Asymmetrical.png
  [4]: mode7images/BBCTaipan_MODE7_FirstTestExample_MarketPrices_TextBlocks.png
  [5]: mode7images/BBCTaipan_MODE7_FirstTestExample_Overspill.png
  [6]: mode7images/BBCTaipan_MODE7_FirstTestExample_NoOverspill.png
  [7]: mode7images/BBCTaipan_MODE7_FirstTestExample_MarketPrices.png
  [8]: mode7images/BBCTaipan_MODE7_FirstTestExample_MarketPrices_TextBlocks.png
  [9]: mode7images/BBCTaipan_MODE7_SeparatedGraphics1.png
  [10]: mode7images/BBCTaipan_MODE7_SeparatedGraphics2.png
  [11]: mode7images/BBCTaipan_MODE7_HeldGraphics.png
  [12]: mode7images/BBCTaipan_MODE7_AnotherHeldGraphicsExample.png
  [13]: mode7images/BBCTaipan_MODE7_GoodsHeld_Plain.png
  [14]: mode7images/BBCTaipan_MODE7_GoodsHeld_BadFive.png
  [15]: mode7images/BBCTaipan_MODE7_GoodsHeld_BadMRE_1.png
  [16]: mode7images/BBCTaipan_MODE7_GoodsHeld_BadMRE_2.png
  [17]: mode7images/BBCTaipan_MODE7_GoodsHeld_BadMRE_3.png
  [18]: mode7images/BBCTaipan_MODE7_GoodsHeld_TextBlocks.png
  [19]: mode7images/BBCTaipan_MODE7_GoodsHeld_TextBlocksBar.png
  [20]: mode7images/BBCTaipan_MODE7_GoodsHeld_TextBlocksBar_Bad.png
  [21]: mode7images/BBCTaipan_MODE7_GoodsHeld_TextBlocksBar_Good.png
  [22]: mode7images/BBCTaipan_MODE7_GoodsHeld_TextBlocksBar_Pipe.png
  [23]: mode7images/BBCTaipan_MODE7_GoodsHeld_TextBlocksTwoBar_Pipe_Bad.png
  [24]: mode7images/BBCTaipan_MODE7_GoodsHeld_TextBlocksTwoBar_Pipe_Bad2.png
  [25]: mode7images/BBCTaipan_MODE7_GoodsHeld_TextBlocksTwoBar_Pipe_Better.png
  [26]: mode7images/BBCTaipan_MODE7_GoodsHeld_TextBlocksTwoBar_Pipe_Shift.png
  [27]: mode7images/BBCTaipan_MODE7_GoodsHeld_TextBlocksTwoBar_Pipe_RightIndent.png
  [28]: mode7images/BBCTaipan_MODE7_GoodsHeld_TextBlocksTwoBar_Pipe_ShiftAgain.png
  [29]: mode7images/BBCTaipan_MODE7_GoodsHeld_TextBlocksTwoBar_Pipe_MoreBlocks.png
  [30]: mode7images/BBCTaipan_MODE7_MarketPrices_Plain.png
  [31]: mode7images/BBCTaipan_MODE7_MarketPrices_TextBlock.png
  [32]: mode7images/BBCTaipan_MODE7_MarketPrices_GraphicsBlock.png
  [100]: mode7images/BBCTaipan_MODE7_CharacterSet_1.png
  [101]: mode7images/BBCTaipan_MODE7_CharacterSet_2.png
  [102]: mode7images/BBCTaipan_MODE7_CharacterSet_3.png
  [103]: mode7images/BBCTaipan_MODE7_CharacterSet_4.png
  [104]: mode7images/BBCTaipan_MODE7_SplashScreenReproduction_Plain.png
  [105]: mode7images/BBCTaipan_MODE7_SplashScreenReproduction_TextBlocks.png
  [106]: mode7images/BBCTaipan_MODE7_SplashScreenReproduction_GraphicsBlocks_Bad.png
  [107]: mode7images/BBCTaipan_MODE7_SplashScreenReproduction_GraphicsBlocks_Bad2.png
  [108]: mode7images/BBCTaipan_MODE7_SplashScreenReproduction_GraphicsBlocks_OK.png
  [109]: mode7images/BBCTaipan_MODE7_BlueOnYellow.png
  [110]: mode7images/BBCTaipan_MODE7_BlueOnYellow_ThreeShiftRight.png
  [111]: mode7images/BBCTaipan_MODE7_BlueOnYellow_MainTable.png
  [112]: mode7images/BBCTaipan_MODE7_BlueOnYellow_MainTable_GoodsWhite.png
  [113]: mode7images/BBCTaipan_MODE7_GreenOnBlack_MainTable_GoodsWhite.png
  [114]: mode7images/BBCTaipan_MODE7_GreenOnBlack_MainTable_GoodsWhite.png
  [115]: mode7images/BBCTaipan_MODE7_BlueOnYellow_MarketPrices_Bad.png
  [116]: mode7images/BBCTaipan_MODE7_BlueOnYellow_MarketPrices_PrefixSpaces.png
  [117]: mode7images/BBCTaipan_MODE7_GreenOnBlack_MarketPrices_Bad.png
  [118]: mode7images/BBCTaipan_MODE7_GreenOnBlack_MarketPrices.png
  [950]: mode7images/BBCTaipan_MODE7_MoreHoldCharacters_Post1.png
  [951]: mode7images/BBCTaipan_MODE7_MoreHoldCharacters_VDU.png
  [952]: mode7images/BBCTaipan_MODE7_AppleMREShoppingList.png
  [953]: mode7images/BBCTaipan_MODE7_BBCMREShoppingList.png
  [966]: mode7images/BBCTaipan_MODE7_MODE5Example.png
  [967]: mode7images/BBCTaipan_MODE7_BBCMREShoppingList_MODE4.png
  [968]: mode7images/BBCTaipan_MODE7_BBCMREShoppingList_MODE4_Overlay.png
  [969]: mode7images/BBCTaipan_MODE7_BBCMREShoppingList_MODE4_Center.png
  [970]: mode7images/BBCTaipan_MODE7_BBCMREShoppingList_MODE1_Center.png
  [971]: mode7images/BBCTaipan_MODE7_BBCMREShoppingList_MODE6_Center.png
  [972]: mode7images/BBCTaipan_MODE7_MODE6_DIM.png
  [973]: mode7images/BBCTaipan_MODE7_MODE1_BadMODE.png


  [1000]: mode7images/BBCTaipan_MODE7_MainScreen_Asymmetrical_GoodsOnlyFixed.png
  [1001]: mode7images/BBCTaipan_MODE7_MainScreen_Asymmetrical_GoodsStatsFixed.png
  [1002]: mode7images/BBCTaipan_MODE7_MainScreen_Asymmetrical_GoodsStatsTempleFixed.png
  [1003]: mode7images/BBCTaipan_MODE7_MainScreen_Asymmetrical_GoodsStatsTempleMarketFixed.png
  [1020]: mode7images/BBCTaipan_MODE7_SplashNotCentered.png
  [1021]: mode7images/BBCTaipan_MODE7_SplashCentered.png


## TODO

Tidy BBC floppy taipan
Add BBC Floppy image taipan
Update WOZ disk for Apple II taipan
