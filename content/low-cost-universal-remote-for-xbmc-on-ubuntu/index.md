+++
title = "Low Cost Universal Remote for XBMC on Ubuntu"
date = 2013-04-21
[taxonomies]
tags = ["linux", "xbmc"]
+++

I currently have an [RCA RCRP05BR 5 Device Cable Replacement Universal Remote (Black)](http://www.amazon.com/gp/product/B0028IKXLS/ref=as_li_qf_sp_asin_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=B0028IKXLS&linkCode=as2&tag=louissimonsco-20), and it works well for controlling an LG TV, Roku XD, and original XBox.  My home theater PC (picked up by a friend when Borders went out of business) did not have a built in infrared receiver, so the objective was to find a receiver compatible with the universal remote that would be nicely detected by Ubuntu.

<!-- more -->

The uncreatively named [Wireless USB PC Remote Control Mouse for PC](http://www.amazon.com/gp/product/B001M56DI0/ref=as_li_qf_sp_asin_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=B001M56DI0&linkCode=as2&tag=louissimonsco-20) was cheap enough that I wouldn’t  be too unhappy if I couldn’t get it to work in Ubuntu. With the receiver plugged into the PC, lsusb indicates that it is a “Chaplet Systems, Inc. infrared dongle for remote.”  It even has a boot interface subclass 1, which means you might be able to use it like a keyboard from inside the BIOS.  Playing around with the remote, you can use it almost like a full keyboard and mouse, if you are patient enough to type via [cellphone-multi-tap-esque](http://en.wikipedia.org/wiki/Multi-tap) entry.

# Assigning Functions

The media control keys seemed to work automatically with XBMC, and the universal remote had no difficulty learning them.

The action of the 5×3 grid of keys is handled on the receiver side.  Pressing the up-arrow/2abc key always sends the same IR signal, but depending on the mode of the current mod of the receiver (toggled by the Numlock key), determines the command seen by the computer.  This means that you need to pick which mode you will use with XBMC, as there’s no great way to handle mode switching on the universal remote.  I opted to set the receiver in “white silkscreen mode.”  This gives you access to the arrow keys, enter key, tab, backspace, and escape, which already have context in XBMC.  The function assignments I came up with are in the following table:

> Note: looks like an upgrade lost my table. :(

[table id=1 /]

I’m a big fan of using the [default controls](http://wiki.xbmc.org/index.php?title=Keyboard#Common_default_keyboard_controls) whenever possible, but I ran out of white silkscreen commands on the PC Remote. I taught the E-mail button of the PC Remote to the Menu key of the universal remote.  Then, I mapped the launch_mail “key” in XBMC by creating a keymap file in the XBMC configuration directory.  On a standard Ubuntu/Mint install, it is `~/.xbmc/userdata/keymaps/`, so I created a new file called pc-remote.xml in that directory with the following contents:

```
<keymap>
  <global>
    <keyboard>
      <launch_mail>ContextMenu</launch_mail>
    </keymap>
  </global>
</keyboard>
```

The full list of keys available for XBMC mapping is available on the XBMC wiki, including the names of [media remote keys](http://wiki.xbmc.org/index.php?title=Keymap#Media_keyboards.2Fremotes) like launch_mail.

# RCRP05BR Key Learning Procedure

For convenience, here’s the procedure for learning a key on the universal remote:

1. Press and hold SETUP until the last-selected mode key blinks twice, then press 9 7 5.
2. Press a mode key once (TV, DVR, CBL, DVD, AUD) to assign a mode for learning.
3. Press the desired key on the RCRP05B once to put the remote in listening mode for the pressed key.
4. Point the teaching remote at the learning remote about 2″ apart.
5. Press (hold if necessary) the button on the teaching remote to be taught.
6. Repeat steps 2 through 5 for each key you want to learn.  Press and hold SETUP until the active mode key blinks twice to exit programming.

# Using Global Volume Lock

It looks like the instructions for assigning the volume controls of one device to multiple devices are incorrect in the RCRP05BR’s manual.  The correct instructions are at [hifi-remote.com’s wiki](http://www.hifi-remote.com/wiki/index.php?title=VPT_Style_2), and follow below:

1. Press the the mode key of the volume controls we want to assign to other modes.
2. Press and hold SETUP until the mode key blinks twice, then press 9 3 3.
3. Press each mode where you want the audio buttons (volume, mute) to follow the mode from step 1.
4. Press SETUP.  The mode from step 1 should blink twice.
