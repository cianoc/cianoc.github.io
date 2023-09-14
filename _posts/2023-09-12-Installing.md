# Installing Common Music

Common Music has been around a while, and there are multiple versions of it. This can make it difficult to work out how to get started with it. In this post I'm going to show you what you
need to get a modern version up and running, which will do realtime scheduling, works with MIDI, OSC (e.g. SuperCollider), will generate sheet music and can even be used for synthesis.

The instructions in this post should work for OSX just fine. On Linux you will need to tweak it a little, but it should be fairly obvious.
Common Music will work on Windows, but I'm afraid I've not used Windows in ten years so I'm not sure what you need to do. Hopefully this post will provide some pointers, and if you
work it out then I'd be happy to update this post with your instructions.

## Background
This guide assumes you have some comfort with the command line, using the terminal and have done at least some programming before.

If you're using a Mac, then make sure you have [brew](https://brew.sh/) installed. The instructions are fairly simple, and there are Youtube videos that will walk you through it. Brew is a package
manager that will install and update software for you. If you're on Linux then you'll obviously use your ditro's package manager.

Then install [SBCL](https://www.sbcl.org/) for your platform.

On OSX this will be:
```bash
brew install sbcl
```

For more information on using LISP I recommend the [https://lispcookbook.github.io/](Lisp Cookbook). In particular the information on using an editor will be extremely important,
as a good editor is crucial to using LISP. There are numerous introductions to Common Lisp, most of them are out of print (though you can usually find copies floating around online).
I recommend either of the following, both of which can be legally read online:
- [Common Lisp - A Gentle Introduction to Symbolic Computation](https://www.cs.cmu.edu/~dst/LispBook/book.pdf)
- [Practical Common Lisp](https://gigamonkeys.com/book/)

If you want something more focused on Common Music, then both of these will get you started:
- [Notes from the MetaLevel](https://www.amazon.com/Notes-Metalevel-Introduction-Computer-Composition/dp/9026519575/)
- [Algorithmic Composition: A Gentle Introduction to Music Composition Using Common LISP and Common Music](https://quod.lib.umich.edu/s/spobooks/bbv9810.0001.001/1:4/--algorithmic-composition-a-gentle-introduction-to-music?rgn=div1;view=fulltext)

You can find online copies of Notes from the MetaLevel if you look for it. All of the other books are legally available online.

## Installing Supporting Libraries
Assuming you've got Common Lisp up and running, we'll now look at getting the supporting libraries setup for Common Music.

    In order to install Incudine you will need:
- JACK or PortAudio >= 1.9
- PortMidi
- libsndfile >= 1.0.19
- FFTW >= 3.0
- GNU Scientific Library (GSL)
- [Optional] FluidSynth
- [Optional] LILV

On Linux you should be able to install these easily with your package manager, and its also relatively trivial on OSX.

On Linux you will have to make a decision as to whether you want to use JACK (I would recommend this, but it is more work), or PortAudio (far easier to work with). If you're not sure which
you have running then its probably portaudio. Go with that - as you can always use Jack later (consult the [incudine](https://incudine.sourceforge.net/) documentation for more details)

On Windows and OSX you will need to use PortAudio.

For OSX type:
```bash
brew install portaudio portmidi libsndfile fftw gsl fluid-synth lilv
```


## Installing Quicklisp
Next you need to install a package manager for LISP. Fortunately this is very easy, just follow the instructions [https://www.quicklisp.org/beta/#installation](here).
If you're running OSX then the instructions on this page tell you exactly what to do (they should also work for Linux).


## Installing Incudine
Open your terminal. Then type the following on Linux/OSX (on Windows this will likely be a little bit different):

```bash
cd .quicklisp/local
```

```bash
git clone git://github.com/titola/incudine.git
```

Then open up the REPL for SBCL, preferably in your editor. Once this has loaded, type:

```common-lisp
(ql:quickload :incudine)
```

This might take a few minutes, but hopefully at the end of it you should have a success message. If you run into problems let me know and I'll try to help you (opening an issue on this blog will work).

Incudine is a very cool synthesis package in its own right, and I recommend checking out the documentation.
In a future post I will discuss how you can use common music to make music in Incudine.

## Installing Common Music
Return to the command line, make sure you're in the same directory as before and type the following:

```bash
git clone https://github.com/ormf/cm.git
git clone https://github.com/ormf/cm-incudine
git clone https://github.com/ormf/fudi-incudine
git clone https://github.com/ormf/fomus
git clone https://github.com/ormf/cm-fomus
```

Now return to your REPL and type:

```common-lisp
(ql:quickload :cm-incudine)
(ql:quickload :cm-fomus)
```

Assuming that there were no errors, you should be ready to go. Congratulations!
