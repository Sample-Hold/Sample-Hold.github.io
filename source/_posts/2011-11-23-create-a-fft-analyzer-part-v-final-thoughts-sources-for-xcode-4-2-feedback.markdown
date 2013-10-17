---
author: fredguile
comments: true
date: 2011-11-23 15:34:55+00:00
layout: post
slug: create-a-fft-analyzer-part-v-final-thoughts-sources-for-xcode-4-2-feedback
title: 'Create a FFT Analyzer part V: final thoughts, sources for XCode 4.2 & feedback'
wordpress_id: 38
categories:
- VST Plugins / Audio Units
tags:
- Audio
- Mac OS X
- XCode
---

[![](http://guileboard.files.wordpress.com/2011/11/au.gif)](http://guileboard.files.wordpress.com/2011/11/au.gif)This part wraps up our tutorial on building an audio effect as Audio Unit for OS X Lion. Though it is not the craziest plug-in you'll ever built, it made us learn some basis about DSP programming, as well it introduced the XCode environment for developing Audio Units. Of course, there are numerous improvements we could do on this project. In this article, I'll make some remarks about my work and  also a few issues I met during development. Last but not least, we recall the GitHub repository URL for you to grab the code and make our own version.




<!-- more -->





### Remarks


After thinking twice about it, here are the things I finally dislike :



	
  * **Mixing C, C++ and Objective-C in a (small) project is a bad idea**: it definitely make your developer duties harder. I would personally prefer to stick to C++ for all tasks. Using Cocoa UI was a good way to challenge the native SDK of Mac, but I may prefer using another SDK that avoid sprinkling our code, and thus, knowledge.

	
  * **Not all features of Cocoa are supported by AU/VST hosts**: for instance, I couldn't resize my spectrum analyzer view in Ableton Live and Garage Band, and I couldn't display the NSMenu as context menu when right-clicking on the Cocoa window.

	
  * **NSBezierPath is slow**: You might have noticed how slow it is when choosing 16384 as your FFT block size. We could  probably have had better results with a Quartz or OpenGL rendering. To be perfectly honest, at first, I tried to elaborate an OpenGL version of this tutorial, but I ran into an graphical bug with Ableton Live: it seemed that my plug-in wasn't properly releasing the OpenGL context, thus corrupting the entire Live UI. I could not get out of this, so I decided to revert to a more "portable" solution for drawing the spectrum graph (though it's exaggerated  writing that Cocoa is "portable" ;)).

	
  * **The overall result isn't as precise as we wanted**: our graph is not big enough to give you valuable informations about dB amplitudes (especially if the host doesn't allow our plug-in to be resized!). We should have added some precious features like zoom, prevision, hold peaks, etc. (this is out of scope of this tutorial). And there might be more precise methods rather than simply adding a constant dB correction into our _SimpleSpectrumProcessor_ class.

	
  * **We should have proposed a solution for developing both VST au AU versions of our Spectrum Analyzer**: but, as a drawback, it would make our tutorial much more difficult. Anyway, I'll probably publish another tutorial for this in the future.




### Give feedback and share this code


Feedback is welcome! You may leave a comment at the bottom of this article.

In the meantime, you can contribute to this tutorial by registering yourself on GitHub and cloning my repository : [https://github.com/Sample-Hold/SimpleSpectrumAnalyzer](https://github.com/Sample-Hold/SimpleSpectrumAnalyzer)


### Final words




I spent a couple of minutes hacking my code in order to show you in a video how I had corrupted the Ableton Live UI when I was using a NSOpenGLView instead of a NSView. Unfortunately, I'm completely unable to reproduce this bug! That'll teach me not to version every compiled binary as nightly builds ;)




Before I throw out this hack, I can show you what it is like this OpenGL version of the Simple Spectrum Analyzer:

{% youtube 8nnl2BFCZQ8 %}

As a conclusion, we can see that the OpenGL alternative still renders slowly with an FFT size of 16384 samples. So there must be a lot of performance tweaks to perform elsewhere. For anybody interested, the alternate code has been pushed to GitHub.
