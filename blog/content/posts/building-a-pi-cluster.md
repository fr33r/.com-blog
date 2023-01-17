+++
title = "Building a Pi Cluster"
date = "2022-08-06T21:22:01-07:00"
publishDate = "2022-08-31T21:22:01-07:00"
author = "fr33r"
cover = "img/jon_blog_building_raspberry_pi_cluster_cover.png"
tags = ["build", "tutorial", "nerdy"]
keywords = ["hardware", "raspberry", "pi", "cluster"]
description = "My journey of building a Raspberry Pi cluster."
showFullContent = false
readingTime = true 
hideComments = false
+++

Having been a software engineer for years, I had heard about Raspberry Pi
computers, and the tinkering people were doing with them. Despite the chatter,
I didn't have any sort of project in mind, and honestly didn't take the time
to learn about them.

One year when the holiday season began rolling around, the
team decided to partake in a "secret santa" gift giving activity. Without any
better ideas, I put down that I was interested an a Raspberry Pi.

Fast-forward three years later and I finally found a project that I was genuinely
excited about: creating a Raspberry Pi cluster. I dug up that model 3B+ that I
got from the gift exchange years prior to began setting it up for the first
time, and I've been at it ever since!

# Hardware

In order to build a Raspberry Pi cluster, you need some gear! Lets walk through
the various items I ended up with.

#### Pis

The most important part of the Pi cluster is... well... the Pis!

I already got one as a gift - check! The remaining three I needed to source internationally
(more on that later). To future proof, I was aiming for the model 4B. Although
it would've been nice to get my hands on a 4GB or 8GB configuration, they were
basically myths during these times. I settled for three 2GB models.

#### PoE+ Hats

Given power supplies aren't included with the Pi itself, I knew that powering
my cluster was going to involve spending some additional cheddar!

