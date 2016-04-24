---
layout: post
title:  "Perspective"
date:   2016-04-23-18:40
categories: Vic20
---

Oh dear.

I'd really hoped that the distance effect I was seeking to achieve would be
convincing without any fancy perspective, but I have to admit that that is not
the case.  I will have to resort to some sort of simulated perspective.
Of course, I cannot really do perspective.  And honestly, whatever I come up
with is going to have to be a drastic compromise in more than one regard.
There's simply nowhere near enough raster time left for that to not be the case.


So, for one thing, I will probably seriously limit the number of objects on
screen.  That means I'll probably have to try hacking in some cheap rendering
to maintain the sense of world.  But I'll worry about that later.


As far as the parallax goes, here's what I'm thinkin'.

I probably cannot afford a huge table to manage the offset (like 1-byte per
raster line), but I will probably need a table of some size- maybe 1 byte per
4 lines or so.

The render loop will then be modified thusly:

Before it was a simple load/store copy:

```
.blit
  lda (.src),y
  sta (.dst),y
  dey
  bpl .blit
```

Since we're now dealing with sprites across columns, that already complicates
it:

```
.blit_r
  lda #$00
  sta tmp
  lda (.src),y
  ldx shamt
  beq .cnt
.shift
  lsr
  ror tmp
  dex
  bpl .shift
.cnt
  sta (.dst),y
  lda tmp
  sta (.dst2),y
  dey
  bpl .blit
```

We have two destination bytes: 1 for the left column of the shifted sprite data
and 1 for the right column.  No big deal- I would've more than likely added
this at some point anyway (if I wanted ANY x-motion).
This will need to be optimized obviously...The obvious method is to buffer the
previous sprite (meaning less shifting if sprites' per-frame motion is small).
That will come later..


The other modification is to our calculation of the destination.

Before:

```
  ldy #SPRITE_XOFFSET
  lda (.sprite),y
  lsr
  lsr
  lsr
  tax
  lda columns,x
  clc
  adc .ypos
  sta .dst
  lda columns+1,x
  adc #$00
  sta .dst+1
```

And with our table-offset modification:

```
  ldy #SPRITE_XOFFSET
  lda (.sprite),y
  lsr
  lsr
  lsr
  tax
  lda columns,x
  ldx .ypos
  clc
  adc parallax,x
  adc .ypos
  sta .dst
  lda columns+1,x
  adc #$00
  sta .dst+1
```

This does not factor in, for example, the camera's x-position.  So for the time
being consider the camera fixed along the x-axis. :)

-Gummy
