+++
date = "2015-03-06 14:47:41"
draft = "false"
title = "Alfred, csshx and terminalception"
slug = "alfred-csshx-and-terminalception"

+++

### Alfred, csshx and terminalception

I use Tmux usually but Tmux on the mac has not been playing nice with csshx for me. Something in the dark magic of perl broke with an error that looks like this

> Mar  5 08:58:42 silvias-MacBookPro.local perl[10828] <Error>: ImageIO: CGImageDestinationFinalize image destination must have at least one image
2015-03-05 08:58:42.473 perl5.18[10828:2436454] CGImageDestinationFinalize failed for output type 'public.tiff'
**** ERROR **** PerlObjCBridge:: convertPerlToObjC(): Referenced thingy not blessed
**** ERROR **** PerlObjCBridge:: convertArg() for index 2: convertPerlToObjC() failed
**** ERROR **** PerlObjCBridge:: sendObjcMessage: Error converting argument 1 for message "setObject:forKey:"
**** ERROR **** PerlObjCBridge: error [1] sending message [__NSDictionaryM setObject:forKey:] at /System/Library/Perl/Extras/5.18/darwin-thread-multi-2level/PerlObjCBridge.pm line 248.

But I wasn't about to let that take away my terminalception magic :) 

Note: This is for a mac environment but you may emulate it in your favourite distro by replacing Alfred with whatever launcher you may use.

#### What you need
Alfred app is my favorite launcher in mac but I suspect Quicksilver (if you are the quaint type) can also run commands directly to the terminal. 

Next install Homebrew. You need this to install csshx easily. Also because GNU tools :)
`ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

Install csshx

`brew install csshx`

If you are chef shop like mine, a lot of times what you are looking for is to ssh to all the machines of a certain type. In chef, we call those roles. This step will change depending on the configuration management/service discovery framework of choice in your infrastructure. 

First, set the terminal app your alfred will use. I set it to Terminal because I wanted this to be separate from my usual workspace in iTerm 2

![Alfred settings](https://farm1.staticflickr.com/735/22396914786_e46c0ed4cd_b.jpg)

Then, when I want to fire up csshX to a bunch of our our servers all at once, it usually looks like this. 

_Protip_: knife ssh can take cssh as an argument, so no awk and bash pipes required.

![alfred term](https://farm6.staticflickr.com/5793/22433815321_7b1f1fced2_b.jpg)

This fires up Terminal, which runs the chef search, and opens separate windows. 

![csshx cap](https://farm6.staticflickr.com/5805/22422959375_891c2f313a_k.jpg)

Your cursor will be in the bottom red window by default and input will appear in all the windows at the same time. When done, CMD+Q will bring this window

![quit window](https://farm1.staticflickr.com/744/22422971645_735d9473b9_b.jpg)

And done :)