The [official power supply](https://www.adafruit.com/product/4298?gclid=Cj0KCQjwxb2XBhDBARIsAOjDZ37bms37yK_7GGNQR5pZ6oqvTZTzyaVe16QH8_NynBC47z0yp6GJZc0aAgFnEALw_wcB) goes for around $8 dollars. Knowing that I was going to shoot for four Pis, that was going
to cost me at least $32 dollars. That wasn't even including the money I'd need to spend
on zip ties and the time I'd need to spend to cable manage. Lastly, I simply
couldn't justify using four outlets!

This is where [Power over Ethernet (PoE)](https://en.wikipedia.org/wiki/Power_over_Ethernet) comes to the rescue. With the correct
networking equipment, enough power to drive the Pis can be delivered over a
simple ethernet cable. No more outlet waste or time spent cable managing! But
there is one catch.

The Pis themselves can't be powered over ethernet. An [additional hat](https://www.adafruit.com/product/5058) needs to
be purchased to achieve this, and they aren't exactly cheap. That said, they
are well worth the price. They are super easy to install on the Pis, plenty of
cases have enough space for them, and they solve the gripes around multiple
power supplies.

#### Switch

When it comes to PoE+ switches, the main thing to focus on is the number of ports.
Obviously, the more ports, the more you will end up spending. No more than four Pis
are needed for my build, so I went with the [Netgear GS305P (v2)](https://www.amazon.com/dp/B08LHL1Q2Z?psc=1&ref=ppx_yo2ov_dt_b_product_details). This switch
is [unmanaged](https://www.fieldengineer.com/blogs/network-switch-managed-vs-unmanaged#:~:text=An%20unmanaged%20switch%20is%20simple,systems%20to%20a%20larger%20network.) (no smartness) and has 5 ports, one of which being the ingress from your existing internet connection.

#### Storage

Each Pi has a micro SD card slot that acts as onboard storage. For my use cases,
I knew I was going to use a network attached storage (NAS) device to store all
of my data, so using a smaller SD card made sense. I ultimately landed on
snagging some [SanDisk 32GB cards](https://www.officedepot.com/a/products/8224492/SanDisk-Ultra-PLUS-microSD-Cards-32GB/), which will be plenty of space for my needs.

#### Cables

To deliver both power and data to the Pis, I needed some ethernet cables. My
only desire for these was that they were small (also called [patch cables](https://www.cables.com/cablesblog/ethernet-vs-patch-cables-whats-the-difference.html)).

I found that cables were typically sized in 6 inch increments. Many places
didn't sell patch cables smaller than 1 foot, but I eventually found [some](https://www.amazon.com/dp/B06XC5PZLV?psc=1&ref=ppx_yo2ov_dt_b_product_details) online
without too much trouble.

If you are looking for any slack at all, I'd recommend getting cables that are about
1 foot in length. Additionally, my cables all weren't the exact same size, so
if sizing consistency is super important, I wouldn't buy the cables I ended up
getting.

#### Case

Now that I had picked out all of my parts, I needed a case to put it in. After
a little digging, I stumbled upon [Uctronics](https://www.uctronics.com/) which produces many [products
specific to Pi clusters](https://www.uctronics.com/cluster-and-rack-mount/for-raspberry-pi/cluster.html).

In particular, I wanted something that could not only house my Pis (keeping in
mind they each would have a PoE+ hat installed), but also something that would
also house my PoE+ switch, and provide simple access to the Pis in case I needed
change anything.

# Cost Breakdown

Below outlines the cost of my build (excluding state-side taxes; VAT was included in some
of the purchases). Keep in mind that the chip shortage, high inflation, and
supply chain woes led to higher costs than what was likely achievable historically
for the same parts.

| Line Item                                                                                                        | Price ($)   |
|------------------------------------------------------------------------------------------------------------------|-------------|
| [Raspberry Pi 3B+ (1GB)](https://www.adafruit.com/product/3775)                                                  | $35.00      |
| Raspberry Pi 4B (2GB)                                                                                            | $52.92      |
| Raspberry Pi 4B (2GB)                                                                                            | $45.78      |
| Raspberry Pi 4B (2GB)                                                                                            | $45.79      |
| [Crappy Standoffs](https://www.adafruit.com/product/2336)                                                        | $8.16       |
| [Good Standoffs](https://www.mouser.com/ProductDetail/RAF-Electronic-Hardware/M2104-2545-AL?qs=2411ckis64rpgA%252BYlrCb%252Bg%3D%3D) | $22.73      |
| [Case](https://www.amazon.com/UCTRONICS-Upgraded-Enclosure-Raspberry-Removable/dp/B09JNHKL2N)                    | $69.99      |
| [Case Fan Adaptor](https://www.amazon.com/dp/B09TP9HT3C?psc=1&ref=ppx_yo2ov_dt_b_product_details)                | $5.99       |
| [Switch](https://www.amazon.com/dp/B08LHL1Q2Z?psc=1&ref=ppx_yo2ov_dt_b_product_details)                          | $69.00      |
| [PoE+ Hat](https://www.adafruit.com/product/5058)                                                                | $20.00      |
| [PoE+ Hat](https://www.adafruit.com/product/5058)                                                                | $20.00      |
| [PoE+ Hat](https://www.adafruit.com/product/5058)                                                                | $20.00      |
| [PoE+ Hat](https://www.adafruit.com/product/5058)                                                                | $20.00      |
| [Cables](https://www.amazon.com/dp/B06XC5PZLV?psc=1&ref=ppx_yo2ov_dt_b_product_details)                          | $14.49      |
| [Micro SD Card (x2)](https://www.officedepot.com/a/products/8224492/SanDisk-Ultra-PLUS-microSD-Cards-32GB/)      | $22.00      |
| [Micro SD Card (x2)](https://www.officedepot.com/a/products/8224492/SanDisk-Ultra-PLUS-microSD-Cards-32GB/)      | $22.00      |
| Shipping                                                                                                         | $76.76      |
| **Total**                                                                                                        | **$570.61** |

# What went not so well

#### Incomplete Case

Even though the quality of the case I got was solid, it was annoyingly incomplete.

The first strike with the case was that it came with fans that could be powered
by the Pi, but didn't include the adaptor necessary. I actually didn't realize
this until I was building the case, as the instruction manual that came with it
referenced the part but it wasn't included. I find it pretty strange that the fans
come with the case but the adaptor isn't included.

My next gripe was that the case advertises that it is compatible with the
PoE+ hat, but didn't come equipped with standoffs compatible with the case.
Instead it came with screws to secure the Pis to the trays, which would essentially
be all that is necessary if there were no hats installed. I give Uctronics more
of a pass here, given not all builds will use hats, but found the advertising
misleading. 

These shortcomings were the only reason I needed to purchase the standoffs and
the fan adaptor, resulting in a total investment in the case to be ~$85, which
I found to be pretty expensive - more expensive than the [case](https://www.amazon.com/Thermaltake-Suppressor-Certified-Computer-CA-1E6-00S1WN-00/dp/B017NWO5V2) I have for my mini-ITX
build.

#### (Lack of) Pi Availability

My interest in creating a Raspberry Pi cluster couldn't have been piqued at a
worse time in terms of availability. Due to both [supply constraints and increased
demand](https://www.raspberrypi.com/news/production-and-supply-chain-update/), they have been nearly impossible to comeby.

After quickly learning that there was no way I was going to be able to purchase
my Pis from [Adafruit](https://www.adafruit.com/), [Digikey](https://www.digikey.com/), or any other retailer I was a previous customer of,
I briefly looked at alternative marketplaces such as Facebook. It was clear that
was going to be a dead-end as well since the only sellers were gouging hard; we're
talking $100+ for model 4B (2G) which [normally retailed for $45](https://www.adafruit.com/product/4292).

I finally discovered a website called [rpilocator](https://rpilocator.com/), which was
completely dedicated to help solving this issue. It was through this website that
I was fortunate enough to snag three model 4B (2G) Pis.

This wasn't a quick process though; it took 2-3 months for the timing to line up right.
It's also worth mentioning that nearly all of these vendor websites had a policy of
one Pi per customer, which made things more difficult. I even saw various vendors
adopt very strict policies that stated only previously existing customers could purchase them, or that
they were no longer going to ship them internationally at all.

#### Cost

Although there were certainly some market elements that were out of my control
that contributed to the overall price (particularly shipping), there was opportunity
to cut costs in several areas.

**Power Supplies vs. PoE Switch**

In total, I spent $150 dollars on the PoE+ specific line items alone. Assuming
you already have an existing switch with 4 available ports, as well as 4 available
power outlets, that would translate to avoiding around $115 of cost, given you'd
need to purchase the power supplies instead (around $32 + shipping).

**Case**

As previously mentioned, I ended up sinking $85 into my case for the Pi cluster.
Sure it came with some frills, but it's arguable whether they justify the extra bucks.
It would definitely be possible to get away with a $15 dollar case like [this one](https://vilros.com/products/vilros-acrylic-4-layer-clear-case-box-enclosure-for-raspberry-pi?variant=29408411877470&currency=USD&utm_medium=product_sync&utm_source=google&utm_content=sag_organic&utm_campaign=sag_organic&gclid=EAIaIQobChMI9ZLJzbC7-QIV9AjnCh0WpAUbEAQYASABEgLszPD_BwE).

# What went well

#### Customer Service

##### International Shipping

Before this project, I really hadn't made an international purchase online before.
For the most part, it was made far easier thanks to rpilocater, as many of the
websites listing inventory were not written in english, and translating the content
was hit or miss. Luckily I was able to stumble my way through check out.

Waiting for the Pis to be shipped was far harder. In particular, one
of the Pis I bought stated that it had been shipped to the US, but when I looked
up the tracking number on the USPS website, it claimed it had never been received.
Calling the USPS, was less than helpful, but my email interactions with the staff
of the store I bought it from were really great - and most importantly, transparent.
They were knowledgable of the customs shipping process and the state of global
shipping, hopeful I'd get it, and remarked that they had just recently gotten
into international shipping. The positivity they had gave me that extra boost
of stamina for the long wait!

##### Standoff Snafu

Okay, so now I had the Pis - great! Next it was time to screw in these
bad boys and get them up and running.

When it came time to install these things into my newly purchased case, I had
bought some standoffs that were a compatible height to support the PoE+ hats I was using. I made
sure that they were M2.5 sized to match the screws provided with the [case](https://www.mouser.com/ProductDetail/RAF-Electronic-Hardware/M2104-2545-AL?qs=2411ckis64rpgA%252BYlrCb%252Bg%3D%3D)
itself; check!

Weirdly, on some of the tray mounts the screws wouldn't turn easily. I naively
thought it they needed an extra 'oomf' to get them to the desired position, so
of course I reached for my pliers!

> NOTE: Don't use pliers to install standoffs into the case.

Without too much additional manpower, I managed to snap off the threaded portion
of a standoff in the tray mount point. Game over! Sure, I could've just not
screwed in a standoff for that one corner, but then it just wouldn't be complete!
Also, since the GPIO pins are used to connect the hat and the Pi, I didn't
want to have to worry about accidentally squishing that limp corner too hard when
working with it in the future. I had to fix this.

I reached out to Uctronics support via their website directly. I explained to
them what happened, and they were incredible. They shipped not one, but two replacement
trays all for something that was completely my fault.

Best of all, thanks to Jeff Geerling and [his prior research](https://github.com/geerlingguy/raspberry-pi-dramble/issues/135), I went ahead and ordered another
set of standoffs with the hope that I'd get it right the second time - it worked like a charm!

# Was the cost worth it?

So was the cost worth it? With so many different Infrastructure as a Service (IaaS)
companies such as Linode, Amazon Web Service (AWS), etc. this is a question
that gets asked more an more.

To get a sense of this, let's consider what the cost could be if I were to choose
a provider such as Linode to host my cluster instead.

> NOTE: This is a rough comparison, as the exact specs of my build are not
a purchasable configuration.

#### Contender #1: Pi Cluster

To kick this off, let's get an idea of the (estimated) all in costs for the Pi cluster.

We've already highlighted the hardware costs above, but what about electricity costs?
Since all of the Pis are being powered by a single PoE+ switch, we can compute the
electricity costs by determining the power usage of the switch and how long we plan
to have it running.

With all ports utilized (which they are), the switch can draw up to 60W of power.
I plan on keeping this cluster up and running all day, everyday. Lastly, we'll
need to factor in average electricity rates for my state.

With the help of [this calculator](https://www.energy.gov/energysaver/maps/appliance-energy-calculator), it appears that all adds up to ~$131 annually.
Using simple maths, I came up with the following breakdown (rounding up the hardware costs to $550):

| Configuration     | Up Front Cost  | Cost per Month | After 6 Months | After 12 Months | After 18 Months |
|-------------------|----------------|----------------|----------------|-----------------|-----------------|
| Pi Cluster        | ~$550          | $11            | $616           | $682            | $748            |

#### Contender #2: Linode

First, let's take a look at some [Linode shared hosting](https://www.linode.com/products/shared/) plans:

| Plan          | CPU Core Count | Cost per Month |
|---------------|----------------|----------------|
| Linode 2GB    | 1              | $10            |
| Linode 4GB    | 2              | $20            |
| Linode 8GB    | 4              | $40            |

My cluster has 16 CPU cores (each Pi has 4) and a total of 7GB of RAM. To attempt a fair comparison,
I'll break down the comparisons into two sections: matching RAM and matching CPU.

**Matching RAM**

The following configurations achieve 8GB RAM.

| Configuration     | Cost per Month | After 6 Months | After 12 Months | After 18 Months |
|-------------------|----------------|----------------|-----------------|-----------------|
| 4 x Linode 2GB    | $40            | $240           | $480            | $720            |
| 2 x Linode 4GB    | $40            | $240           | $480            | $720            |
| 1 x Linode 8GB    | $40            | $240           | $480            | $720            |

With this approach, my Pi cluster would prove to be more cost effective after **~19 months**.

**Matching CPU Core Count**

The following configurations achieve 16 CPU cores.

| Configuration     | Cost per Month | After 6 Months | After 12 Months | After 18 Months |
|-------------------|----------------|----------------|-----------------|-----------------|
| 16 x Linode 2GB   | $160           | $960           | $1920           | $2880           |
| 8 x Linode 4GB    | $160           | $960           | $1920           | $2880           |
| 4 x Linode 8GB    | $160           | $960           | $1920           | $2880           |

With this approach, my Pi cluster would prove to be more cost effective after **~3.75 months**.

#### Conclusion

Yes, at least for me it was worth it, but only because I plan on hosting this
cluster in perpituity, want to have the extra control over my data, and want the
performance of dedicated hosting. The experience of learning how to build the
cluster from scratch is also an added educational bonus.

To be honest though, this experience showed me that it's probably better to at
least start out by using an IaaS platform. Depending on the configuration, you
could achieve a comparable setup that is actually cheaper for 12 or even 18 months.

I hope you enjoyed following along! Have any ideas for what I should use my
Pi cluster for? Toss me a comment and maybe I'll give it a try. Also, I can't
wait to bring you all along for what I might do with this next - there is so
much out there!
