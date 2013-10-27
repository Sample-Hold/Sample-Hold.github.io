---
author: fredguile
comments: true
date: 2011-12-19 04:00:26+00:00
layout: post
slug: make-fun-of-your-launchpad-with-launchplay-vst-plugin
title: Make fun of your Launchpad with LaunchPlay VST plugin
wordpress_id: 255
categories:
- VST Plugins / Audio Units
tags:
- Audio
- Launchpad
- Mac
- MIDI
- PC
- VST Plugin
---

{% img alignleft /images/blog/launchplay-image.png %} I bought a Launchpad controller from Novation a few years ago, and although it's a great midi interface offering a perfect native remote for Ableton Live sequencer, I couldn't help thinking that this amazing tool could certainly be used in other unexpected ways, thus controlling alternate gears or software. So on the same time, I was both enjoying my Launchpad and drooling to what would have offered at more experimental controller such as [monome device](http://monome.org/devices)...




Luckily, Novation made a programming guide available for the Lauchpad, as well as Ableton proposed an extended version of Live integrating Max/MSP (that is called "Max for Live") that would help me satisfy my nerd-est desires. So I started to draft a layout for simple jamming. But I wanted more : my tool would work on any sequencer, with both Mac and Windows platforms, all of this requiring no additional license.




Evidently, I got a bunch of new ideas when I started writing for sample-hold.com: I was now dreaming about a VST plugin that would act as a MIDI effect, allowing pure jams with a touch of randomness. I think I've come up with a preliminary version called "LaunchPlay VST". Let's look at it and explain how to use this strange plugin... <!-- more -->





### Disclaimer




First of all, the jamming idea of LaunchPlay VST is not really new and I would like to thanks Batuhan Bozkurt for his genius work on Otomata ([his website](http://www.earslap.com/)), an online generative music instrument, because this is from where my work starts. While Batuhan made a mobile application of his work, on my side I have always considered that this would not suit the needs of some musicians and producers that were buying a lot a gears or synthetizers, and just wanted... to use them, leaving their cellphone in their pocket. That could be the benefit of a MIDI plugin.




So, LaunchPlay is a** VST plugin for Mac and Windows**, containing three sub plugins :




- LaunchPlay VSTi: the main sequencer that you will insert into a new "instrument" track,




- LaunchPlayVirtualCable VST: a great companion that will help you overcome the MIDI routing issue we describe below,




- LaunchPlay MidiFilter VST: another companion that can help you achieve the MIDI routing.





### LaunchPlay layout


Here is the very simple layout available for use in LaunchPlay:

{% img centered /images/blog/launchplay-layout.png %}


You may notice that the rightmost buttons of the Launchpad have been turned into a MIDI channel selector : this will allow us to send generative music to up to eight gears (virtual instruments or real synthetizers), bringing a nice multi-timbral feature to our generative music experience!




The grid is where you play notes in a quite similar way than Otomata (well, not exactly). And we can reuse the top buttons for some new features not included in the genuine LaunchPad:






	
  * up, down, left, right: this is where you change the direction before creating a new "worker" on the grid,

	
  * delete one: this is a switch button. When it's red, you're on the "remove" mode and you should be able to individually remove workers,

	
  * delete all: this is an instant button, used to remove all workers on the current grid,

	
  * ticks: they are pretty useless visual markers indicating that your sequencer is running.




Before I explain the required MIDI routing, let's watch a demo video:




{% youtube JWVFdrywEqc %}




### MIDI routing




For proper work, LaunchPlay needs you to setup this MIDI routing in your VST Host:






	
  * LaunchPlay VSTi must receive notes from LaunchPad

	
  * It will send MIDI events from channel 1 to channel 9:

	
    * channel 1 is dedicated for feedback events. They must be routed to the LaunchPad, using either my MidiFilter or VirtualCable plugins.

	
    * channels 2-9 are receiving notes coming from the eight different layers of the LaunchPlay Sequencer





{% img centered /images/blog/routing.png %}



You can use:



	
  * **LaunchPlayMidiFilter**, which aims to filter received MIDI events in order to "pass-thru" events that correspond to a given channel,

	
  * **LaunchPlayVirtualCable**, which allow to bind channels from the LaunchPlaySequencer to any MIDI track of your VST Host (very handy, for instance, in Ableton Live, which doesn't allow to routing many MIDI channels between VST plugins)

	
  * or your VST host native routing features, if any.


I can show you my typical routing done in Live:


{% youtube lJOVjKdoO9E %}




### Grab the source code & contribute




You can as usual grab everything for free on the [projects' GitHub page](https://github.com/Sample-Hold/LaunchPlayMIDIEffect).




Please note that everything is licensed under a [Creative Commons Attribution-NonCommercial-ShareAlike 3.0 Unported License](http://creativecommons.org/licenses/by-nc-sa/3.0/), so don't make any commercial use of this, share it, talk about it, in the name of fun, thanks!




Final thoughts




To wrap up this article, Launch Play was as fun to build as it's probably fun to use! Of course, I had a lot of issues to overcome. For instance, VST plugins are not really tailored to behave a MIDI effect, and I had to find out a few tricks to be able to correctly route MIDI messages  between the LaunchPlay sequencer and instruments. I discovered that there is no official API for designing MIDI effect, despite that Steinberg provides the [VST Module Architecture SDK](http://www.steinberg.net/en/company/developer.html), which is unfortunately not used by most of the sequencers.




**Edit 28/12/11**: some users have reported that the "Virtual Cable" mode can be very laggy on some configurations. I am working on an solution using another library for sending messages between channels. In the meantime, I suggest using pure MDI routing: for this, unfortunately, you must switch the last VST parameter from "Virtual" to "MIDI" and set up your own routing.




**Edit 17/01/12**: for those who asked, yeees, by the way, the first video I recorded is back online! One can enjoy  my sweety pony melody again:




{% youtube pdshwDY19s4 %}


I'm thinking about some new tutorials to write on sample-hold.com. I don't know yet, we could discuss about writing a new VST plugin from scratch, or we could study some programming tricks based on the LaunchPlay sources. If you have any precise idea on what you would like to see on this website, please leave a comment!
