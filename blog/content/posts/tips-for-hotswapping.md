+++
title = "Tips for Hotswapping"
date = "2022-08-15T10:33:48-07:00"
author = "fr33r"
cover = "img/hotswap-tips-and-tricks.webp"
tags = ["mechanical", "keyboard", "tips",]
keywords = ["hotswap", "mechanical", "keyboard"]
description = "Some tips & tricks to make building hotswap mechanical keyboards easier."
showFullContent = false
readingTime = true
hideComments = false
+++

When I build mechanical keyboards, I almost always introduce [hotswap][hotswap] support.
The dogmatic battles wage on over whether or not hotswap boards provide an
equivalent or better experience as non-hotswapped versions, but it really just
comes down to preference.

# Why I Hotswap

For me building in hotswap support is a no brainer. Here are some reasons why
it is the ideal choice for me.

## No Desoldering Switches

Obviously, having a hotswap board means that you can easily pop out switches
and put in new ones. There are an insane number of variations of switches out
there, and unless you plan on building a particular configuration and then turning
around and selling it, I'd be willing to bet that you would at least entertain
the idea of changing the switches out at some point.

Some say that instead of hotswapping, you could desolder your switches when you
decided to change it up. They aren't wrong, but let's be real: desoldering sucks.
It requires extra gear ([desoldering][desoldering] [pumps][desoldering-pump]
at least) and takes way more time than popping out the existing switches and
putting new ones in on a hotswap board.

Most importantly, desoldering requires getting the soldering iron out and heating
up each joint, which could lead to a greater risk of accidental damage.

## Switch Compatibility

Most of the popular switches are compatible with hotswap sockets. You won't feel
limited in your choices purely by making the decision to hotswap.

> _**NOTE**_: Check out my tip below to see how to best determine what switches
will be compatible with your sockets.

## Iterability

I personally like to keep the door open. You never know what new switch may
come out in the future. Also, it will take time for you to try different
switches out to see what you like. You may even change your opinion of a
particular switch over time as you get to use it more and more.

Take the pressure off of buying the "perfect" switches. Like I already
mentioned, there are almost too many to count, each having their own
characteristics.

## Board Scarcity

It is very common for various build kits to be a part of a group buy. These
group buys can be very infrequent, and they often take many months to fulfill.
As such, it simply isn't a viable option to sell your board with your existing
switches in hopes of getting the same PCB and trying out different ones.

By hotswapping the PCB, you can keep your board and instead give some new
switches a spin.

# Tips & Tricks

## Check Switch Compatibility

Not all switches can be hotswapped. More specifically, the contacts may not fit
inside the sockets you have chosen.

Thankfully, the mechanical keyboard community has open sourced maintaining a
[shared spreadsheet][socket-compatibility] that outlines switch
compatibilities for Mill Max 7305 and 0305 sockets. Kailh MX hotswap sockets
are advertised as being completely compatible for any MX switch.

> _**NOTE**_: I've hotswapped boards using both Kailh sockets and Mill Max
7305 sockets, and have never had any issues with switch compatibility.

According to [Deskthority][hotswap-sockets], there are other kinds of hotswap
(& pin; using them interchangeably) sockets, and to be
totally transparent, I've never seen these sold from any retailers I've visited,
nor seen conversation of them where I lurk on the interwebs. As such, I can't
comment on these, besides just suggesting that you'll increase the chance of
success by using sockets others are familiar with and have tested.

## Hold Sockets Using Painters / Masking Tape

{{< figure src="/img/taped-board.jpg" alt="Taped PCB" position="center" caption="Sockets taped to PCB.">}}

The thought of adding sockets to the board can seem daunting at first. For me,
separating the step of inserting the sockets (and holding them in place) from
the soldering work helps with approaching the build in a more modular way.

For this task, the secret weapon is tape; more specifically, painters tape or
[masking tape][masking-tape].

> _**WARNING**_: Do not use [Scotch magic tape][scotch-tape] or
[packing tape][packing-tape]. These will melt onto the top of the sockets when
you solder them in. Additionally, both of these kinds of tape easily tear
when attempting to remove it, where painters or masking tape is less likely to
do so.

