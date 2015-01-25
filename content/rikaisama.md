Title: Real-time Anki import from your browser with Rikaisama on Arch Linux
Date: 2015-01-21
Category: PSA
Slug: anki-rikaisama-arch
Tags: Arch Linux, Python 3, Japanese, Firefox
Author: Roger Braun
Summary: Rikaisama's Anki integration does not work on Arch Linux, because it expects Python 2, whereas Arch Linux uses Python 3. This can be solved by unpacking the extension, editing a script and repackaging.

One of the best tools for learning Japanese (or any language, really) is the spaced repetition program [Anki](https://en.wikipedia.org/wiki/Anki_%28software%29). It is available for a lot of platforms, it's free and open source (GPL3) and can synchronize your learning progress across devices.

Another great tool is the Firefox add-on [Rikaisama](http://rikaisama.sourceforge.net/). Based on the more popular [Rikaichan](https://addons.mozilla.org/en-US/firefox/addon/rikaichan/), it's a dictionary for looking up Japanese words directly in your browser. Rikaisama adds a lot more advanced functionality, like using EPWING dictionaries, querying Japanese web dictionaries and the feature I'm going to talk about here, adding words to your Anki deck automatically, complete with reading and translation.

If you do a lot of reading on the web, this can nearly completely eliminate the tedious process of creating an Anki deck. I find that, in my experience, it also encourages you to add more words to your deck than you would have without it, because it's just a button press and not several copy-and-pastes.

On the technical side, this works by installing an [Anki plugin](https://ankiweb.net/shared/info/2512410601) that makes Anki listen on a local UDP port for new cards. Rikaisama in turn starts a Python script that writes the cards to this UDP port when called.

And here's where the problem begins for Arch Linux users.  Most distribution ship with Python 2 as a standard, but Arch Linux uses the slightly incompatible Python 3. You can install [Python 2](https://wiki.archlinux.org/index.php/Python) on Arch Linux, but the script inside of Rikaisama will still refer to just 'python', which is Python 3 on Arch.
It's easy to modify Rikaisama to use the right version, though. Rikaisama is provided as an .xpi file, which is a packed Firefox add-on. You can unpack the file (which is in fact a zip file) with 7zip:

    :::shell
    $ 7z x Rikaisama_v20.3.xpi

You quick grep will tell you which file to change:

    :::shell
    $ grep -r python
    udp/RealTimeImport_UDP_Client.py:#!/usr/bin/env python2
    udp/run_udp.sh:python "$1" $2 "$3"

As you see, udp/run_udp.sh doesn't use the correct python version. Change the line to read

    :::shell
    python2 "$1" $2 "$3"
and you're done. Now you can re-pack the file for use in Firefox. In the folder with the source files, pack the files with 7zip:

    :::shell
    $ 7z a ../Rikaisama.fixed.xpi * -r

You can now use Rikaisama.fixed.xpi to install the add-on, it will use the correct python version and you can finally import directly into Anki.


You can find a [version of Rikaisama](https://github.com/rogerbraun/rikaisama/raw/master/Rikaisama_v20.3_arch.xpi) containing this change on [GitHub](http://github.com/rogerbraun/rikaisama).

