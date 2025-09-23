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

<sub>Apologies for its disjointed nature, but this page has been written, more or less, in chronicalogical order. As such, it describes a voyage of discovery (over the space of about four weeks, along with watching a **lot** of *Homicide:Life on the streets*), whilst I slowly got my head around the basic grphical options available in BBC BASIC, all the while (mostly) circling over, and eventually returning (i.e. resigning myself) to, `MODE 7`.</sub>

## Contents

 - BBCBASIC_BlockGraphicsDemo
   - [See also](#see-also)
   - [Links](#links)
   - [Notes](#notes)
     - [Initial foray](#initial-foray)
     - [Separated graphics - ugly](#separated-graphics-ugly)
     - [Held graphics](#)
       - [Another held graphics example](#)
     - [The Taipan main goods held table](#)
       - [Using normal text block]()
       - [Using graphics blocks](#)
         - [Using the graphics blocks \(`CHR$255`\) in normal mode instead of the thick pipe](#)
         - [Using the graphics thick pipe \(`CHR$181`/`CHR$234`\)](#)
     - [Market prices](#)
       - [Using normal text block](#)
       - [Using graphics blocks](#)
     - [Character set](#)
     - [Reproducing the splash screen](#)
     - [Using colour (for the table)](#)
       - [Right shift by three characters](#)
       - [Blue and Yellow table](#)
       - [Green on Black table (like Apple II monochrome)](#)
     - [Using colour (for the Market Prices)](#)
       - [Blue and Yellow market prices](#)
       - [Green on Black (like Apple II monochrome) market prices](#)
     - [Using line graphics (not blocks)](#)
   - [Interim test UI patches](#)
     - [Black and White](#)
     - [Colour](#)
       - [Blue on Yellow](#)
         -  [Goods/values in white text](#)
       - [Green/Blue](#)
         - [Goods/values in white text](#)
   - [More hold characters (`CHR$158`)](#)
     - [MRE: Dummy shopping list table in Apple II and BBC](#)
   - [A different MODE?](#)
     - [Equivalents of `NORMAL` and `INVERSE`](#)
     - [The shopping list MRE](#)
       - [In `MODE 4`](#)
       - [In `MODE 1`](#)
       - [In `MODE 6`](#)
     - [Splash screen](#)
   - [39 Columns: Stuck with `MODE 7` - and the indent](#)
     - [Splash screen seems a bit off center](#)
   - [Full UI patch](#)
     - [Black and White - Indented](#)
     
   - [TODO](#todo)  
     - [More TDOD](#more-todo)  
 
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

### Initial foray

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

Need to add blocks around the market prices heading text, to make it consistent. See **Two options: Questions about the blocks** below.

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

Not too bad - still left hand indent of one character – which, as we will see, is unavoidable.

### Using line graphics (not blocks)

TODO: Fill this???

Using which `MODE`?



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

Note: Line 141 - Should the second `CHR$151` actually be `CHR$135`? No, it doesn't *have* to be, as capital letter characters "punch through" the graphics mode. If number, or lower case, characters were to be printed then yes, the second `CHR$` *should* be `CHR$135`.

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
 - Black Text/White Background: `COLOUR 0: COLOUR 131`  ---- inverse

For mode 7:

 - Where would black text, black graphics control character fit? Respectively, `CHR$128` and `CHR$144` seem to be the locations, and they are unused control characters:
```none
10 P. CHR$(135)CHR$(157)CHR$(128)"HI"
```  

[![BCTaipan_MODE7_Inverse_Bad][1500]][1500]


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

[![BBCTaipan_MODE7_MODE6_DIM][972]][972]

Using `MODE 6` results in a `DIM space at line 30` error. Not enough memory! 

Both `MODE 1` and `MODE 4` give `Bad MODE` error:

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

### More indentation

Also indenting the player's stats, above the table, in lines 130-140

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

Also the market prices (and market menu)

```none
220 GOSUB 790: GOSUB 1340:INVERSE=0:PRINT TAB(7,10); " ";L$(L);" MARKET PRICES ":NORMAL=0:PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(1,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);

230 PRINT: PRINT TAB(1,17) " B)UY, S)ELL, L)EAVE, OR R)ETIRE?" 
```

[![BBCTaipan_MODE7_MainScreen_Asymmetrical_GoodsStatsTempleMarketFixed][1003]][1003]

TODO: Also the Lender

```none
470 GOSUB 130: GOSUB 1340:PRINT  TAB(1,11) "[WANCHAI DISTRICT OF HONGKONG: HOME OF "; W$;"]":PRINT " ";W$;" GREETS YOU, TAIPAN, AND WISHES YOU WELL.":GOSUB 780:PRINT  TAB(0,13)A$:PRINT A$
480 PRINT  TAB(1,13)"WU STATES THAT HIS IRON LOTUS": PRINT "TRIAD HAS BEEN WATCHING YOU.": GOSUB 780
490 PRINT  TAB(1,13)A$: PRINT A$: PRINT A$:PRINT TAB(1,13) W$;" ASKS, DO YOU":PRINT A$;:PRINT TAB(1,14) "WISH TO B)ORROW, P)AY, OR Q)UIT?    ";: NUM$ = ""

510 IF LD = 1 AND (B = 1 OR D > 1E4) THEN GOSUB 1340: PRINT TAB(0,14) "WU REGRETS THAT HE CANNOT LOAN": PRINT "YOU MORE AT THIS TIME, TAIPAN.";: GOSUB 760:GOTO 490

520 GOSUB 1340: PRINT TAB(0,14) "HOW MUCH DO YOU WISH TO ";LD$;",":PRINT "TAIPAN? ";:GOSUB 310:NUM = VAL (NUM$)

541 IF NUM > 2 * C THEN PRINT  TAB(0,14)A$:PRINT  TAB(1,14)W$;" REGRETS THAT HE":PRINT " CANNOT LOAN YOU THAT MUCH.";: GOSUB 760:GOTO 490

560 IF NUM > C THEN NUM = C: D = D - C:C=0:GOSUB 130:PRINT  TAB(1,14)W$;" THANKS YOU,":PRINT " TAIPAN, FOR THE PAYMENT.";: GOSUB 780:IF D < 0 THEN D = 0:GOSUB 130:GOTO 490

570 IF NUM > D THEN D = 0: C = C - NUM:PRINT  TAB(1,14) W$;" THANKS YOU FOR YOUR STARTLING GENEROSITY!";:GOSUB 130:GOSUB 780:GOTO 490
580 C = C - NUM: D = D - NUM:GOSUB 130:PRINT  TAB(1,14)W$;" ACCEPTS YOUR PAYMENT WITH GRATITUDE, TAIPAN   ";:GOSUB 780: GOTO 490
```

Also the Goods menu, move from 2 to 3???

```none
250 PRINT TAB(0,17)A$;:PRINT TAB(3,17) T$;" ";TC$;"?": GOSUB 260:GOTO 280
```

TODO: Also the XXXX

```none
```

Even though, it is now symmetical, with the fully indented 39 column screen, the table in the main display *still* just looks ***awful***. This is mainly because it is not possible to invert the characters in the table headings/title.

There is no *real* need (except for one of consistency) to change the x coordinates of other messages, on other screens, if there is no inverse, nor graphics characters employed there.

**IMPORTANT NOTE**: The indentation by `TAB()` from 0 to 1, is ***not*** necessary, if *every* string to be printed is preceded by a string containing a colour control chracter, i.e., `FG$=CHR$151`, then that indentation will occur automatically... see **Separated main screen left-hand indent - By prefix control string**

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

250 PRINT TAB(0,17)A$;:PRINT TAB(3,17) T$;" ";TC$;"?": GOSUB 260:GOTO 280

470 GOSUB 130: GOSUB 1340:PRINT  TAB(1,11) "[WANCHAI DISTRICT OF HONGKONG: HOME OF "; W$;"]":PRINT " ";W$;" GREETS YOU, TAIPAN, AND WISHES YOU WELL.":GOSUB 780:PRINT  TAB(0,13)A$:PRINT A$
480 PRINT  TAB(1,13)"WU STATES THAT HIS IRON LOTUS": PRINT "TRIAD HAS BEEN WATCHING YOU.": GOSUB 780
490 PRINT  TAB(1,13)A$: PRINT A$: PRINT A$:PRINT TAB(1,13) W$;" ASKS, DO YOU":PRINT A$;:PRINT TAB(1,14) "WISH TO B)ORROW, P)AY, OR Q)UIT?    ";: NUM$ = ""

510 IF LD = 1 AND (B = 1 OR D > 1E4) THEN GOSUB 1340: PRINT TAB(0,14) "WU REGRETS THAT HE CANNOT LOAN": PRINT "YOU MORE AT THIS TIME, TAIPAN.";: GOSUB 760:GOTO 490

520 GOSUB 1340: PRINT TAB(0,14) "HOW MUCH DO YOU WISH TO ";LD$;",":PRINT "TAIPAN? ";:GOSUB 310:NUM = VAL (NUM$)

541 IF NUM > 2 * C THEN PRINT  TAB(0,14)A$:PRINT  TAB(1,14)W$;" REGRETS THAT HE":PRINT " CANNOT LOAN YOU THAT MUCH.";: GOSUB 760:GOTO 490

560 IF NUM > C THEN NUM = C: D = D - C:C=0:GOSUB 130:PRINT  TAB(1,14)W$;" THANKS YOU,":PRINT " TAIPAN, FOR THE PAYMENT.";: GOSUB 780:IF D < 0 THEN D = 0:GOSUB 130:GOTO 490

570 IF NUM > D THEN D = 0: C = C - NUM:PRINT  TAB(1,14) W$;" THANKS YOU FOR YOUR STARTLING GENEROSITY!";:GOSUB 130:GOSUB 780:GOTO 490
580 C = C - NUM: D = D - NUM:GOSUB 130:PRINT  TAB(1,14)W$;" ACCEPTS YOUR PAYMENT WITH GRATITUDE, TAIPAN   ";:GOSUB 780: GOTO 490

590 PRINT TAB(6,14);CHR$135"PRESS <";:FLASH=1:PRINT CHR$136"SPACEBAR"CHR$137;:NORMAL=1:PRINT "> TO START";CHR$151 : GOSUB 60:IF X$ = " " THEN RETURN

830 IF C > 100 AND TR = 0AND L = 0 THEN GOSUB 1340:P. TAB(1,11);YS$;", LIEUTENANTS":P. TAB(1,12);"OF THE MARINER, ";LY$;", ASK IF": P. TAB(1,13);"YOU WILL DONATE ";:Q = DN:GOSUB 1330:P. TAB(1,14);"TO THE TEMPLE OF TIN HAU, THE"
831 IF C > 100 AND TR = 0 AND L = 0THEN PRINT TAB(1,15);"SEA GODDESS. (Y) OR (N)"

840 IF C > 0 AND TR = 0 AND L = 0 THEN GOSUB 60: IF X$ = "Y" THEN C = C - DN:TR = 1:GOSUB 130:GOSUB 1340:PRINT TAB(1,11);" ":PRINT " " ;YS$;" THANK": PRINT TAB(1,12);" YOU, AND DEPART.": PRINT:PRINT A$;
841 IF C > 0 AND TR = 0 AND L = 0THEN IF X$ <> "Y" THEN GOSUB 1340: PRINT TAB(1,11); " ":PRINT TAB(1,12);" ";YS$;" DEPART":PRINT TAB(1,13);"ABRUPTLY IN A CHILLY SILENCE.": PRINT A$;

```

TODO: Need to shift the Lender as well. And any other main screen activities

#### Separated main screen left-hand indent

##### By `TAB`

Lender, Temple, Stats

```none
130 VDU12:PRINT TAB(1,0); "PORT ";L$(L);: PRINT TAB(27,0);M$(M);". ";DA+1;",";Y
140 INVERSE=0:PRINT TAB(1,1); "CASH ";: Q = C:GOSUB 1330:NORMAL=0:PRINT TAB(27,1); "GUNS ";:Q=G:GOSUB 1330: PRINT TAB(1,2); "DEBT ";: Q = D:GOSUB 1330:PRINT TAB(27,2); "HOLD ";: Q = SH:GOSUB 1330

230 PRINT: PRINT TAB(1,17) " B)UY, S)ELL, L)EAVE, OR R)ETIRE?" 

250 PRINT TAB(0,17)A$;:PRINT TAB(3,17) T$;" ";TC$;"?": GOSUB 260:GOTO 280

470 GOSUB 130: GOSUB 1340:PRINT  TAB(1,11) "[WANCHAI DISTRICT OF HONGKONG: HOME OF "; W$;"]":PRINT " ";W$;" GREETS YOU, TAIPAN, AND WISHES YOU WELL.":GOSUB 780:PRINT  TAB(0,13)A$:PRINT A$
480 PRINT  TAB(1,13)"WU STATES THAT HIS IRON LOTUS": PRINT "TRIAD HAS BEEN WATCHING YOU.": GOSUB 780
490 PRINT  TAB(1,13)A$: PRINT A$: PRINT A$:PRINT TAB(1,13) W$;" ASKS, DO YOU":PRINT A$;:PRINT TAB(1,14) "WISH TO B)ORROW, P)AY, OR Q)UIT?    ";: NUM$ = ""

510 IF LD = 1 AND (B = 1 OR D > 1E4) THEN GOSUB 1340: PRINT TAB(0,14) "WU REGRETS THAT HE CANNOT LOAN": PRINT "YOU MORE AT THIS TIME, TAIPAN.";: GOSUB 760:GOTO 490

520 GOSUB 1340: PRINT TAB(0,14) "HOW MUCH DO YOU WISH TO ";LD$;",":PRINT "TAIPAN? ";:GOSUB 310:NUM = VAL (NUM$)

541 IF NUM > 2 * C THEN PRINT  TAB(0,14)A$:PRINT  TAB(1,14)W$;" REGRETS THAT HE":PRINT " CANNOT LOAN YOU THAT MUCH.";: GOSUB 760:GOTO 490

560 IF NUM > C THEN NUM = C: D = D - C:C=0:GOSUB 130:PRINT  TAB(1,14)W$;" THANKS YOU,":PRINT " TAIPAN, FOR THE PAYMENT.";: GOSUB 780:IF D < 0 THEN D = 0:GOSUB 130:GOTO 490

570 IF NUM > D THEN D = 0: C = C - NUM:PRINT  TAB(1,14) W$;" THANKS YOU FOR YOUR STARTLING GENEROSITY!";:GOSUB 130:GOSUB 780:GOTO 490
580 C = C - NUM: D = D - NUM:GOSUB 130:PRINT  TAB(1,14)W$;" ACCEPTS YOUR PAYMENT WITH GRATITUDE, TAIPAN   ";:GOSUB 780: GOTO 490

830 IF C > 100 AND TR = 0AND L = 0 THEN GOSUB 1340:P. TAB(1,11);YS$;", LIEUTENANTS":P. TAB(1,12);"OF THE MARINER, ";LY$;", ASK IF": P. TAB(1,13);"YOU WILL DONATE ";:Q = DN:GOSUB 1330:P. TAB(1,14);"TO THE TEMPLE OF TIN HAU, THE"
831 IF C > 100 AND TR = 0 AND L = 0THEN PRINT TAB(1,15);"SEA GODDESS. (Y) OR (N)"

840 IF C > 0 AND TR = 0 AND L = 0 THEN GOSUB 60: IF X$ = "Y" THEN C = C - DN:TR = 1:GOSUB 130:GOSUB 1340:PRINT TAB(1,11);" ":PRINT " " ;YS$;" THANK": PRINT TAB(1,12);" YOU, AND DEPART.": PRINT:PRINT A$;
841 IF C > 0 AND TR = 0 AND L = 0THEN IF X$ <> "Y" THEN GOSUB 1340: PRINT TAB(1,11); " ":PRINT TAB(1,12);" ";YS$;" DEPART":PRINT TAB(1,13);"ABRUPTLY IN A CHILLY SILENCE.": PRINT A$;
```

##### By prefix control string

Lender, Temple, Stats.

Note the addtion of line 230, the market menu, which was original indented anyway, has had the indent remove (and the preceding space in the string)

Note the addition of line 250, the goods selection menu

Note lines 470, 480, 490, 510, 520, 541, 560, 570 and 580 of the lender routine. The problem is that the text "wraps around"

```none
7 FG$=CHR$132:FT$=CHR$148:BG$=CHR$147:GC$=CHR$135:SR$=CHR$(129):SG$=CHR$(130):SGG$=CHR$(146): REM COLOUR VARIABLES
8 BK$=CHR$157:LB$=CHR$181:RB$=CHR$234:FB$=CHR$255:LG$=CHR$(141):FLSH$=CHR$(136):STDY$=CHR$(137): REM COLOUR CONSTANTS - DO NOT CHANGE

130 VDU12:PRINT TAB(0,0)FT$"PORT ";L$(L);: PRINT TAB(27,0);M$(M);". ";DA+1;",";Y
140 INVERSE=0:PRINT TAB(0,1)FT$"CASH ";: Q = C:GOSUB 1330:NORMAL=0:PRINT TAB(27,1); "GUNS ";:Q=G:GOSUB 1330: PRINT TAB(0,2)FT$"DEBT ";: Q = D:GOSUB 1330:PRINT TAB(27,2); "HOLD ";: Q = SH:GOSUB 1330

230 PRINT: PRINT TAB(0,17)FT$"B)UY, S)ELL, L)EAVE, OR R)ETIRE?" 

250 PRINT TAB(0,17)A$;:PRINT TAB(0,17)FT$T$;" ";TC$;"?": GOSUB 260:GOTO 280

470 GOSUB 130: GOSUB 1340:PRINT  TAB(0,11)FT$"[WANCHAI DISTRICT OF HONGKONG: HOME OF "; W$;"]":PRINTFT$W$;" GREETS YOU, TAIPAN, AND WISHES YOU WELL.":GOSUB 780:PRINT  TAB(0,13)A$:PRINT A$
480 PRINT  TAB(0,13)FT$"WU STATES THAT HIS IRON LOTUS": PRINTFT$"TRIAD HAS BEEN WATCHING YOU.": GOSUB 780
490 PRINT  TAB(0,13)A$: PRINT A$: PRINT A$:PRINT TAB(0,13)FT$W$;" ASKS, DO YOU":PRINT A$;:PRINT TAB(0,14)FT$"WISH TO B)ORROW, P)AY, OR Q)UIT?    ";: NUM$ = ""

510 IF LD = 1 AND (B = 1 OR D > 1E4) THEN GOSUB 1340: PRINT TAB(0,14)FT$"WU REGRETS THAT HE CANNOT LOAN": PRINTFT$"YOU MORE AT THIS TIME, TAIPAN.";: GOSUB 760:GOTO 490

520 GOSUB 1340: PRINT TAB(0,14)FT$"HOW MUCH DO YOU WISH TO ";LD$;",":PRINT FT$"TAIPAN? ";:GOSUB 310:NUM = VAL (NUM$)

541 IF NUM > 2 * C THEN PRINT  TAB(0,14)A$:PRINT  TAB(0,14)FT$W$;" REGRETS THAT HE":PRINT FT$"CANNOT LOAN YOU THAT MUCH.";: GOSUB 760:GOTO 490

560 IF NUM > C THEN NUM = C: D = D - C:C=0:GOSUB 130:PRINT  TAB(0,14)FT$W$;" THANKS YOU,":PRINT FT$"TAIPAN, FOR THE PAYMENT.";: GOSUB 780:IF D < 0 THEN D = 0:GOSUB 130:GOTO 490

570 IF NUM > D THEN D = 0: C = C - NUM:PRINT  TAB(0,14) FT$W$;" THANKS YOU FOR YOUR STARTLING GENEROSITY!";:GOSUB 130:GOSUB 780:GOTO 490
580 C = C - NUM: D = D - NUM:GOSUB 130:PRINT  TAB(0,14)FT$W$;" ACCEPTS YOUR PAYMENT WITH GRATITUDE, TAIPAN   ";:GOSUB 780: GOTO 490

830 IF C > 100 AND TR = 0AND L = 0 THEN GOSUB 1340:P. TAB(0,11)FT$YS$;", LIEUTENANTS":P. TAB(0,12)FT$"OF THE MARINER, ";LY$;", ASK IF": P. TAB(0,13)FT$"YOU WILL DONATE ";:Q = DN:GOSUB 1330:P. TAB(0,14)FT$"TO THE TEMPLE OF TIN HAU, THE"
831 IF C > 100 AND TR = 0 AND L = 0THEN PRINT TAB(0,15)FT$"SEA GODDESS. (Y) OR (N)"

840 IF C > 0 AND TR = 0 AND L = 0 THEN GOSUB 60: IF X$ = "Y" THEN C = C - DN:TR = 1:GOSUB 130:GOSUB 1340:PRINT TAB(0,11)FT$" ":PRINT " " ;YS$;" THANK": PRINT TAB(0,12)FT$" YOU, AND DEPART.": PRINT:PRINT A$;
841 IF C > 0 AND TR = 0 AND L = 0THEN IF X$ <> "Y" THEN GOSUB 1340: PRINT TAB(0,11)FT$" ":PRINT TAB(0,12)FT$" ";YS$;" DEPART":PRINT TAB(0,13)FT$"ABRUPTLY IN A CHILLY SILENCE.": PRINT A$;
```


 
#### Delimiiting goods table ends (left and right edge)

Having to split line 150 into lines 150, 151 and 152, due to excessive length (again):

```none
141 P. TAB(0,3); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3);CHR$151;CHR$255;"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(25,3);"HONGKONG";TAB(34,3);"GODOWN"

150 FORI=0TO5:P.TAB(0,4+I);CHR$151;CHR$181:P.TAB(3,4+I);G$(I):P.TAB(7,4+I);CHR$151;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330
151 P.TAB(22,4+I);CHR$151;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(39,4+I);CHR$151;CHR$181:NEXTI
151 P.TAB(22,4+I);CHR$151;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(39,4+I);CHR$181:NEXTI
151 P.TAB(22,4+I);CHR$151;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(38,4+I);CHR$151;CHR$181:NEXTI
152 P.CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:RETURN
```

[![BBCTaipan_MODE7_BadTableEnds][1022]][1022]

The inverse bar for market prices (line 152) gets lost, by the `CHR$151` and `CHR$181` in line 151. No, well yes, the inverse bar gets lost, but because it is now printed on a line lower (line 11) than the market heading (which is line 10) and then gets blanked out. The row of text blocks can be seen briefly when leaing the market, or entering the market (i.e. option T)rade from main port menu).

It happens too quickly to get a screenshot

[![BBCTaipan_MODE7_FleetingGlimpse][1023]][1023]

To fix, add a `TAB(0,10)`

```none
152 P.TAB(0,10);CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:RETURN
```

Missing final character, so print another one:

```none
152 P.TAB(0,10);CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:P.CHR$255:RETURN
```

Better, but the right hand table delimiter needs to be a right-hand vertical bar (`CHR$234`), not a left-hand vertical bar (`CHR$181`), so:

```none
151 P.TAB(22,4+I);CHR$151;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(38,4+I);CHR$151;CHR$234:NEXTI
```


##### Nearly the final answer

```none
141 P. TAB(0,3); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3);CHR$151;CHR$255;"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(25,3);"HONGKONG";TAB(34,3);"GODOWN"

150 FORI=0TO5:P.TAB(0,4+I);CHR$151;CHR$181:P.TAB(3,4+I);G$(I):P.TAB(7,4+I);CHR$151;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330
151 P.TAB(22,4+I);CHR$151;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(38,4+I);CHR$151;CHR$234:NEXTI
152 P.TAB(0,10);CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:P.CHR$255:RETURN

```

Not too shabby looking really.

[![BBCTaipan_MODE7_IndentedTableEnds][1024]][1024]

##### Tidying things up

Need to shift the goods names left by one:

```none
150 FORI=0TO5:P.TAB(0,4+I);CHR$151;CHR$181:P.TAB(2,4+I);G$(I):P.TAB(7,4+I);CHR$151;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330
```

[![BBCTaipan_MODE7_IndentedTableEnds_GoodsL][1025]][1025]

Need to also shift right the second column to make room for the names of the goods.

```none
150 FORI=0TO5:P.TAB(0,4+I);CHR$151;CHR$181:P.TAB(2,4+I);G$(I):P.TAB(8,4+I);CHR$151;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330
```

[![BBCTaipan_MODE7_IndentedTableEnds_GoodsLR][1026]][1026]

It would be good to get the right hand edge of the column headers as a block, after `GODOWN"`:

```none
141 P. TAB(0,3); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3);CHR$151;CHR$255;"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(24,3);"HONGKONG";TAB(33,3);"GODOWN";CHR$255
```

[![BBCTaipan_MODE7_IndentedTableEnds_GodownEdge][1027]][1027]

#### Two options: Questions about the blocks

Are the blocks in the middle of phrases, like `ABOARD SHIP` and `HONGKONG GODOWN` bad/confusing?

Without the graphics blocks:

```none
141 P. TAB(0,3); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3);CHR$151;CHR$255;"GOODS"; TAB(10,3);"ABOARD SHIP";TAB(24,3);"HONGKONG GODOWN";CHR$255
```

[![BBCTaipan_MODE7_IndentedTableEnds_NoBlocks][1028]][1028]

Note that the Market Prices heading has spaces at either end, whereas the Goods heading have very little room to accomodate extra spaces.

Fixing this inconsistency by removing the spaces from around the market prices heading:

```none
220 GOSUB 790: GOSUB 1340:INVERSE=0:PRINT TAB(8,10);L$(L);" MARKET PRICES":NORMAL=0:PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(1,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);
```

[![BBCTaipan_MODE7_IndentedTableEnds_NoBlocksNorSpaces][1029]][1029]


If not, then the `XXX MARKET PRICES` heading also needs blocks interspersed as spaces. Personally, I find them jarring to read, as well as the fact that they look as if they are separate entities which are missing columns half-bar graphics, i.e. "ABROAD" and "SHIP", as if they were two distinct columns.

Market prices *with* graphics blocks:

```none
220 GOSUB 790: GOSUB 1340:INVERSE=0:PRINT TAB(0,10);CHR$151;TAB(7,10)CHR$255;L$(L);CHR$255;"MARKET";CHR$255;"PRICES":NORMAL=0:PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(1,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);
```

[![BBCTaipan_MODE7_IndentedTableEnds_MarketBlocks][1030]][1030]

Note that this, consistently, has no spaces either side of headings, neither for goods nor market prices.

#### Does `MAXINT` fit?

TODO: Need to test with `MAXINT`, which for the BBC, is XXXX????.

[![BBCTaipan_MODE7_IndentedTableEnds_MaxInt][1040]][1040]

### Final complete black and white patch for goods table

Going with the no-spaces-around-headings and no blocks-as-spaces option:

```none
10 HOME=0:VDU12:A$ = "                                        ":W$ = "ELDER BROTHER WU":LY$ = "LI YUEN":YS$ = "YANATO & SMYTHE":TC$ = "O, S, T, A, P, OR R"
11 B$="                                       "
20 PRINT TAB(0,3);CHR$151;:FOR N=1 TO 37: PRINT CHR$255;: NEXT N: PRINT CHR$255:PRINT: PRINT TAB(12); "T A I P A N:": PRINT TAB(12);"_________________"
21 SPEED = 100:PRINT TAB(13,8);"A  G A M E  I N":PRINT TAB( 14);"C O N T E X T": PRINT:PRINT TAB( 13);"HAYDEN BOOK CO."
22 SPEED = 255:PRINT TAB(0,14);CHR$151;:FOR N=1 TO 37: PRINT CHR$255;: NEXT N: PRINT CHR$255

130 VDU12:PRINT TAB(1,0); "PORT ";L$(L);: PRINT TAB(27,0);M$(M);". ";DA+1;",";Y
140 INVERSE=0:PRINT TAB(1,1); "CASH ";: Q = C:GOSUB 1330:NORMAL=0:PRINT TAB(27,1); "GUNS ";:Q=G:GOSUB 1330: PRINT TAB(1,2); "DEBT ";: Q = D:GOSUB 1330:PRINT TAB(27,2); "HOLD ";: Q = SH:GOSUB 1330
141 P. TAB(0,3); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3);CHR$151;CHR$255;"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(24,3);"HONGKONG";TAB(33,3);"GODOWN";CHR$255

150 FORI=0TO5:P.TAB(0,4+I);CHR$151;CHR$181:P.TAB(2,4+I);G$(I):P.TAB(8,4+I);CHR$151;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330
151 P.TAB(22,4+I);CHR$151;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(38,4+I);CHR$151;CHR$234:NEXTI
152 P.TAB(0,10);CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:P.CHR$255:RETURN

220 GOSUB 790: GOSUB 1340: PRINT TAB(8,10);L$(L);" MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(1,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);

230 PRINT: PRINT TAB(1,17) " B)UY, S)ELL, L)EAVE, OR R)ETIRE?" 

590 PRINT TAB(6,14);CHR$135"PRESS <";:FLASH=1:PRINT CHR$136"SPACEBAR"CHR$137;:NORMAL=1:PRINT "> TO START";CHR$151 : GOSUB 60:IF X$ = " " THEN RETURN

830 IF C > 100 AND TR = 0AND L = 0 THEN GOSUB 1340:P. TAB(1,11);YS$;", LIEUTENANTS":P. TAB(1,12);"OF THE MARINER, ";LY$;", ASK IF": P. TAB(1,13);"YOU WILL DONATE ";:Q = DN:GOSUB 1330:P. TAB(1,14);"TO THE TEMPLE OF TIN HAU, THE"
831 IF C > 100 AND TR = 0 AND L = 0THEN PRINT TAB(1,15);"SEA GODDESS. (Y) OR (N)"

840 IF C > 0 AND TR = 0 AND L = 0 THEN GOSUB 60: IF X$ = "Y" THEN C = C - DN:TR = 1:GOSUB 130:GOSUB 1340:PRINT TAB(1,11);" ":PRINT " " ;YS$;" THANK": PRINT TAB(1,12);" YOU, AND DEPART.": PRINT:PRINT A$;
841 IF C > 0 AND TR = 0 AND L = 0THEN IF X$ <> "Y" THEN GOSUB 1340: PRINT TAB(1,11); " ":PRINT TAB(1,12);" ";YS$;" DEPART":PRINT TAB(1,13);"ABRUPTLY IN A CHILLY SILENCE.": PRINT A$;

```

## Using custom graphics characters for inverse letters

You can not, as there are no custom graphics in `MODE 7`, only *sixels* and colours.

## Using colour (blue on yellow) again

The problem with the blocks as spaces (above) is that the text itself is *not* inverse, hence the "jarring" effect. If, colour, that cn be inversed – such as blue on yellow, is used just for the Goods and the Market Prices, and yellow is used for the graphic half bar sourrounds, that might be less jarring. However, the control characters will leave blanks, and will need careful alignent.

Basic example from **Using colour (for the table)** above:

```none
141 PRINT TAB(0,3);CHR$(131);CHR$(157);CHR$(132); "GOODS     ABOARD SHIP    HONGKONG GODOWN"
150 FOR I = 0 TO 5: PRINT TAB(0,4+I);CHR$135;G$(I):P.TAB(7,4+I);CHR$147;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330:P.TAB(22,4+I);CHR$147;CHR$181;CHR$135:PRINT TAB(27,4+I);:Q=GG(I):GOSUB 1330:NEXTI
```

Using this to adapt the "Final answer"

Note: Lines 141 and 152 have been greatly simplifed by using just the background colour – no need for graphics blocks.

```none
141 P. TAB(0,3); CHR$147;CHR$(157);CHR$(132);:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3);CHR$147;CHR$(157);CHR$(148);CHR$255;"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(24,3);"HONGKONG";TAB(33,3);"GODOWN";CHR$255
141 P. TAB(0,3); CHR$147;CHR$(157);CHR$(132);:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3);CHR$147;CHR$(157);CHR$(148);"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(24,3);"HONGKONG";TAB(33,3);"GODOWN";CHR$255
141 REM remove: P. TAB(0,3); CHR$147;CHR$(157);CHR$(132);:FOR N=1 TO 39: P. CHR$255;: NEXT N:
141 PRINT TAB(0,3);CHR$147;CHR$(157);CHR$(148);"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(24,3);"HONGKONG";TAB(33,3);"GODOWN"

150 FORI=0TO5:P.TAB(0,4+I);CHR$147;CHR$181:P.TAB(2,4+I);G$(I):P.TAB(8,4+I);CHR$147;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330
151 P.TAB(22,4+I);CHR$147;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(38,4+I);CHR$147;CHR$234:NEXTI
152 P.TAB(0,10);CHR$147;:FOR N=1TO38:P.CHR$255;:NEXTN:P.CHR$255:RETURN
152 P.TAB(0,10);CHR$147;CHR$(157);CHR$(132);A$:RETURN

220 GOSUB 790: GOSUB 1340: PRINT TAB(0,10);CHR$147;CHR$(157);CHR$(148);TAB(8,10);L$(L);"MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(1,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);

```

[![BBCTaipan_MODE7_ColourTable][1050]][1050]

Note that the goods names are in yellow. Changing to white:

```none
150 FORI=0TO5:P.TAB(0,4+I);CHR$147;CHR$181:P.TAB(2,4+I);CHR$135;G$(I):P.TAB(8,4+I);CHR$147;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330

```

[![BBCTaipan_MODE7_ColourTable_WhiteGoods][1051]][1051]

But this right shifts the goods names over by one, thereby truncating "PEPPER" again

So shifting TAB 8 -> 9

```none
150 FORI=0TO5:P.TAB(0,4+I);CHR$147;CHR$181:P.TAB(2,4+I);CHR$135;G$(I):P.TAB(9,4+I);CHR$147;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330

```

[![BBCTaipan_MODE7_ColourTable_WhiteGoodsShifted][1052]][1052]

### Final complete colour patch for goods table

#### Blue on yellow

White goods names and values

```none
141 PRINT TAB(0,3);CHR$147;CHR$(157);CHR$(148);"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(24,3);"HONGKONG";TAB(33,3);"GODOWN"

150 FORI=0TO5:P.TAB(0,4+I);CHR$147;CHR$181:P.TAB(2,4+I);CHR$135;G$(I):P.TAB(9,4+I);CHR$147;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330
151 P.TAB(22,4+I);CHR$147;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(38,4+I);CHR$147;CHR$234:NEXTI
152 P.TAB(0,10);CHR$147;CHR$(157);CHR$(132);A$:RETURN

220 GOSUB 790: GOSUB 1340: PRINT TAB(0,10);CHR$147;CHR$(157);CHR$(148);TAB(8,10);L$(L);"MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(1,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);
```

As above

[![BBCTaipan_MODE7_ColourTable_WhiteGoodsShifted][1052]][1052]

#### Blue on green

Note the evolution of line 150, into lines 150 and 151:

##### White goods names and values

```none
141 PRINT TAB(0,3);CHR$(130);CHR$(157);CHR$(132);"GOODS     ABOARD SHIP    HONGKONG GODOWN"
141 PRINT TAB(0,3);CHR$146;CHR$(157);CHR$(148);"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(24,3);"HONGKONG";TAB(33,3);"GODOWN"

150 FOR I = 0 TO 5: PRINT TAB(0,4+I);CHR$130;G$(I):P.TAB(7,4+I);CHR$146;CHR$181;CHR$130:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330:P.TAB(22,4+I);CHR$146;CHR$181;CHR$130:PRINT TAB(27,4+I);:Q=GG(I):GOSUB 1330:NEXTI
150 FORI=0TO5:P.TAB(0,4+I);CHR$146;CHR$181:P.TAB(2,4+I);CHR$135;G$(I):P.TAB(9,4+I);CHR$146;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330
151 P.TAB(22,4+I);CHR$146;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(38,4+I);CHR$146;CHR$234:NEXTI
152 P.TAB(0,10);CHR$146;CHR$157;CHR$132;A$:RETURN

220 GOSUB 790: GOSUB 1340:PRINT TAB(0,10);CHR$(130);CHR$(157);CHR$(132);"        ";L$(L);" MARKET PRICES ":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(0,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);
```

[![BBCTaipan_MODE7_BonG_ColourTable_WhiteGoodsShifted][1053]][1053]

##### All green (no white)

```none
141 PRINT TAB(0,3);CHR$146;CHR$(157);CHR$(148);"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(24,3);"HONGKONG";TAB(33,3);"GODOWN"
150 FORI=0TO5:P.TAB(0,4+I);CHR$146;CHR$181:P.TAB(2,4+I);CHR$130;G$(I):P.TAB(9,4+I);CHR$146;CHR$181;CHR$130:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330
151 P.TAB(22,4+I);CHR$146;CHR$181;CHR$130:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(38,4+I);CHR$146;CHR$234:NEXTI
152 P.TAB(0,10);CHR$146;CHR$157;CHR$132;A$:RETURN

220 GOSUB 790: GOSUB 1340:PRINT TAB(0,10);CHR$(130);CHR$(157);CHR$(132);"        ";L$(L);" MARKET PRICES ":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(0,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);
```

[![BBCTaipan_MODE7_BonG_ColourTable_AllGreen][1054]][1054]


## Adding border for the market prices

Note that the numbers *were* right justified before applying this fix, then they are left justified.

### Black and white market prices table

The left and right hand sides will need to be indented by one or two characters, for control characters and the graphics blocks.

Taking line 220 from the "final answer" and the so far untouched bare-bones line 221, and adding graphic characters to the start and end

```none
220 GOSUB 790: GOSUB 1340: PRINT TAB(8,10);L$(L);" MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2:P.TAB(0,12+I/2);CHR$151;CHR$181;TAB(2,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);
221 PRINT TAB(29) GP(I + 1) :P.TAB(38,12+I/2);CHR$147;CHR$234:NEXT I
```

gives corrupted numbers

[![BBCTaipan_MODE7_MarketTableBad][1060]][1060]

Fixing the end column's colour, but adding an alphanumberic control character does not fix the numbers

```none
220 GOSUB 790: GOSUB 1340: PRINT TAB(8,10);L$(L);" MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2:P.TAB(0,12+I/2);CHR$151;CHR$181;CHR$135;TAB(2,12+I/2);G$(I);: PRINT TAB(9) GP(I) ;: PRINT TAB(20); G$(I + 1);
221 PRINT TAB(29) GP(I + 1) :P.TAB(38,12+I/2);CHR$151;CHR$234:NEXT I
```

[![BBCTaipan_MODE7_MarketTableStillBad][1061]][1061]

---

##### An aside: removing `PRINT` statements

Removing the superfluous `PRINT` statements causes the **left hand justification** (in the left hand column) *and* still does not fix the numbers

```none
220 GOSUB 790: GOSUB 1340: PRINT TAB(8,10);L$(L);" MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2:P.TAB(0,12+I/2);CHR$151;CHR$181;CHR$135;TAB(2,12+I/2);G$(I); TAB(9) GP(I) ; TAB(20); G$(I + 1);
221 PRINT TAB(29) GP(I + 1) :P.TAB(38,12+I/2);CHR$151;CHR$234:NEXT I
```

[![BBCTaipan_MODE7_MarketTableStillBad2][1062]][1062]

Removing the superfluous `PRINT` statements but adding explict TAB(x,y) instead of just TAB(x)

Note: These may not be superfluous, and may be the only way to get right justified numbers, as no `;` is used.

```none
220 GOSUB 790: GOSUB 1340: PRINT TAB(8,10);L$(L);" MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2:P.TAB(0,12+I/2);CHR$151;CHR$181;CHR$135;TAB(2,12+I/2);G$(I); TAB(9,12+I/2) GP(I) ; TAB(20,12+I/2); G$(I + 1);
221 PRINT TAB(29,12+I/2) GP(I + 1) :P.TAB(38,12+I/2);CHR$151;CHR$234:NEXT I

```

Still looks bad *and* there is left justification in the left most column:

[![BBCTaipan_MODE7_MarketTableStillBad2_ExplictTAB][1063]][1063]

Giving up on that, as it is very strange.

---

##### Continuing on

Leaving in the PRINT statements and adding *another* alphanumeric control seems to fix the numbers as graphics issue, although there is now left justification in the first column:

```none
220 GOSUB 790: GOSUB 1340: PRINT TAB(8,10);L$(L);" MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2:P.TAB(0,12+I/2);CHR$151;CHR$181;CHR$135;TAB(2,12+I/2);G$(I);: PRINT TAB(9);CHR$135; GP(I) ;: PRINT TAB(20); G$(I + 1);
221 PRINT TAB(29) GP(I + 1) :P.TAB(38,12+I/2);CHR$151;CHR$234:NEXT I
```

[![BBCTaipan_MODE7_MarketTableBetter][1064]][1064]

Fixing the first column's left justification

```none
220 GOSUB 790: GOSUB 1340: PRINT TAB(8,10);L$(L);" MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2:P.TAB(0,12+I/2);CHR$151;CHR$181;TAB(2,12+I/2);G$(I): PRINT TAB(8,12+I/2);CHR$135:P.TAB(9,12+I/2) GP(I) : PRINT TAB(20,12+I/2); G$(I + 1);
```

Note the difference between these lines

```
220 GOSUB 790: GOSUB 1340: PRINT TAB(8,10);L$(L);" MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2:P.TAB(0,12+I/2);CHR$151;CHR$181;TAB(2,12+I/2);G$(I): PRINT TAB(8,12+I/2);CHR$135;TAB(9,12+I/2) GP(I) : PRINT TAB(20,12+I/2); G$(I + 1);

and

220 GOSUB 790: GOSUB 1340: PRINT TAB(8,10);L$(L);" MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2:P.TAB(0,12+I/2);CHR$151;CHR$181;TAB(2,12+I/2);G$(I): PRINT TAB(8,12+I/2);CHR$135:P.TAB(9,12+I/2) GP(I) : PRINT TAB(20,12+I/2); G$(I + 1);
```

To avoid left justification, you need a new `PRINT `statement, with no `;`. Just the one semicolon causes a mess, like so: 

```none
PRINT TAB(8,12+I/2);CHR$135;TAB(9,12+I/2) GP(I)`
```

[![BBCTaipan_MODE7_MarketTableBetter_LJust][1065]][1065]

But we still have the second column numbers on the following line

```none
Original
221 PRINT TAB(29) GP(I + 1) :P.TAB(38,12+I/2);CHR$151;CHR$234:NEXT I
221 PRINT TAB(29,12+I/2) GP(I + 1) :P.TAB(38,12+I/2);CHR$151;CHR$234:NEXT I
221 PRINT TAB(28,12+I/2) GP(I + 1) :P.TAB(38,12+I/2);CHR$151;CHR$234:NEXT I
```

Better. It seems as if more indent is required to clear the right most control character (maybe?)

[![BBCTaipan_MODE7_MarketTableBetter_LJustRJust][1066]][1066]

#### Probable B/W solution for market prices border

```none
220 GOSUB 790: GOSUB 1340: PRINT TAB(8,10);L$(L);" MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2:P.TAB(0,12+I/2);CHR$151;CHR$181;TAB(2,12+I/2);G$(I): PRINT TAB(8,12+I/2);CHR$135:P.TAB(9,12+I/2) GP(I) : PRINT TAB(20,12+I/2); G$(I + 1);
221 PRINT TAB(28,12+I/2) GP(I + 1) :P.TAB(38,12+I/2);CHR$151;CHR$234:NEXT I
```


#### Bottom bar

Adding a bottom bar (that may well overlap the messages area)

```none
222 P.TAB(0,15);CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:P.CHR$255
```

[![BBCTaipan_MODE7_MarketTableBottom][1067]][1067]

#### Top line – empty or shift?

Fixing the missing top empty line, in one of two ways:

 - Add an empty top line
 - Shift it all up

Empty top line
 
```none
223 PRINT TAB(0,11);CHR$151;CHR$181: PRINT TAB(38,11);CHR$151;CHR$234
```

[![BBCTaipan_MODE7_MarketTableBottom_EmptyTop][1068]][1068]

Shift it all up

```none
220 GOSUB 790: GOSUB 1340: PRINT TAB(8,10);L$(L);" MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2:P.TAB(0,11+I/2);CHR$151;CHR$181;TAB(2,11+I/2);G$(I): PRINT TAB(8,11+I/2);CHR$135:P.TAB(9,11+I/2) GP(I) : PRINT TAB(20,11+I/2); G$(I + 1);
221 PRINT TAB(29,11+I/2) GP(I + 1) :P.TAB(38,11+I/2);CHR$151;CHR$234:NEXT I
221 PRINT TAB(28,11+I/2) GP(I + 1) :P.TAB(38,11+I/2);CHR$151;CHR$234:NEXT I
222 P.TAB(0,14);CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:P.CHR$255

NOT this:

220 GOSUB 790: GOSUB 1340: PRINT TAB(8,10);L$(L);" MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2:P.TAB(0,11+I/2);CHR$151;CHR$181;CHR$135;TAB(2,11+I/2);G$(I);: PRINT TAB(9);CHR$135; GP(I) ;: PRINT TAB(20); G$(I + 1);
221 PRINT TAB(29) GP(I + 1) :P.TAB(38,11+I/2);CHR$151;CHR$234:NEXT I
222 P.TAB(0,14);CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:P.CHR$255
```

[![BBCTaipan_MODE7_MarketTableBottom_ShiftUp][1069]][1069]

The empty top line looks more elegant (and less crowded), but the shift up is probably more efficient of real estate. Note that the original Apple II version also has the blank line

[![BBCTaipan_MODE7_MarketTable_AppleII][1070]][1070]

#### Adding a vertical bar for market prices

Re-organising lines 220 and 221

```none
220 GOSUB 790: GOSUB 1340: PRINT TAB(8,10);L$(L);" MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2:P.TAB(0,12+I/2);CHR$151;CHR$181;TAB(2,12+I/2);G$(I): PRINT TAB(8,12+I/2)CHR$135:P.TAB(9,12+I/2) GP(I)
221 PRINT TAB(20,12+I/2)CHR$151;CHR$181; G$(I + 1);:P.TAB(27,12+I/2)CHR$135: PRINT TAB(28,12+I/2) GP(I + 1) :P.TAB(38,12+I/2);CHR$151;CHR$234:NEXT I
223 PRINT TAB(0,11);CHR$151;CHR$181;TAB(21)CHR$181; TAB(39,11)CHR$234
```

[![BBCTaipan_MODE7_MarketTableBottom_MiddleBar_Good][1071]][1071]

### Complete B/W patch for goods table and market prices

With first line empty (as per Apple II)

#### Without center bar

```none
10 HOME=0:VDU12:A$ = "                                        ":W$ = "ELDER BROTHER WU":LY$ = "LI YUEN":YS$ = "YANATO & SMYTHE":TC$ = "O, S, T, A, P, OR R"
11 B$="                                       "
20 PRINT TAB(0,3);CHR$151;:FOR N=1 TO 37: PRINT CHR$255;: NEXT N: PRINT CHR$255:PRINT: PRINT TAB(12); "T A I P A N:": PRINT TAB(12);"_________________"
21 SPEED = 100:PRINT TAB(13,8);"A  G A M E  I N":PRINT TAB( 14);"C O N T E X T": PRINT:PRINT TAB( 13);"HAYDEN BOOK CO."
22 SPEED = 255:PRINT TAB(0,14);CHR$151;:FOR N=1 TO 37: PRINT CHR$255;: NEXT N: PRINT CHR$255

130 VDU12:PRINT TAB(1,0); "PORT ";L$(L);: PRINT TAB(27,0);M$(M);". ";DA+1;",";Y
140 INVERSE=0:PRINT TAB(1,1); "CASH ";: Q = C:GOSUB 1330:NORMAL=0:PRINT TAB(27,1); "GUNS ";:Q=G:GOSUB 1330: PRINT TAB(1,2); "DEBT ";: Q = D:GOSUB 1330:PRINT TAB(27,2); "HOLD ";: Q = SH:GOSUB 1330
141 P. TAB(0,3); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3);CHR$151;CHR$255;"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(24,3);"HONGKONG";TAB(33,3);"GODOWN";CHR$255

150 FORI=0TO5:P.TAB(0,4+I);CHR$151;CHR$181:P.TAB(2,4+I);G$(I):P.TAB(8,4+I);CHR$151;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330
151 P.TAB(22,4+I);CHR$151;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(38,4+I);CHR$151;CHR$234:NEXTI
152 P.TAB(0,10);CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:P.CHR$255:RETURN

220 GOSUB 790: GOSUB 1340: PRINT TAB(8,10);L$(L);" MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2:P.TAB(0,12+I/2);CHR$151;CHR$181;TAB(2,12+I/2);G$(I): PRINT TAB(8,12+I/2);CHR$135:P.TAB(9,12+I/2) GP(I) : PRINT TAB(20,12+I/2); G$(I + 1);
221 PRINT TAB(28,12+I/2) GP(I + 1) :P.TAB(38,12+I/2);CHR$151;CHR$234:NEXT I
222 P.TAB(0,15);CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:P.CHR$255
223 PRINT TAB(0,11);CHR$151;CHR$181: PRINT TAB(38,11);CHR$151;CHR$234

230 PRINT: PRINT TAB(1,17) " B)UY, S)ELL, L)EAVE, OR R)ETIRE?" 

590 PRINT TAB(6,14);CHR$135"PRESS <";:FLASH=1:PRINT CHR$136"SPACEBAR"CHR$137;:NORMAL=1:PRINT "> TO START";CHR$151 : GOSUB 60:IF X$ = " " THEN RETURN

830 IF C > 100 AND TR = 0AND L = 0 THEN GOSUB 1340:P. TAB(1,11);YS$;", LIEUTENANTS":P. TAB(1,12);"OF THE MARINER, ";LY$;", ASK IF": P. TAB(1,13);"YOU WILL DONATE ";:Q = DN:GOSUB 1330:P. TAB(1,14);"TO THE TEMPLE OF TIN HAU, THE"
831 IF C > 100 AND TR = 0 AND L = 0THEN PRINT TAB(1,15);"SEA GODDESS. (Y) OR (N)"

840 IF C > 0 AND TR = 0 AND L = 0 THEN GOSUB 60: IF X$ = "Y" THEN C = C - DN:TR = 1:GOSUB 130:GOSUB 1340:PRINT TAB(1,11);" ":PRINT " " ;YS$;" THANK": PRINT TAB(1,12);" YOU, AND DEPART.": PRINT:PRINT A$;
841 IF C > 0 AND TR = 0 AND L = 0THEN IF X$ <> "Y" THEN GOSUB 1340: PRINT TAB(1,11); " ":PRINT TAB(1,12);" ";YS$;" DEPART":PRINT TAB(1,13);"ABRUPTLY IN A CHILLY SILENCE.": PRINT A$;
```

#### With center bar

```none
10 HOME=0:VDU12:A$ = "                                        ":W$ = "ELDER BROTHER WU":LY$ = "LI YUEN":YS$ = "YANATO & SMYTHE":TC$ = "O, S, T, A, P, OR R"
11 B$="                                       "
20 PRINT TAB(0,3);CHR$151;:FOR N=1 TO 37: PRINT CHR$255;: NEXT N: PRINT CHR$255:PRINT: PRINT TAB(12); "T A I P A N:": PRINT TAB(12);"_________________"
21 SPEED = 100:PRINT TAB(13,8);"A  G A M E  I N":PRINT TAB( 14);"C O N T E X T": PRINT:PRINT TAB( 13);"HAYDEN BOOK CO."
22 SPEED = 255:PRINT TAB(0,14);CHR$151;:FOR N=1 TO 37: PRINT CHR$255;: NEXT N: PRINT CHR$255

130 VDU12:PRINT TAB(1,0); "PORT ";L$(L);: PRINT TAB(27,0);M$(M);". ";DA+1;",";Y
140 INVERSE=0:PRINT TAB(1,1); "CASH ";: Q = C:GOSUB 1330:NORMAL=0:PRINT TAB(27,1); "GUNS ";:Q=G:GOSUB 1330: PRINT TAB(1,2); "DEBT ";: Q = D:GOSUB 1330:PRINT TAB(27,2); "HOLD ";: Q = SH:GOSUB 1330
141 P. TAB(0,3); CHR$151;:FOR N=1 TO 39: P. CHR$255;: NEXT N:PRINT TAB(0,3);CHR$151;CHR$255;"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(24,3);"HONGKONG";TAB(33,3);"GODOWN";CHR$255

150 FORI=0TO5:P.TAB(0,4+I);CHR$151;CHR$181:P.TAB(2,4+I);G$(I):P.TAB(8,4+I);CHR$151;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330
151 P.TAB(22,4+I);CHR$151;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(38,4+I);CHR$151;CHR$234:NEXTI
152 P.TAB(0,10);CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:P.CHR$255:RETURN

220 GOSUB 790: GOSUB 1340: PRINT TAB(8,10);L$(L);" MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2:P.TAB(0,12+I/2);CHR$151;CHR$181;TAB(2,12+I/2);G$(I): PRINT TAB(8,12+I/2)CHR$135:P.TAB(9,12+I/2) GP(I)
221 PRINT TAB(20,12+I/2)CHR$151;CHR$181; G$(I + 1);:P.TAB(27,12+I/2)CHR$135: PRINT TAB(28,12+I/2) GP(I + 1) :P.TAB(38,12+I/2);CHR$151;CHR$234:NEXT I
222 P.TAB(0,15);CHR$151;:FOR N=1TO38:P.CHR$255;:NEXTN:P.CHR$255
223 PRINT TAB(0,11);CHR$151;CHR$181;TAB(21)CHR$181; TAB(39,11)CHR$234

230 PRINT: PRINT TAB(1,17) " B)UY, S)ELL, L)EAVE, OR R)ETIRE?" 

590 PRINT TAB(6,14);CHR$135"PRESS <";:FLASH=1:PRINT CHR$136"SPACEBAR"CHR$137;:NORMAL=1:PRINT "> TO START";CHR$151 : GOSUB 60:IF X$ = " " THEN RETURN

830 IF C > 100 AND TR = 0AND L = 0 THEN GOSUB 1340:P. TAB(1,11);YS$;", LIEUTENANTS":P. TAB(1,12);"OF THE MARINER, ";LY$;", ASK IF": P. TAB(1,13);"YOU WILL DONATE ";:Q = DN:GOSUB 1330:P. TAB(1,14);"TO THE TEMPLE OF TIN HAU, THE"
831 IF C > 100 AND TR = 0 AND L = 0THEN PRINT TAB(1,15);"SEA GODDESS. (Y) OR (N)"

840 IF C > 0 AND TR = 0 AND L = 0 THEN GOSUB 60: IF X$ = "Y" THEN C = C - DN:TR = 1:GOSUB 130:GOSUB 1340:PRINT TAB(1,11);" ":PRINT " " ;YS$;" THANK": PRINT TAB(1,12);" YOU, AND DEPART.": PRINT:PRINT A$;
841 IF C > 0 AND TR = 0 AND L = 0THEN IF X$ <> "Y" THEN GOSUB 1340: PRINT TAB(1,11); " ":PRINT TAB(1,12);" ";YS$;" DEPART":PRINT TAB(1,13);"ABRUPTLY IN A CHILLY SILENCE.": PRINT A$;
```



### Blue on yellow market prices table

Adding colour. For some reason the space was missing between town and "MARKET", so I added (reinstated) a space at the start of the "MARKET PRICES" string:

```none
220 GOSUB 790: GOSUB 1340: PRINT TAB(0,10);CHR$147;CHR$(157);CHR$(148);TAB(8,10);L$(L);" MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(0,12+I/2)CHR$147;CHR$181;CHR$135;G$(I);: P.TAB(9)CHR$135;GP(I) ;: PRINT TAB(20); G$(I + 1);
221 PRINT TAB(29) GP(I + 1) :P.TAB(38,12+I/2);CHR$147;CHR$234:NEXT I
```

[![BBCTaipan_MODE7_ColourMarketTable][1080]][1080]

Two issues can be seen: the left justification of the first column, and; the missing price of rice in the second column.

Replacing the semicolon with a space between the `CHR$135 GP(I)`, in line 220, makes it right justified too much!

```none
220 GOSUB 790: GOSUB 1340: PRINT TAB(0,10);CHR$147;CHR$(157);CHR$(148);TAB(8,10);L$(L);" MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(0,12+I/2)CHR$147;CHR$181;CHR$135;G$(I);: P.TAB(9)CHR$135 GP(I) ;: PRINT TAB(20); G$(I + 1);
221 PRINT TAB(29) GP(I + 1) :P.TAB(38,12+I/2);CHR$147;CHR$234:NEXT I
```

[![BBCTaipan_MODE7_ColourMarketTable_RJust][1081]][1081]

Trying a new separate `PRINT` statement and `TAB` without a preceding semicolon (`;`), for `GP(I)`. This is getting difficult as line is reaching maximum length

```none
220 GOSUB 790: GOSUB 1340: P.TAB(0,10);CHR$147;CHR$(157);CHR$(148);TAB(8,10);L$(L);" MARKET PRICES":P.A$:FOR I = 0 TO 4 STEP 2: P.TAB(0,12+I/2)CHR$147;CHR$181;CHR$135;G$(I);: P.TAB(9)CHR$135:P.TAB(10,12+I/2)GP(I) ;:P.TAB(15);G$(I + 1);
```

[![BBCTaipan_MODE7_ColourMarketTable_RJust_Bad1][1082]][1082]


Changed `TAB` of second goods name. New print statememt,. 

Adding explict TAB(x,y) throughout

```none
220 GOSUB 790: GOSUB 1340: P.TAB(0,10);CHR$147;CHR$(157);CHR$(148);TAB(8,10);L$(L);" MARKET PRICES":P.A$:FORI=0TO4STEP2: P.TAB(0,12+I/2)CHR$147;CHR$181;CHR$135;G$(I);: P.TAB(8)CHR$135:P.TAB(9,12+I/2)GP(I);:P.TAB(16,12+I/2);G$(I + 1);
```

[![BBCTaipan_MODE7_ColourMarketTable_RJust_Bad2][1083]][1083]

This is just a disaster, and just gets worse!

Time for a rethink. The second columns' TAB values (from the B/W version) are 20 for the goods name and 29 for the goods price.

```none
220 GOSUB 790: GOSUB 1340: P.TAB(0,10);CHR$147;CHR$(157);CHR$(148);TAB(8,10);L$(L);" MARKET PRICES":P.A$:FORI=0TO4STEP2: P.TAB(0,12+I/2)CHR$147;CHR$181;CHR$135;G$(I);: P.TAB(8)CHR$135:P.TAB(9,12+I/2)GP(I);:P.TAB(20,12+I/2);G$(I + 1);
221 PRINT TAB(18) GP(I + 1) :P.TAB(38,12+I/2);CHR$147;CHR$234:NEXT I
```

For some reason, in line 221, the second number's (the goods price) TAB has to be 18!!!??? Very weird and nowhere near 29.

`TAB(19)` - the rice price is still missing

[![BBCTaipan_MODE7_ColourMarketTable_RJust_TAB19][1084]][1084]

`TAB(18)` - golilocks!

[![BBCTaipan_MODE7_ColourMarketTable_RJust_TAB18][1085]][1085]

`TAB(17)` - too much of an right-hand indent

[![BBCTaipan_MODE7_ColourMarketTable_RJust_TAB17][1086]][1086]

Looking at lines 220 and 221, there is some inconsistant, and possibly inefficient or unnecessary, use of `TAB(x)` and `TAB(x,y)`. `TAB(x)` can be used if there is a preceding semicolon (as can be seen in line 221). However, the right hand justification of numbers ***seems to*** necessitte a separate `PRINT` statement.

Giving the two lines a little tidy up (but ensuring that the screen still *looks the same*:

```none
220 GOSUB 790: GOSUB 1340: P.TAB(0,10);CHR$147;CHR$(157);CHR$(148);TAB(8);L$(L);" MARKET PRICES":P.A$:FORI=0TO4STEP2: P.TAB(0,12+I/2)CHR$147;CHR$181;CHR$135;G$(I);: P.TAB(8)CHR$135:P.TAB(9,12+I/2)GP(I);TAB(20);G$(I + 1);
221 PRINT TAB(18) GP(I + 1);:P.TAB(38);CHR$147;CHR$234:NEXT I
```

For some reason `TAB(20)` causes the goods names second column to be too far to the right, and using `TAB(20,12+I/2)` fixes it. In other words, an explict `TAB(x,y)` *is* required, for whatever reason.

[![BBCTaipan_MODE7_ColourMarketTable_RJust_TAB20][1087]][1087]

TODO: Investigate, with an MRE, why the `TAB` values are so wrong


#### Bottom bar

Adding a bottom bar (that may well overlap the messages area)

```none
222 P.TAB(0,15);CHR$147;CHR$(157);CHR$(132);A$
```

[![BBCTaipan_MODE7_ColourMarketTableBottom][1090]][1090]

#### Top line – empty or shift?

Fixing the missing top empty line, in one of two ways:

 - Add an empty top line
 - Shift it all up

Empty top line
 
```none
223 PRINT TAB(0,11);CHR$147;CHR$181: PRINT TAB(38,11);CHR$147;CHR$234
```

[![BBCTaipan_MODE7_ColourMarketTableBottom_EmptyTop][1093]][1093]
 
 
Shift it all up:
 
```none
220 GOSUB 790: GOSUB 1340: PRINT TAB(0,10);CHR$147;CHR$(157);CHR$(148);TAB(8,10);L$(L);" MARKET PRICES":PRINT A$:FOR I = 0 TO 4 STEP 2: PRINT TAB(0,11+I/2)CHR$147;CHR$181;CHR$135;G$(I);: P.TAB(9)CHR$135; GP(I) ;: PRINT TAB(20); G$(I + 1);
220 GOSUB 790: GOSUB 1340: P.TAB(0,10);CHR$147;CHR$(157);CHR$(148);TAB(8,10);L$(L);" MARKET PRICES":P.A$:FORI=0TO4STEP2: P.TAB(0,11+I/2)CHR$147;CHR$181;CHR$135;G$(I);: P.TAB(8)CHR$135:P.TAB(9,11+I/2)GP(I);:P.TAB(20,11+I/2);G$(I + 1);
221 PRINT TAB(29) GP(I + 1) :P.TAB(38,11+I/2);CHR$147;CHR$234:NEXT I
221 PRINT TAB(18) GP(I + 1) :P.TAB(38,11+I/2);CHR$147;CHR$234:NEXT I

222 P.TAB(0,14);CHR$147;CHR$(157);CHR$(132);A$
```

[![BBCTaipan_MODE7_ColourMarketTableBottom_ShiftUp][1094]][1094]

The empty top line looks more elegant (and less crowded), but the shift up is probably more efficient of real estate. Note that the original Apple II version also has the blank line

[![BBCTaipan_MODE7_MarketTable_AppleII][1070]][1070]

#### Adding a vertical bar for market prices

TODO: Maybe add a vertical bar to separate the two columns of prices?


```none
220 GOSUB 790: GOSUB 1340: P.TAB(0,10);CHR$147;CHR$(157);CHR$(148);TAB(8,10);L$(L);" MARKET PRICES":P.A$:FORI=0TO4STEP2: P.TAB(0,12+I/2)CHR$147;CHR$181;CHR$135;G$(I);: P.TAB(8)CHR$135:P.TAB(9,12+I/2)GP(I);:P.TAB(20,12+I/2)CHR$147;CHR$181;CHR$135;G$(I + 1);
```

The line is too long! Major re-organisation as moving the end on to line 221

```none
220 GOSUB 790: GOSUB 1340: P.TAB(0,10);CHR$147;CHR$(157);CHR$(148);TAB(8,10);L$(L);" MARKET PRICES":P.A$:FORI=0TO4STEP2: P.TAB(0,12+I/2)CHR$147;CHR$181;CHR$135;G$(I);: P.TAB(8)CHR$135:P.TAB(9,12+I/2)GP(I);
221 P.TAB(20,12+I/2)CHR$147;CHR$181;CHR$135;G$(I + 1);:PRINT TAB(18) GP(I + 1) :P.TAB(38,12+I/2);CHR$147;CHR$234:NEXT I
```

[![BBCTaipan_MODE7_ColourMarketTableBottom_MiddleBar_EmptyLine][1095]][1095]

and with empty line having the middle column as well. Note that the redundant repeated `CHR$147` have been removed. Only one is required, not one before each graphic column:

```none
223 PRINT TAB(0,11);CHR$147;CHR$181;TAB(20);CHR$181; TAB(38,11);CHR$234
```

[![BBCTaipan_MODE7_ColourMarketTableBottom_MiddleBar_Bad][1096]][1096]

Shifting over the bars right by one, due to the removed redundant control characters, and simplifying the `TAB()`

```none
223 PRINT TAB(0,11);CHR$147;CHR$181;TAB(21);CHR$181; TAB(39);CHR$234
```

[![BBCTaipan_MODE7_ColourMarketTableBottom_MiddleBar_Good][1097]][1097]

Personally, I think this looks better, if not the best.  Way better than the black and white as it is less "jarring", as it has actual *inverse* characters.

## New changes

### Adjustable colours

Instead of hard coding the control character values, i.e. `CHR$(151)`, use 

```none
10 FG = 151
20 PRINT CHR$FG
```

This makes it easier to change the colour and/or graphics characters, as well as saving on memory and characters per line of statements.

 - Other dark/light colour combinations:
   - Red on Cyan
     - `10 PRINT CHR$145;CHR$157;CHR$150;"HI"`
   - Green on Magenta
     - `20 PRINT CHR$149;CHR$157;CHR$146;"HI"`
   - Blue on Green
     - `30 PRINT CHR$148;CHR$157;CHR$146;"HI"`
   - Examples with images
   - Should use variables, i.e. `FG` and `BG`
     - Versatile
     - Save RAM space, and characters used per line, by one byte/character per `CHR$`
     - `40 FG = 146: BG = 149: BK = 157`
     - `50 PRINT CHR$BG;CHR$BK;CHR$FG;"HI"`
     
#### Simple Example
     
```none
40 FG = 146: BG = 149: BK = 157
50 PRINT CHR$BG;CHR$BK;CHR$FG;"HI"
```

#### Cycling example

```none
40 FG = 146: BG = 149: BK = 157
45 FOR FG=145 TO 151
46 FOR BG=145 TO 151
50 PRINT CHR$BG;CHR$BK;CHR$FG;"HI"
80 NEXT BG
90 NEXT FG
```


### Efficiency savings

After watching [How to code games in BBC BASIC](https://www.youtube.com/watch?v=11YxFhygOb8&list=PLDgmuFhqDXWp2HroRhOvcSAiuObCsuWdX&index=1), a useful trick is to put the whole control character into a string variable:

```none
10 GG$="CHR$151"
20 PRINT GG$
```

Saves even more memory.

### Adding `ON ERROR`

After watching [How to code games in BBC BASIC](https://www.youtube.com/watch?v=11YxFhygOb8&list=PLDgmuFhqDXWp2HroRhOvcSAiuObCsuWdX&index=3)

```none
6 MODE 7: ON ERROR GOTO 3240

3240 REM **** Error Routine ****
3250 IF ERR = 17 GOTO 120
3260 REPORT : PRINT" in line ";ERL
3270 END
```


### Funky splash screen

After watching [How to code games in BBC BASIC](https://www.youtube.com/watch?v=11YxFhygOb8&list=PLDgmuFhqDXWp2HroRhOvcSAiuObCsuWdX&index=5), it seems a good idea to use large multi-coloured characters for the splash screen, in order to "liven it up" a bit:

```none
9 LG$=CHR$(141): SR$=CHR$(129) :SG$=CHR$(130) : REM SPLASH CONTROL

20 PRINT TAB(0,3);CHR$151;:FOR N=1 TO 37: PRINT CHR$255;: NEXT N: PRINT CHR$255:PRINT: PRINT TAB(12)LG$SR$ "T A I P A N:":PRINT TAB(12)LG$SG$ "T A I P A N:": PRINT TAB(12);"_________________"
21 SPEED = 100:PRINT TAB(13,8);"A  G A M E  I N":PRINT TAB( 14);"C O N T E X T": PRINT:PRINT TAB( 13);"HAYDEN BOOK CO."
22 SPEED = 255:PRINT TAB(0,14);CHR$151;:FOR N=1 TO 37: PRINT CHR$255;: NEXT N: PRINT CHR$255
```

[![BBCTaipan_MODE7_FunkySplash][1300]][1300]


Or maybe it should just *all* be in green on black (green monochrome), as a *homage* to the original Apple II version?

New colour control variables:

```none
LG$   - Large (double height) characters
SR$   - Splash alpha red
SG$   - Splash alpha green
SGG$  - Splash graphics green
STDY$ - Steady (no flash)
FLSH$ - Flash
```

Code:

```none
9 LG$=CHR$(141): SR$=CHR$(129) :SG$=CHR$(130) :SGG$=CHR$(146) : FLSH$=CHR$(136):STDY$=CHR$(137): REM SPLASH CONTROL

20 PRINT TAB(0,3)SGG$;:FOR N=1 TO 37: PRINT FB$;: NEXT N: PRINT FB$:PRINT: PRINT TAB(12)LG$SG$ "T A I P A N:":PRINT TAB(12)LG$SG$ "T A I P A N:": PRINT TAB(12)SG$"_________________"
21 SPEED = 100:PRINT TAB(13,8)SG$"A  G A M E  I N":PRINT TAB(14)SG$"C O N T E X T": PRINT:PRINT TAB(13)SG$"HAYDEN BOOK CO."
22 SPEED = 255:PRINT TAB(0,14)SGG$;:FOR N=1 TO 37: PRINT FB$;: NEXT N: PRINT FB$

590 PRINT TAB(6,14)SG$"PRESS <";:FLASH=1:PRINT FLSH$"SPACEBAR"STDY$;:NORMAL=1:PRINT "> TO START";SGG$ : GOSUB 60:IF X$ = " " THEN RETURN
```

Not too bad looking indeed

[![BBCTaipan_MODE7_GreenSplash][1301]][1301]

### All character codes

So far we have

```none
7 FG$=CHR$132:FT$=CHR$148:BG$=CHR$147:GC$=CHR$135 : REM COLOUR VARIABLES
8 BK$=CHR$157:LB$=CHR$181:RB$=CHR$234:FB$=CHR$255: REM COLOUR CONSTANTS - DO NOT CHANGE
9 LG$=CHR$(141): SR$=CHR$(129) :SG$=CHR$(130) :SGG$=CHR$(146) : FLSH$=CHR$(136):STDY$=CHR$(137): REM SPLASH CONTROL
```

Or dispensing with line 9 (from the **Funky Splash screen**), add sharing into the existing lines 7 and 8

```none
7 FG$=CHR$132:FT$=CHR$148:BG$=CHR$147:GC$=CHR$135:SR$=CHR$(129):SG$=CHR$(130):SGG$=CHR$(146): REM COLOUR VARIABLES
8 BK$=CHR$157:LB$=CHR$181:RB$=CHR$234:FB$=CHR$255:LG$=CHR$(141):FLSH$=CHR$(136):STDY$=CHR$(137): REM COLOUR CONSTANTS - DO NOT CHANGE
```

However, we could have a more comprehensive list, like this:

```none
T_R$=CHR$129
T_G$=CHR$130
T_Y$=CHR$131
T_B$=CHR$132
T_M$=CHR$133
T_C$=CHR$134
T_W$=CHR$135
C_FLSH$=CHR$136
T_STDY$=CHR$137
C_SGL$=CHR$140
C_DBL$=CHR$141
G_R$=CHR$145
G_G$=CHR$146
G_Y$=CHR$147
G_B$=CHR$148
G_M$=CHR$149
G_C$=CHR$150
G_W$=CHR$151
C_CONCEAL$=CHR$152
C_CONTIG$=CHR$153
C_SEP$=CHR$154
C_BKBKL$=CHR$156
C_BKNEW$=CHR$157
C_HOLD$=CHR$158
C_RLSE$=CHR$159
```
where

```none
T_ Alpha
G_ Graphics
C_ Control
```
Except that `_` is not valid as a variable name – yes it is! See [Keybard mappings](https://acorn.huininga.nl/pub/software/BeebEm/BeebEm-4.14.68000-20160619/Help/keyboard.html)

In one line

```none
9 T_R$=CHR$129:T_G$=CHR$130:T_Y$=CHR$131:T_B$=CHR$132:T_M$=CHR$133:T_C$=CHR$134:T_W$=CHR$135:C_FLSH$=CHR$136:T_STDY$=CHR$137:C_SGL$=CHR$140:C_DBL$=CHR$141:C_CONCEAL$=CHR$152:C_CONTIG$=CHR$153:C_SEP$=CHR$154:C_BKBKL$=CHR$156:C_BKNEW$=CHR$1<--TOO LONG!57:C_HOLD$=CHR$158:C_RLSE$=CHR$159:G_R$=CHR$145:G_G$=CHR$146:G_Y$=CHR$147:G_B$=CHR$148:G_M$=CHR$149:G_C$=CHR$150:G_W$=CHR$151
```

In two lines

```none
9 T_R$=CHR$129:T_G$=CHR$130:T_Y$=CHR$131:T_B$=CHR$132:T_M$=CHR$133:T_C$=CHR$134:T_W$=CHR$135:G_R$=CHR$145:G_G$=CHR$146:G_Y$=CHR$147:G_B$=CHR$148:G_M$=CHR$149:G_C$=CHR$150:G_W$=CHR$151
10 C_FLSH$=CHR$136:T_STDY$=CHR$137:C_SGL$=CHR$140:C_DBL$=CHR$141:C_CONCEAL$=CHR$152:C_CONTIG$=CHR$153:C_SEP$=CHR$154:C_BKBKL$=CHR$156:C_BKNEW$=CHR$157:C_HOLD$=CHR$158:C_RLSE$=CHR$159
11 
```

In three lines

```none
9 T_R$=CHR$129:T_G$=CHR$130:T_Y$=CHR$131:T_B$=CHR$132:T_M$=CHR$133:T_C$=CHR$134:T_W$=CHR$135:REM Alphanumeric
10 C_FLSH$=CHR$136:T_STDY$=CHR$137:C_SGL$=CHR$140:C_DBL$=CHR$141:C_CONCEAL$=CHR$152:C_CONTIG$=CHR$153:C_SEP$=CHR$154:C_BKBKL$=CHR$156:C_BKNEW$=CHR$157:C_HOLD$=CHR$158:C_RLSE$=CHR$159: REM Control
11 G_R$=CHR$145:G_G$=CHR$146:G_Y$=CHR$147:G_B$=CHR$148:G_M$=CHR$149:G_C$=CHR$150:G_W$=CHR$151: REM Graphics
```

In `MODE 7` when pasting underscore into BeebEm, the undercore is represented by a dash or minus. But in `MODE 0` the underscore is presented correctly. I discovered this after trying the code from [ASCII art on the Beeb](https://stardot.org.uk/forums/viewtopic.php?t=17317)

Test:

```none
10 T_R$="HI"
20 PRINT T_R$
```

Paste the above code in `MODE 7`, `LIST` and the underscore is a dash. Now, change to `MODE 0` and then re-`LIST`. The underscore is shown correctly.



### Complete colour patch for goods table and market prices

With first line empty (as per Apple II)

#### Without center bar

##### Hardcoded colours

```none
141 PRINT TAB(0,3);CHR$147;CHR$(157);CHR$(148);"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(24,3);"HONGKONG";TAB(33,3);"GODOWN"

150 FORI=0TO5:P.TAB(0,4+I);CHR$147;CHR$181:P.TAB(2,4+I);CHR$135;G$(I):P.TAB(9,4+I);CHR$147;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330
151 P.TAB(22,4+I);CHR$147;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(38,4+I);CHR$147;CHR$234:NEXTI
152 P.TAB(0,10);CHR$147;CHR$(157);CHR$(132);A$:RETURN

220 GOSUB 790: GOSUB 1340: P.TAB(0,10);CHR$147;CHR$(157);CHR$(148);TAB(8,10);L$(L);" MARKET PRICES":P.A$:FORI=0TO4STEP2: P.TAB(0,12+I/2)CHR$147;CHR$181;CHR$135;G$(I);: P.TAB(8)CHR$135:P.TAB(9,12+I/2)GP(I);:P.TAB(20,12+I/2);G$(I + 1);
221 PRINT TAB(18) GP(I + 1) :P.TAB(38,12+I/2);CHR$147;CHR$234:NEXT I
222 P.TAB(0,15);CHR$147;CHR$(157);CHR$(132);A$
223 PRINT TAB(0,11);CHR$147;CHR$181: PRINT TAB(38,11);CHR$147;CHR$234
```

##### Adjustable

Key - set in lines 7 and 8

```none
FG=132 CHR$(132) -> CHR$FG    - Foreground graphics colour
FT=148 CHR$(148) -> CHR$FT    - Foreground text colour
BG=147 CHR$147   -> CHR$BG    - Background colour
BK=157 CHR$(157) -> CHR$BK    - Background control (Do NOT change!)
GC=135 CHR$135   -> CHR$GC    - Goods colour (white)
LB=181 CHR$181 -> CHR$LB      - Left half block
RB=234 CHR$234 -> CHR$RB      - Right half block
FB=255 CHR$255 -> CHR$FB      - Full block
```

Code

```none
7 FG=132:FT=148:BG=147:GC=135 : REM COLOUR VARIABLES
8 BK=157:LB=181:RB=234:FB=255: REM COLOUR CONSTANTS - DO NOT CHANGE
141 PRINT TAB(0,3);CHR$BG;CHR$BK;CHR$FT;"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(24,3);"HONGKONG";TAB(33,3);"GODOWN"

150 FORI=0TO5:P.TAB(0,4+I);CHR$BG;CHR$LB:P.TAB(2,4+I);CHR$GC;G$(I):P.TAB(9,4+I);CHR$BG;CHR$LB;CHR$GC:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330
151 P.TAB(22,4+I);CHR$BG;CHR$LB;CHR$GC:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(38,4+I);CHR$BG;CHR$RB:NEXTI
152 P.TAB(0,10);CHR$BG;CHR$BK;CHR$FG;A$:RETURN

220 GOSUB 790: GOSUB 1340: P.TAB(0,10);CHR$BG;CHR$BK;CHR$FT;TAB(8,10);L$(L);" MARKET PRICES":P.A$:FORI=0TO4STEP2: P.TAB(0,12+I/2)CHR$BG;CHR$LB;CHR$GC;G$(I);: P.TAB(8)CHR$GC:P.TAB(9,12+I/2)GP(I);:P.TAB(20,12+I/2);G$(I + 1);
221 PRINT TAB(18) GP(I + 1) :P.TAB(38,12+I/2);CHR$BG;CHR$RB:NEXT I
222 P.TAB(0,15);CHR$BG;CHR$BK;CHR$FG;A$
223 PRINT TAB(0,11);CHR$BG;CHR$LB: PRINT TAB(38,11);CHR$BG;CHR$RB
```

##### Adjustable - more efficient

Key - set in lines 7 and 8

```none
FG$= CHR$(132) - Foreground graphics colour
FT$= CHR$(148) - Foreground text colour
BG$= CHR$147   - Background colour
BK$= CHR$(157) - Background control (Do NOT change!)
GC$= CHR$135   - Goods colour (white)
LB$= CHR$181   - Left half block
RB$= CHR$234   - Right half block
FB$= CHR$255   - Full block
```

Code

```none
7 FG$=CHR$132:FT$=CHR$148:BG$=CHR$147:GC$=CHR$135 : REM COLOUR VARIABLES
8 BK$=CHR$157:LB$=CHR$181:RB$=CHR$234:FB$=CHR$255: REM COLOUR CONSTANTS - DO NOT CHANGE
141 PRINT TAB(0,3)BG$BK$FT$;"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(24,3);"HONGKONG";TAB(33,3);"GODOWN"

150 FORI=0TO5:P.TAB(0,4+I)BG$LB$:P.TAB(2,4+I)GC$;G$(I):P.TAB(9,4+I)BG$LB$GC$:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330
151 P.TAB(22,4+I)BG$LB$GC$:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(38,4+I)BG$RB$:NEXTI
152 P.TAB(0,10)BG$BK$FG$;A$:RETURN

220 GOSUB 790: GOSUB 1340: P.TAB(0,10)BG$BK$FT$;TAB(8,10);L$(L);" MARKET PRICES":P.A$:FORI=0TO4STEP2: P.TAB(0,12+I/2)BG$LB$GC$G$(I);: P.TAB(8)GC$:P.TAB(9,12+I/2)GP(I);:P.TAB(20,12+I/2);G$(I + 1);
221 PRINT TAB(18) GP(I + 1) :P.TAB(38,12+I/2)BG$RB$:NEXT I
222 P.TAB(0,15)BG$BK$FG$A$
223 PRINT TAB(0,11)BG$LB$: PRINT TAB(38,11)BG$RB$
```

#### With center bar

##### Hardcoded colours

```none
141 PRINT TAB(0,3);CHR$147;CHR$(157);CHR$(148);"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(24,3);"HONGKONG";TAB(33,3);"GODOWN"

150 FORI=0TO5:P.TAB(0,4+I);CHR$147;CHR$181:P.TAB(2,4+I);CHR$135;G$(I):P.TAB(9,4+I);CHR$147;CHR$181;CHR$135:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330
151 P.TAB(22,4+I);CHR$147;CHR$181;CHR$135:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(38,4+I);CHR$147;CHR$234:NEXTI
152 P.TAB(0,10);CHR$147;CHR$(157);CHR$(132);A$:RETURN

220 GOSUB 790: GOSUB 1340: P.TAB(0,10);CHR$147;CHR$(157);CHR$(148);TAB(8,10);L$(L);" MARKET PRICES":P.A$:FORI=0TO4STEP2: P.TAB(0,12+I/2)CHR$147;CHR$181;CHR$135;G$(I);: P.TAB(8)CHR$135:P.TAB(9,12+I/2)GP(I);
221 P.TAB(20,12+I/2)CHR$147;CHR$181;CHR$135;G$(I + 1);:PRINT TAB(18) GP(I + 1) :P.TAB(38,12+I/2);CHR$147;CHR$234:NEXT I
222 P.TAB(0,15);CHR$147;CHR$(157);CHR$(132);A$
223 PRINT TAB(0,11);CHR$147;CHR$181;TAB(21);CHR$181; TAB(39);CHR$234
```

##### Adjustable

Key - set in lines 7 and 8

```none
FG=132 CHR$(132) -> CHR$FG    - Foreground graphics colour
FT=148 CHR$(148) -> CHR$FT    - Foreground text colour
BG=147 CHR$147   -> CHR$BG    - Background colour
BK=157 CHR$(157) -> CHR$BK    - Background control (Do NOT change!)
GC=135 CHR$135   -> CHR$GC    - Goods colour (white)
LB=181 CHR$181 -> CHR$LB      - Left half block
RB=234 CHR$234 -> CHR$RB      - Right half block
FB=255 CHR$255 -> CHR$FB      - Full block
```

Code

```none
7 FG=132:FT=148:BG=147:GC=135 : REM COLOUR VARIABLES
8 BK=157:LB=181:RB=234:FB=255: REM COLOUR CONSTANTS - DO NOT CHANGE
141 PRINT TAB(0,3);CHR$BG;CHR$BK;CHR$FT;"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(24,3);"HONGKONG";TAB(33,3);"GODOWN"

150 FORI=0TO5:P.TAB(0,4+I);CHR$BG;CHR$LB:P.TAB(2,4+I);CHR$GC;G$(I):P.TAB(9,4+I);CHR$BG;CHR$LB;CHR$GC:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330
151 P.TAB(22,4+I);CHR$BG;CHR$LB;CHR$GC:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(38,4+I);CHR$BG;CHR$RB:NEXTI
152 P.TAB(0,10);CHR$BG;CHR$BK;CHR$FG;A$:RETURN

220 GOSUB 790: GOSUB 1340: P.TAB(0,10);CHR$BG;CHR$BK;CHR$FT;TAB(8,10);L$(L);" MARKET PRICES":P.A$:FORI=0TO4STEP2: P.TAB(0,12+I/2)CHR$BG;CHR$LB;CHR$GC;G$(I);: P.TAB(8)CHR$GC:P.TAB(9,12+I/2)GP(I);
221 P.TAB(20,12+I/2)CHR$BG;CHR$LB;CHR$GC;G$(I + 1);:PRINT TAB(18) GP(I + 1) :P.TAB(38,12+I/2);CHR$BG;CHR$RB:NEXT I
222 P.TAB(0,15);CHR$BG;CHR$BK;CHR$FG;A$
223 PRINT TAB(0,11);CHR$BG;CHR$LB;TAB(21);CHR$LB; TAB(39);CHR$RB
```

##### Adjustable - more efficient

Key - set in lines 7 and 8

```none
FG$= CHR$(132) - Foreground graphics colour
FT$= CHR$(148) - Foreground text colour
BG$= CHR$147   - Background colour
BK$= CHR$(157) - Background control (Do NOT change!)
GC$= CHR$135   - Goods colour (white)
LB$= CHR$181   - Left half block
RB$= CHR$234   - Right half block
FB$= CHR$255   - Full block
```

Code

```none
7 FG$=CHR$132:FT$=CHR$148:BG$=CHR$147:GC$=CHR$135 : REM COLOUR VARIABLES
8 BK$=CHR$157:LB$=CHR$181:RB$=CHR$234:FB$=CHR$255: REM COLOUR CONSTANTS - DO NOT CHANGE
141 PRINT TAB(0,3)BG$BK$FT$;"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(24,3);"HONGKONG";TAB(33,3);"GODOWN"

150 FORI=0TO5:P.TAB(0,4+I)BG$LB$:P.TAB(2,4+I);GC$;G$(I):P.TAB(9,4+I)BG$LB$GC$:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330
151 P.TAB(22,4+I)BG$LB$GC$:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(38,4+I)BG$RB$:NEXTI
152 P.TAB(0,10)BG$BK$FG$;A$:RETURN

220 GOSUB 790: GOSUB 1340: P.TAB(0,10)BG$BK$FT$;TAB(8,10);L$(L);" MARKET PRICES":P.A$:FORI=0TO4STEP2: P.TAB(0,12+I/2)BG$LB$GC$;G$(I);: P.TAB(8)GC$:P.TAB(9,12+I/2)GP(I);
221 P.TAB(20,12+I/2)BG$LB$GC$;G$(I + 1);:PRINT TAB(18) GP(I + 1) :P.TAB(38,12+I/2)BG$RB$:NEXT I
222 P.TAB(0,15)BG$BK$FG$;A$
223 PRINT TAB(0,11)BG$LB$;TAB(21)LB$; TAB(39)RB$
```

```
220 GOSUB 790: GOSUB 1340: P.TAB(0,10)BG$BK$FT$;TAB(8,10);L$(L);" MARKET PRICES":P.A$:FORI=0TO4STEP2: P.TAB(0,12+I/2)BG$LB$GC$G$(I);: P.TAB(8)GC$:P.TAB(9,12+I/2)GP(I);:P.TAB(20,12+I/2);G$(I + 1);
221 PRINT TAB(18) GP(I + 1) :P.TAB(38,12+I/2)BG$RB$:NEXT I
222 P.TAB(0,15)BG$BK$FG$A$
223 PRINT TAB(0,11)BG$LB$: PRINT TAB(38,11)BG$RB$
```

### Complete green monochrome patch for goods table and market prices

With first line empty (as per Apple II)

#### With center bar

##### Adjustable - more efficient

Key - set in lines 7 and 8

```none
FG$= CHR$(132) - Foreground graphics colour
FT$= CHR$(148) - Foreground text colour
BG$= CHR$147   - Background colour
BK$= CHR$(157) - Background control (Do NOT change!)
GC$= CHR$135   - Goods colour (white)
LB$= CHR$181   - Left half block
RB$= CHR$234   - Right half block
FB$= CHR$255   - Full block
HC$= CHR$148   - Heading text colour
```

Code

```none
7 FG$=CHR$146:FT$=CHR$130:BG$=CHR$146:GC$=CHR$135:SR$=CHR$(129):SG$=CHR$(130):SGG$=CHR$(146): REM COLOUR VARIABLES
8 BK$=CHR$157:LB$=CHR$181:RB$=CHR$234:FB$=CHR$255:LG$=CHR$(141):FLSH$=CHR$(136):STDY$=CHR$(137): REM COLOUR CONSTANTS - DO NOT CHANGE

20 PRINT TAB(0,3)SGG$;:FOR N=1 TO 37: PRINT FB$;: NEXT N: PRINT FB$:PRINT: PRINT TAB(12)LG$SG$ "T A I P A N:":PRINT TAB(12)LG$SG$ "T A I P A N:": PRINT TAB(12)SG$"_________________"
21 SPEED = 100:PRINT TAB(13,8)SG$"A  G A M E  I N":PRINT TAB(14)SG$"C O N T E X T": PRINT:PRINT TAB(13)SG$"HAYDEN BOOK CO."
22 SPEED = 255:PRINT TAB(0,14)SGG$;:FOR N=1 TO 37: PRINT FB$;: NEXT N: PRINT FB$

141 PRINT TAB(0,3)BG$BK$FT$;"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(24,3);"HONGKONG";TAB(33,3);"GODOWN"

150 FORI=0TO5:P.TAB(0,4+I)BG$LB$:P.TAB(2,4+I)GC$;G$(I):P.TAB(9,4+I)BG$LB$GC$:P.TAB(12,4+I);: Q=SG(I):GOSUB 1330
151 P.TAB(22,4+I)BG$LB$GC$:P.TAB(27,4+I);:Q=GG(I):GOSUB 1330:P.TAB(38,4+I)BG$RB$:NEXTI
152 P.TAB(0,10)BG$BK$FG$;A$:RETURN

220 GOSUB 790: GOSUB 1340: P.TAB(0,10)BG$BK$FT$;TAB(8,10);L$(L);" MARKET PRICES":P.A$:FORI=0TO4STEP2: P.TAB(0,12+I/2)BG$LB$GC$;G$(I);: P.TAB(8)GC$:P.TAB(9,12+I/2)GP(I);
221 P.TAB(20,12+I/2)BG$LB$GC$;G$(I + 1);:PRINT TAB(18) GP(I + 1) :P.TAB(38,12+I/2)BG$RB$:NEXT I
222 P.TAB(0,15)BG$BK$FG$;A$
223 PRINT TAB(0,11)BG$LB$;TAB(21)LB$; TAB(39)RB$

590 PRINT TAB(6,14)SG$"PRESS <";:FLASH=1:PRINT FLSH$"SPACEBAR"STDY$;:NORMAL=1:PRINT "> TO START";SGG$ : GOSUB 60:IF X$ = " " THEN RETURN
```

and the indents

```none
7 FG$=CHR$146:FT$=CHR$130:BG$=CHR$146:GC$=CHR$135:SR$=CHR$(129):SG$=CHR$(130):SGG$=CHR$(146): REM COLOUR VARIABLES
8 BK$=CHR$157:LB$=CHR$181:RB$=CHR$234:FB$=CHR$255:LG$=CHR$(141):FLSH$=CHR$(136):STDY$=CHR$(137): REM COLOUR CONSTANTS - DO NOT CHANGE

130 VDU12:PRINT TAB(0,0)FT$"PORT ";L$(L);: PRINT TAB(27,0);M$(M);". ";DA+1;",";Y
140 INVERSE=0:PRINT TAB(0,1)FT$"CASH ";: Q = C:GOSUB 1330:NORMAL=0:PRINT TAB(27,1); "GUNS ";:Q=G:GOSUB 1330: PRINT TAB(0,2)FT$"DEBT ";: Q = D:GOSUB 1330:PRINT TAB(27,2); "HOLD ";: Q = SH:GOSUB 1330

230 PRINT: PRINT TAB(0,17)FT$"B)UY, S)ELL, L)EAVE, OR R)ETIRE?" 

250 PRINT TAB(0,17)A$;:PRINT TAB(0,17)FT$T$;" ";TC$;"?": GOSUB 260:GOTO 280

470 GOSUB 130: GOSUB 1340:PRINT  TAB(0,11)FT$"[WANCHAI DISTRICT OF HONGKONG: HOME OF "; W$;"]":PRINTFT$W$;" GREETS YOU, TAIPAN, AND WISHES YOU WELL.":GOSUB 780:PRINT  TAB(0,13)A$:PRINT A$
480 PRINT  TAB(0,13)FT$"WU STATES THAT HIS IRON LOTUS": PRINTFT$"TRIAD HAS BEEN WATCHING YOU.": GOSUB 780
490 PRINT  TAB(0,13)A$: PRINT A$: PRINT A$:PRINT TAB(0,13)FT$W$;" ASKS, DO YOU":PRINT A$;:PRINT TAB(0,14)FT$"WISH TO B)ORROW, P)AY, OR Q)UIT?    ";: NUM$ = ""

510 IF LD = 1 AND (B = 1 OR D > 1E4) THEN GOSUB 1340: PRINT TAB(0,14)FT$"WU REGRETS THAT HE CANNOT LOAN": PRINTFT$"YOU MORE AT THIS TIME, TAIPAN.";: GOSUB 760:GOTO 490

520 GOSUB 1340: PRINT TAB(0,14)FT$"HOW MUCH DO YOU WISH TO ";LD$;",":PRINT FT$"TAIPAN? ";:GOSUB 310:NUM = VAL (NUM$)

541 IF NUM > 2 * C THEN PRINT  TAB(0,14)A$:PRINT  TAB(0,14)FT$W$;" REGRETS THAT HE":PRINT FT$"CANNOT LOAN YOU THAT MUCH.";: GOSUB 760:GOTO 490

560 IF NUM > C THEN NUM = C: D = D - C:C=0:GOSUB 130:PRINT  TAB(0,14)FT$W$;" THANKS YOU,":PRINT FT$"TAIPAN, FOR THE PAYMENT.";: GOSUB 780:IF D < 0 THEN D = 0:GOSUB 130:GOTO 490

570 IF NUM > D THEN D = 0: C = C - NUM:PRINT  TAB(0,14) FT$W$;" THANKS YOU FOR YOUR STARTLING GENEROSITY!";:GOSUB 130:GOSUB 780:GOTO 490
580 C = C - NUM: D = D - NUM:GOSUB 130:PRINT  TAB(0,14)FT$W$;" ACCEPTS YOUR PAYMENT WITH GRATITUDE, TAIPAN   ";:GOSUB 780: GOTO 490

830 IF C > 100 AND TR = 0AND L = 0 THEN GOSUB 1340:P. TAB(0,11)FT$YS$;", LIEUTENANTS":P. TAB(0,12)FT$"OF THE MARINER, ";LY$;", ASK IF": P. TAB(0,13)FT$"YOU WILL DONATE ";:Q = DN:GOSUB 1330:P. TAB(0,14)FT$"TO THE TEMPLE OF TIN HAU, THE"
831 IF C > 100 AND TR = 0 AND L = 0THEN PRINT TAB(0,15)FT$"SEA GODDESS. (Y) OR (N)"

840 IF C > 0 AND TR = 0 AND L = 0 THEN GOSUB 60: IF X$ = "Y" THEN C = C - DN:TR = 1:GOSUB 130:GOSUB 1340:PRINT TAB(0,11)FT$" ":PRINT " " ;YS$;" THANK": PRINT TAB(0,12)FT$" YOU, AND DEPART.": PRINT:PRINT A$;
841 IF C > 0 AND TR = 0 AND L = 0THEN IF X$ <> "Y" THEN GOSUB 1340: PRINT TAB(0,11)FT$" ":PRINT TAB(0,12)FT$" ";YS$;" DEPART":PRINT TAB(0,13)FT$"ABRUPTLY IN A CHILLY SILENCE.": PRINT A$;
```

[![BBCTaipan_MODE7_Monochrome_InvisibleHeadings][1350]][1350]

It becomes apparent that the heading colour, which is blue, and is only ever printed upon an "inverse" background, needs its own "variable constant" (i.e.`HC$=CHR$148`), as the `FG$` and `FT$` are used by the rest of the information on the screen.  

Fortunately, only three lines need to be changed:

```none
7 FG$=CHR$146:FT$=CHR$130:BG$=CHR$146:GC$=CHR$135:HC$=CHR$148:SR$=CHR$(129):SG$=CHR$(130):SGG$=CHR$(146): REM COLOUR VARIABLES
141 PRINT TAB(0,3)BG$BK$HC$;"GOODS"; TAB(10,3);"ABOARD";TAB(17,3);"SHIP";TAB(24,3);"HONGKONG";TAB(33,3);"GODOWN"
220 GOSUB 790: GOSUB 1340: P.TAB(0,10)BG$BK$HC$;TAB(8,10);L$(L);" MARKET PRICES":P.A$:FORI=0TO4STEP2: P.TAB(0,12+I/2)BG$LB$GC$;G$(I);: P.TAB(8)GC$:P.TAB(9,12+I/2)GP(I);
```

[![BBCTaipan_MODE7_Monochrome_VisibleHeadings][1351]][1351]

The goods colour is still white – just change `GC$` from `135` to `130`.

---

## Conclusion

### Reaching a settlement

It was necessary to settle with a 39 column screen, in the form of `MODE 7`, due to:

 - Memory considerations, mostly

### A tale of two parts

The code is in two of three parts:

 - Choose between:
   - Colour patch, see [Complete colour patch for goods table and market prices](#complete-colour-patch-for-goods-table-and-market-prices)
   - Black and White patch, see [Complete B/W patch for goods table and market prices](#complete-B/W-patch-for-goods-table-and-market-prices)
 - Add the 39 column indentation, see [Separated main screen left-hand indent](#separated-main-screen-left-hand-indent)

### Making the best of two bad situations

Out of the two options, I prefer the colour tables as they *seem* less "jarring":

#### Black and white

[![BBCTaipan_MODE7_MarketTableBottom_MiddleBar_Good][1071]][1071]

#### Colour

[![BBCTaipan_MODE7_ColourMarketTableBottom_MiddleBar_Good][1097]][1097]




This is the end, my friend

---

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
  [1022]: mode7images/BBCTaipan_MODE7_BadTableEnds.png
  [1023]: mode7images/BBCTaipan_MODE7_FleetingGlimpse.png
  [1024]: mode7images/BBCTaipan_MODE7_IndentedTableEnds.png
  [1025]: mode7images/BBCTaipan_MODE7_IndentedTableEnds_GoodsL.png
  [1026]: mode7images/BBCTaipan_MODE7_IndentedTableEnds_GoodsLR.png
  [1027]: mode7images/BBCTaipan_MODE7_IndentedTableEnds_GodownEdge.png
  [1028]: mode7images/BBCTaipan_MODE7_IndentedTableEnds_NoBlocks.png
  [1029]: mode7images/BBCTaipan_MODE7_IndentedTableEnds_NoBlocksNorSpaces.png
  [1030]: mode7images/BBCTaipan_MODE7_IndentedTableEnds_MarketBlocks.png
  [1040]: mode7images/BBCTaipan_MODE7_IndentedTableEnds_MaxInt.png
  [1050]: mode7images/BBCTaipan_MODE7_ColourTable.png
  [1051]: mode7images/BBCTaipan_MODE7_ColourTable_WhiteGoods.png
  [1052]: mode7images/BBCTaipan_MODE7_ColourTable_WhiteGoodsShifted.png
  [1053]: mode7images/BBCTaipan_MODE7_BonG_ColourTable_WhiteGoodsShifted.png
  [1054]: mode7images/BBCTaipan_MODE7_BonG_ColourTable_AllGreen.png
  [1060]: mode7images/BBCTaipan_MODE7_MarketTableBad.png
  [1061]: mode7images/BBCTaipan_MODE7_MarketTableStillBad.png
  [1062]: mode7images/BBCTaipan_MODE7_MarketTableStillBad2.png
  [1063]: mode7images/BBCTaipan_MODE7_MarketTableStillBad2_ExplictTAB.png
  [1064]: mode7images/BBCTaipan_MODE7_MarketTableBetter.png
  [1065]: mode7images/BBCTaipan_MODE7_MarketTableBetter_LJust.png
  [1066]: mode7images/BBCTaipan_MODE7_MarketTableBetter_LJustRJust.png
  [1067]: mode7images/BBCTaipan_MODE7_MarketTableBottom.png
  [1068]: mode7images/BBCTaipan_MODE7_MarketTableBottom_EmptyTop.png
  [1069]: mode7images/BBCTaipan_MODE7_MarketTableBottom_ShiftUp.png
  [1070]: mode7images/BBCTaipan_MODE7_MarketTable_AppleII.png
  [1071]: mode7images/BBCTaipan_MODE7_MarketTableBottom_MiddleBar_Good.png
  [1080]: mode7images/BBCTaipan_MODE7_ColourMarketTable.png
  [1081]: mode7images/BBCTaipan_MODE7_ColourMarketTable_RJust.png
  [1082]: mode7images/BBCTaipan_MODE7_ColourMarketTable_RJust_Bad1.png
  [1083]: mode7images/BBCTaipan_MODE7_ColourMarketTable_RJust_Bad2.png
  [1084]: mode7images/BBCTaipan_MODE7_ColourMarketTable_RJust_TAB19.png
  [1085]: mode7images/BBCTaipan_MODE7_ColourMarketTable_RJust_TAB18.png
  [1086]: mode7images/BBCTaipan_MODE7_ColourMarketTable_RJust_TAB17.png
  [1087]: mode7images/BBCTaipan_MODE7_ColourMarketTable_RJust_TAB20.png
  [1090]: mode7images/BBCTaipan_MODE7_ColourMarketTableBottom.png
  [1091]: mode7images/BBCTaipan_MODE7_ColourMarketTableBottom_FixedRiceNope.png
  [1092]: mode7images/BBCTaipan_MODE7_ColourMarketTableBottom_FixedRiceMeh.png
  [1093]: mode7images/BBCTaipan_MODE7_ColourMarketTableBottom_EmptyTop.png
  [1094]: mode7images/BBCTaipan_MODE7_ColourMarketTableBottom_ShiftUp.png
  [1095]: mode7images/BBCTaipan_MODE7_ColourMarketTableBottom_MiddleBar_EmptyLine.png
  [1096]: mode7images/BBCTaipan_MODE7_ColourMarketTableBottom_MiddleBar_bad.png
  [1097]: mode7images/BBCTaipan_MODE7_ColourMarketTableBottom_MiddleBar_Good.png
  [1300]: mode7images/BBCTaipan_MODE7_FunkySplash.png
  [1301]: mode7images/BBCTaipan_MODE7_GreenSplash.png
  [1350]: mode7images/BBCTaipan_MODE7_Monochrome_InvisibleHeadings.png
  [1351]: mode7images/BBCTaipan_MODE7_Monochrome_VisibleHeadings.png
  [1500]: mode7images/BBCTaipan_MODE7_Inverse_Bad.png

## TODO

 - Tidy BBC floppy taipan - DONE!
 - Add BBC Floppy image taipan - DONE!
 - Update WOZ disk for Apple II taipan
 - Clean Taipan file.
 - Move weird Taipan to xtras/
 - Why are the TABs so weird in lines 220 and 221
 - Add center market prices bar for BW - DONE!
 - Add conclusion - DONE!
 - `MAXINT` test
 - Other dark/light colour combinations:
   - Red on Cyan
     - `10 PRINT CHR$145;CHR$157;CHR$150;"HI"`
   - Green on Magenta
     - `20 PRINT CHR$149;CHR$157;CHR$146;"HI"`
   - Blue on Green
     - `30 PRINT CHR$148;CHR$157;CHR$146;"HI"`
   - Examples with images
   - Should use variables, i.e. `FG` and `BG`
     - Versatile
     - Save RAM space, and characters used per line, by one byte/character per `CHR$`
     - `40 FG = 146: BG = 149: BK = 157`
     - `50 PRINT CHR$BG;CHR$BK;CHR$FG;"HI"`
 - Make a totally green monochrome UI
   - Use Blue on Green as inverse
 - Add *all* character codes, i.e. `T_G$` and `G_G$` for text green and graphics green respectively  
 - Why multiple lines 141? Was it development of the colour UI? Yes, see **Using colour (blue on yellow) again**. Still present in **Final complete colour patch for goods table - Blue on yellow**. Search for `REM remove:`
 - Why multiple lines 221? See **#### Top line – empty or shift?**. Was it development of the colour UI? Yes, see **Using colour (blue on yellow) again**. 

  - Use pre-greening (control coharacters down the left sid eof the screen, technique from video. Make it a subroutine.
    - By pre-greening, everything will be indented. However, the `TAB` statement would need to change to avoid overwriting the control character at `TAB(0)`
  - Every string printed should be preceded by a string varaiable. That way the colour can be changed globally. This can be done for, and then derived from, the green monochrome verison.
    - However, if every `PRINT` is preficed with a string variable, then the `TAB` value can remain unchanged - easier!
  - MAke a PRINT AT, or LOCATE routine for PET


### More TODO 

 - Add offset (constant) variables:
   - x - For non graphics characters (at the start of the line) lines without graphics control characters, *at the start* of the line.
   - y - For centralising the screen(?). We have more lines on the BBC (25)  vs Apple II (24)
 - Interesting?: http://www.riscos.com/support/developers/bbcbasic/part2/screenmodes.html
 - Add index/contents (TODO: Compare to links in the SSD1306 readme.md).
   - [Anchors in Markdown](https://gist.github.com/asabaylus/3071099)

 - Show fonts in modes 1,4 and 6 - DONE!
 - Show MODE errors for 1 and 4 - DONE!
 - Use custom graphics characters for inverse letters
   - But `MODE 7` does not have custom characters!
 - Make yellow and blue table - DONE!
 - Make table for Market Prices - DONE!

## Useful tables

### Control codes summary
|Code|	Attribute|
|---|---|
|129	|alphanumeric red|
|130	|alphanumeric green|
|131	|alphanumeric yellow|
|132	|alphanumeric blue|
|133	|alphanumeric magenta|
|134	|alphanumeric cyan|
|135	|alphanumeric white|
|136	|flash|
|137	|steady|
|140	|normal height|
|141	|double height|
|145	|graphics red|
|146	|graphics green|
|147	|graphics yellow|
|148	|graphics blue|
|149	|graphics magenta|
|150	|graphics cyan|
|151	|graphics white|
|152	|conceal|
|153	|contiguous graphics|
|154	|separated graphics|
|156	|black background|
|157	|new background|
|158	|hold graphics|
|159	|release graphics|

Source: [BBC BASIC for Windows - MODE 7 - Teletext](https://www.bbcbasic.co.uk/bbcwin/manual/bbcwinh.html)