Insert sockets one row at a time. When you have finished inserting the sockets
for a row, tear off a single piece of tape that is long enough to cover then
entire row, with at least `2` or `3` inches extra on each end of the board. Using
a single piece of tape makes it super easy to remove the tape after the soldering
is complete. The extra tape on the sides can be used to fold over itself,
essentially creating makeshift "tabs" that you can grab onto easily when removing.

> _**NOTE**_: Creating these tabs on the ends isn't strictly necessary, but
strongly recommended. You'll be scratching at the PCB attempting to lift the
tape otherwise, which can scuff or even damage the board if careless.

When moving on to additional rows, attempt to minimize the amount of overlap
between the tape for each row. You want to make sure that as you peel off one
row, that you don't accidentally peel off the tape for an adjacent row. Going
one row at a time ensures you don't remove tape that you may need to reapply
(if you find a mistake) and makes it easier to flip the board around as you
inspect it.

As you peel the tape off each row after soldering, test each set of sockets with
a switch to ensure that it inserts properly.

## Buy a Solid Pair of Tweezers

{{< figure src="/img/tweezers.jpeg" alt="Curved Point Tip Tweezers" position="center" caption="VETUS curved point tip tweezers.">}}

No, don't use any tweezers you might already have laying around. I strongly suggest
getting a pair that have a curved point tip, as it is way more ergonomic for
this use case. They are super affordable; usually `$5` to `$10` dollars.

> _**BONUS**_: If you are delicate enough, you can insert the pointed tips of the
tweezers into the socket, and then insert the socket into the board. After some
practice, you can go beast mode and put two sockets on the tweezers at once!

## Rig Up a Raised Working Surface

{{< figure src="/img/helping-hands.jpeg" alt="Helping Hands Tool" position="center" caption="Helping hands tool.">}}

When inserting the sockets into the board, I strongly recommend that you
raise the board off of your working surface. You'll benefit from this because
the sockets themselves are taller than the thickness of the PCB - they won't be
able to be inserted fully otherwise.

The solutions are endless here, but don't overthink it. If you have two sets of
[helping hands][helping-hands], having those set up such that each set holds up one side of the
PCB could do the trick. If you have two spare rolls of tape, you could place each
down on your surface to prop up two opposite sides. You could even snag two
pencils or pens to use instead of the rolls of tape; just make sure they can't
roll!

## Swap with Intention

{{< figure src="/img/hotswap-switches.webp" alt="Hotswap Switches" position="center" caption="Hotswap switches.">}}

Something that doesn't get emphasized too often is that just because you can
change out your switches, doesn't mean you should do it just to do it. In other
words, the sockets themselves have lifespans that you could technically measure
in the number of swaps, just as you often see the lifespan of switches measured
in keystrokes.

> _**NOTE**_: The Kailh hotswap sockets are [rated for 100 swaps][kailh-lifespan].
There doesn't appear to be explicit mentioning of the lifespan for Mill Max
sockets.

My recommendation is to view the ability to hotswap as a 'future proofing'
characteristic, rather than a daily or weekly activity. It keeps the door open
for you to change your switches if you decided to do so, allowing you to
iterate on your keyboard more gracefully over time.

## Buy Extra Sockets

{{< figure src="/img/mill-max-sockets.webp" alt="Mill Max Sockets" position="center" caption="Mill Max 3305 Sockets" >}}

The sockets can be tiny (`2.67mm` tall for the
[Mill Max 7305][mill-max-7305] for example). You will almost certainly drop a
couple of these and have no clue where they went.

In addition to the reality that you may lose some of them, it's also good to
have extras in case you accidentally apply too much solder and get solder inside
the socket itself. Once solder is in the socket, there is no fixing it - consider
it a cost of doing business!

Lastly, once you do it once, it's likely you'll want to do it again. Maybe you
have a keyboard with number keys, such as a 40%, 60%, 65%, etc. and you later
decided you want a sidecar numpad. Maybe you want to build another board altogether.
You don't want to be halfway through a build and find out you don't enough
sockets.

