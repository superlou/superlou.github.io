+++
title = "Jumper T8SG V2.0 Plus Review"
date = 2019-04-27

[taxonomies]
tags = ["quadcopters"]
+++

The Jumper T8SG V2.0 Plus is a multi-protocol remotes with Hall-effect gimbals commonly available near the $100 (USD) price point. It supports the Bayang protocol without any additional modules, which is perfect for the E011C I’ve been flying. At this price, it is a very interesting option for those just getting into flying but tired of toy remotes.

<!-- more -->

I ordered the [Jumper T8SG V2.0 Plus from Banggood](https://www.banggood.com/Jumper-T8SG-V2_0-Plus-Hall-Gimbal-Multi-protocol-Advanced-2_7-OLED-Transmitter-for-Flysky-Frsky-p-1257102.html?ID=42482&cur_warehouse=USA&p=9P091027571160201812&custlinkid=249027) for $108.11, including shipping insurance, and received it in under two weeks. I ordered the Mode 2 (left-hand throttle) variant in yellow. There is also a [“carbon fiber” finished V2.0 Plus](https://us.banggood.com/Jumper-T8SG-V2_0-Plus-Carbon-Special-Edition-Hall-Gimbal-Multi-protocol-Advanced-Transmitter-for-Flysky-Frsky-p-1442802.html?p=9P091027571160201812&custlinkid=249029) for a little bit more. Be careful that you are looking at the proper version, as there are non-V2.0, non-Plus, and “Lite” model also floating around.

If you are looking at a next step remote control, I didn’t see much below the $100 threshold that had great reviews. Around $40, Jumper makes a [T8SG Lite](https://www.banggood.com/Jumper-T8SG-Lite-Multi-Protocol-12CH-S-FHSS-DeviationTX-Compact-Full-Range-Radio-Transmitter-p-1338229.html?akmClientCountry=America&cur_warehouse=CN&p=9P091027571160201812&custlinkid=257875), but it is much more bare bones. It has a toy gimbals, LCD instead of OLED display, fewer switches, no DIY box, smaller antenna, and a much more limited number of protocols. The [FlySky FS-i6](https://www.banggood.com/FlySky-FS-i6-2_4G-6CH-AFHDS-RC-Transmitter-With-FS-iA6B-Receiver-p-983537.html?rmmds=buy&ID=42482&cur_warehouse=CN&p=9P091027571160201812&custlinkid=257876) (a.k.a Eachine i6, Turnigy TGY-i6) is also at a similar price as the Lite but has much more restricted radios, and I don’t know what my next drone might be.

I’ve put together a [2-minute unboxing and pairing video](https://www.youtube.com/watch?v=5hl1SUkehK8) demonstrating how easy it is to get flying with this remote and the budget drone I built earlier.

# What's Inside

![Remove box](jumper_box_small.jpg)

I received the remote in just under two weeks, and I was a little nervous that the minimalist packaging didn’t identify the specific T8SG model. However, a barcoded sticker on the side did identify the exact model, and the contained remote was marked with the proper silkscreens.

A very basic quick start guide is included. The most useful information there is two links:

* [http://www.jumper.xyz](http://www.jumper.xyz) for a software configuration guide
* [https://www.deviationtx.com](https://www.deviationtx.com) for details on the supported protocols.

The remote itself is shipped inside a snug “carbon fiber” carrying case.

![Remote and case](jumper_case_small.jpg)

The included carrying case has a feel between nylon and styrofoam. While a bit flexible, it is stiff enough to protect the remote against light pressures, like when pressed in a backpack against other gear.

![Remote accessories](jumper_accessories_small.jpg)

Some accessories are packed in with the remote:

* Jumper branded neck strap
* USB Type A to mini USB cable
* Alternate battery pack for running on 3.7 V batteries (18650 batteries)
* Plastic centering lever and spring to optionally configure the throttle for self-centering

From what I’ve read, the alternate battery pack and centering parts might be recent additions to the packaging of the yellow V2 Plus. The carbon fiber variant lists these items as part of the product. The remote itself was packed with foam spacers and wrapping, as well as protective film over the display.

# First Impressions

I find holding the remote with a pinching or thumb grip very comfortable, and it is less bulky than older RC gear that I have used. I don’t have large hands, and be aware that it is smaller than some of its peers, like the Taranis Q X7. The Jumper also does not have rubber grips on the back that I’ve seen on pictures of the Taranis.

![Remote front](jumper_remote_small.jpg)

After installing 4 AA batteries, the remote powers up with a very readable OLED display. The yellow-on-black contrast has been night and day compared to LCD display remotes I’ve used. Because I mostly use the remote indoors, I haven’t tried it under peak daylight conditions. It has been easily readable, though, when I flew outside.

![DIY Box](jumper_diy_box_small.jpg)

There are a few rattles on this budget remote. The navigation buttons can wiggle in the chassis, though only if you’re looking for problems. The “DIY Box” on the back of the remote seats loosely, but a thin piece of self-stick weather stripping fixes the knocking sound it makes.

![DIY Box fix](jumper_diy_box_fix_small.jpg)

The switches and buttons actuate crisply, and the gimbals feel very smooth. The throttle travel slides a little more loosely than other remotes, but not so loose as to be an issue.

![Remote gimbal](jumper_gimbal_small.jpg)

On the bottom left of the remote, the “enter” button is above the “back” button, and significantly smaller, which took some getting used to. The navigation menus are easy to decipher.

# Pairing

The remote integrates four RF modules: CC2500, CYRF6936, A7105, NRF2401. The loaded build of Deviation firmware (t8sg_v2_plus-v5.0.0-e801244) supports an impressive number of protocols: AFHDS-2A, ASSAN, Bayang, BlueFly, Bugs3, Bugs3Mini, CFlie, CG023, Corona, CRSF, CX10, DEVO, DM002, DSM2, DSMX, E012, E015, ESky, ESky150, Flysky, FQ777, Frsky, Frsky-V8, FrskyX, FY326, GW008, H377, H8-3D, HiSky, Hitec, HM830, HonTai, Hubsan4, iNAV, J6PRO, Joysway, KN, MJXq, MT99XX, PPM, Q303, S-FHSS, SBUS, Skyartec, SLT, SymaX, USBHID, V202, WFLY, WK2401, WK2601, WK2801, and YD717.

Pairing with my modified E011C was painless.

1. Enter the “Model menu”
2. Enter “Model setup”
3. Select a new “file”
4. Change the model name to “E011C,” or whatever you prefer
5. Change the model type to “Multi”
6. Select “Bayang” from the protocol list
7. Click “Re-Init” once the drone is powered up.

The remote also attempts on power-up to bind with the last model selected, which is very convenient since I only have one quadcopter right now.

# Conclusions

Flying with the T8SG V2 Plus instead of the toy controller that comes with the E011C was a bigger difference than I expected. I have much better control authority, especially in the vertical axis. I’m by no means a good pilot, but some of the maneuvers I’ve seen online feel achievable with practice, rather than magic. I also can go back to a “pinch” grip which is what I’m used to from other RC aircraft.

Since the remote is running Deviation firmware, there is a full package of custom mixing settings that I have yet to go through. This allows an incredible amount of configuration, even without using the USB connection. I set a flight timer based on the position of toggle switch G, and positioned it on the screen, while decluttering some items from the screen I didn’t need.

In the end, the T8SG V2 Plus is about future proofing my remote without breaking the bank. A full-sized remote is like learning to fly the quadcopter all over again, but in a good way.
