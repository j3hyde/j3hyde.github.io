---
layout: post
title: "Building pyPortMidi"
tags: howto
date: 2015-01-22 12:00:00
---
I recently decided to look into communicating with MIDI devices from
Python on Windows.  As a side interest I have two Korg EA-1 synths and
one ER-1 mkII.  The impetus for looking into
[pyPortMidi][http://alumni.media.mit.edu/~harrison/pyportmidi.html] was
just to setup some simple, repeating patterns to play the EA-1 while I
tweaked and tuned the oscillators and filters manually.

Out of the various Python packages available for MIDI, I found that most
of them focused on parsing and playing the MIDI file format.  What I
needed was something to send/receive MIDI messages to/from Windows MIDI
devices.  In the end pyPortMidi seemed to be the best candidate but the
only Windows binary packages available were for Python 2.2 and 2.4
whereas I'd need a package for Python 2.7.  Thus I set out to build
Windows binary packages for Python 2.7.

Pre-requisites
Steps
Download