# Downsides

Building a mechanical keyboard with hotswap support isn't a home run; it does
come with some downsides.

Most notably, it will be more expensive. Mill max sockets aren't cheap, and
depending on the size of your board, you may use a lot of them (two per switch).
Luckily, most online retailers for the sockets provide bulk discount rates. If
you can swing it, I strongly recommend buying enough to take advantage of some
of these discounts.

> _**EXAMPLE**_: For example, buying `499` [Mill Max 7305][mill-max-7305]
sockets on Digikey will cost you `$158.02` (`$0.31668` each), while `500`
will cost `$150.79` (`$0.30158` each), which is `~4.8%` discount. And yep you
read that right, you save `$7` even though you are getting one more socket.

Next, it will require a little more upfront time investment. You'll be working
with tweezers to insert each socket, taping them in place, and soldering them in.
Overall though, I've found in practice it doesn't add too much time;
it's the same amount of soldering, so really it comes down to the prep work.

Lastly, they are a little trickier to solder than the pins on a switch. Depending
on whether you get longer sockets, (like the [0305s][mill-max-0305]; `3.94mm`),
or shorter ones (like the [7305s][mill-max-7305]; `2.67mm`), there is an
increased likelyhood you'll accidentally get solder in the socket itself and
need to replace it altogether.

# Anything I Missed?

These tips & tricks are the ones I have found most impactful during my builds
when it comes to hotswapping. I hope that they will help you as well.

Building these boards is a bit of a craft. I'd love to hear about other approaches
that you might have taken!

[mill-max-7305]: https://www.digikey.com/en/products/detail/mill-max-manufacturing-corp/7305-0-15-15-47-27-10-0/1765737
[mill-max-0305]: https://www.digikey.com/en/products/detail/mill-max-manufacturing-corp./0305-2-15-80-47-80-10-0/2639493?utm_adgroup=Terminals%20-%20PC%20Pin%20Receptacles%2C%20Socket%20Connectors&utm_source=google&utm_medium=cpc&utm_campaign=Shopping_Product_Connectors%2C%20Interconnects_NEW&utm_term=&utm_content=Terminals%20-%20PC%20Pin%20Receptacles%2C%20Socket%20Connectors&gclid=Cj0KCQjwgO2XBhCaARIsANrW2X0ZAFszkHLN835TWu_m3qZyo4K4jiFYgfiW2spKUK9OuxyNLCD5oZwaAs34EALw_wcB
[scotch-tape]: https://en.wikipedia.org/wiki/Scotch_Tape
[packing-tape]: https://en.wikipedia.org/wiki/Box-sealing_tape
[desoldering]: https://en.wikipedia.org/wiki/Desoldering
[socket-compatibility]: https://docs.google.com/spreadsheets/d/1NhrXy6k88eY9bBqVuPWTAGW2q3GzszJ1JH-zuuGQ-iU/edit#gid=0
[desoldering-pump]: https://www.amazon.com/Teenitor-Solder-Sucker-Desoldering-Removal/dp/B0739LXQ6N/ref=zg_bs_8107034011_sccl_2/140-5809656-7869339?pd_rd_i=B0739LXQ6N&psc=1
[hotswap]: https://deskthority.net/wiki/Hot-swap
[hotswap-sockets]: https://deskthority.net/wiki/Hot-swap#Hot-swap_switch_sockets
[helping-hands]: https://www.amazon.com/Neiko-01902-Adjustable-Magnifying-Alligator/dp/B000P42O3C/ref=asc_df_B000P42O3C/?tag=hyprod-20&linkCode=df0&hvadid=312096335436&hvpos=&hvnetw=g&hvrand=4691685772583898714&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9031554&hvtargid=pla-448870101576&psc=1
[kailh-lifespan]: https://divinikey.com/products/kailh-hot-swap-sockets
[masking-tape]: https://en.wikipedia.org/wiki/Masking_tape
