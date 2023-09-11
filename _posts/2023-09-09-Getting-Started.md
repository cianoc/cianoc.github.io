# CommonMusic Reference
This post assumes that you have some experience with CommonMusic. 
If you don't, that's fine, I'll introduce in a later post how to install and run it.

## Starting Common Music Notation

```common-music
(ql:quickload :incudine)

(ql:quickload :cm-incudine) ;; this order is important, as this allows cm-incudine to check midi setup.

(in-package :cm) ;; we will be working within the cm package to do things.
                 ;; not great style these days, but it's old
```

This will setup start cm-incudine with the default MIDI input and output ports. To see what it is using if you're using portmidi:

```common-music
(pm-input-stream)

(pm-output-stream)
```

Currently it only allows you to use the standard input stream, but this will be changed soon...

*** Setting up OSC

```common-lisp
(osc-open-default :host "127.0.0.1" :port 3003 :direction :output)

(osc-open-default :host "127.0.0.1" :port 3004 :direction :input)

; and an example of using this.

(sprout
 (process
   repeat 2
   output (new osc :time (now) :types "iiii" :message '(1 2 3 4))
   wait 1))
```

## Note Names & Music Notation
### Notes
Note names are the symbols a-g (e.g. ```'c```)

These can be flattened sharpened by adding on to the end ```ff```, ```f``` ```n``` ```s``` ```sss```.
You can also use these on their own:

```common-lisp
(note 'c4 :accidental 's)
(keynum 'dff4)
```

Octaves are just numbers, with ```c4``` representing middle C. The lowest octave is ```-1```, the highest is ```10```.

### Rests
Rests are notated using ```r```

### Key Numbers in the Chromatic scale
Key numbers in the standard chromatic scale may be expressed as integers or a floating point numbers. 
A floating point key number is interpreted kkk.cc where kkk is the integer keynum and cc is cents above the Hertz value of kkk.

For example, 60.5 means 50 cents (quarter tone) above the frequency of
keynum 60, or 269.291 Hz. These are the same as the midinumber scale.

```common-lisp
(keynum 439 :hz)
(hertz 69)
```

### Rhythm
The basis for all time in Common Music are seconds. Standard rhythmic measures (beats, along with Common Music notation) 
can easily be converted to this (see conversion section below for function descriptions.

#### Notation for rhythms

- ```m``` maxima
- ```l``` longa
- ```b``` brevis
- ```w``` whole
- ```h``` half
- ```q``` quarter
- ```e``` eighth
- ```s``` sixteenth
- ```t``` thirty-second
- ```x``` sixty-fourth

These can be combined with division letters:

- ```t``` - triplet
- ```q``` - quintuplet

And in addition dots can be added on the end (with the standard music meaning)

- **quarter** ```q```
- **Dotted Quarter** - ```q.```
- **triplet quarter** - ```tq```

You can also add, subtract and even multiply divisions:

- **sixteenth plus triplet quarter** - s+tq
- **whole minus triplet sixteenth** - w-ts
- **whole times 4** - w*4

### Amplitude
All the standard symbols from ```ffff``` to ```pppp``` can be used. These will be converted to appropriate db levels.
See the dictionary for exact numbers if curious.

You can also work in values between ```0.0``` and ```1.0``` where these map to ```pppp``` and ```ffff``` respectively.

These values by default increase linearly, which is probably not what you want. You can adjust how these map using:
- ```*loudest*``` 
- ```*softest*```
- ```*power*```

loudest and softest do what you expect. Power adjusts the power curve. The default value is 1.0. which means that amplitude 
values increase linearly. Whether this matters depends upon what you're trying to do.

TODO - work out what the proper default should be to get appropriate db output.

## Conversion
CommonMusic has great functions for converting in pretty much anyway you can imagine (and plenty that you can't).

### hertz
```common-lisp
(defun hertz (freq &key (hz nil) (in *scale*) through))
```

Returns the hertz value of freq, which can be a notename (as a symbol), key number (midi number), Hertz value - 
or a list of any of the above.

```hertz``` supports the following keyword arguments:
- ```:hz boolean``` - If true then a numerical freq is interpreted as a Hertz value rather than as a key number.
- ```:in tuning``` - The tuning to return the Hertz value from. The default value is \*scale\*.
- ```:through { tuning | mode}``` Filters freq through tuning or mode and returns the Hertz value of the closest Hertz values.

### keynum
```common-lisp
(defun keynum (freq &key (hz nil) (from *scale*) in? to through))
```
Returns the key number of freq, which can be a note name, key number, Hertz value or list of the same. 

If freq is in Hertz then the return value will be a floating point keynum where the fractional portion times 
100 represents that may cents above the integer keynum.  For example, the floating point keynum 60.5 stands 
for the Hertz value 50 cents above Middle C, or 269.291 Hz.

keynum supports the following keyword arguments:

- ```:hz boolean``` - If true then a numerical freq is interpreted as a Hertz value rather than as a key number.
- ```:from { tuning | mode}``` - The tuning or mode to return the key number from. The default value is *scale*. If mode is specified then freq is a modal key number and the value returned is the equivalent key number in the mode's tuning. if tuning is specified then freq is an integer key number and the value returned is the corresponding, possibly floating point, key number in the standard chromatic scale.
- ```:in? { tuning | mode}``` - Tests if freq references a key number in tuning or mode. Returns the key number or false.
- ```:to mode``` - Forces freq to the closet key number in mode.
- ```:through { tuning | mode}``` - Filters freq through tuning or mode and returns the closest integer tuning keynum.

### note
```common-lisp
(defun note (freq &key hz in in? through))
```

Returns the note name of freq, which can be a note name, key number, Hertz value or list of the same.

note supports the following keyword arguments:

- ```:hz boolean``` - If true then a numerical freq is interpreted as a Hertz value rather than as a key number.
- ```:in { tuning | mode}``` - The tuning or mode to return the note from. The default value is *scale*. If a mode is specified then freq must be a modal key number.
- ```:in? { tuning | mode}``` - Tests if freq references a note name in the specified tuning or mode. Returns the note name or false.
- ```:through mode``` - Filters freq through mode and returns the closest note.

### transpose
```common-lisp
(defun transpose (freq interval &optional scale))
```

Returns the transposed value of freq in scale. freq may be a note name, key number or list of the same. 
Interval may be an integer, encoded interval or note.

transpose supports the following optional arguments:

- **scale** - The tuning in which the transposion occurs. The default value is *scale*

### rhythm
```common-lisp
(defun rhythm (rhythm &optional tempo beat)
```
Returns the value of rhythm converted to seconds. 

Rhythm can be a proportion (number), a rhythmic symbol or a list of the same. 
The optional tempo is a metronome speed, and defaults to *tempo*. The optional Beat parameter specifies the 
metronome's pulse and defaults to *beat*.

#### Examples
```common-lisp
(rhythm 'q)
⇒ 1.0
(rhythm 1/4 120.0)
⇒ .5
(rhythm 'w+q )
⇒ 5.0
(define *tempo* 120.0)

(rhythm '(q th e.. x) )
⇒ (0.5 0.6666666666666666 0.4375 0.03125)
```

### Amplitude
```common-lisp
(amplitude amp &optional softest loudest power)
```

Converts a logical amplitude amp to an amplitude value according to its optional arguments. 
A logical amplitude is a value from 0.0 to 1.0 inclusive, or one of the following symbols:

niente pppp ppp pp p mp mf f ff fff ffff

amplitude supports the following optional arguments:
- ```softest``` - The value to return when amp is 0 or niente. The default value is *softest*.
- ```loudest``` - The value to return when amp is 1 or ffff. The default value is *loudest*.
- ```power``` - If power is 1 then the amplitude returned is calculated linearly, otherwise the value lies on an exponential curve amppower. The default value is *power*.

#### Examples

```common-lisp
(amplitude 'mp)
⇒ .5
(amplitude 'mp 0 127)
⇒ 63.5
(amplitude .5 0 1 4)
⇒ 0.0625
```

## Outputting Music
Common Music uses objects for the different types of music output (e.g. midi, or music notation). In this section I describe the different types.

*Note:* Common Music uses objects for most types of musical event, and to make things easier it adds the macro `new` as this reduces boiler plate. There is of course no reason you need to use `new`, but it does reduce line noise.

### MIDI 
This section will just describe notes and CC messages. All other event types are possible (including stuff defined in the MIDI file standard) and you should consult the dictionary for these.

#### Midi Notes
Generally you create a MIDI note event with 
```common-lisp
(new midi &keys time channel keynum duration amplitude)
```

This is just a standard Common Lisp object, but CM added the convenience macr

This will generate the note on and off events. If for some reason you need to generate just one of those, then this also possible (see the ditionary for details).

Keys are as follows:
- ```:time number``` The start time of the object.
- ```:channel integer``` A MIDI channel number. The default value is 0.
- ```:keynum {keynum | note}``` A MIDI key number or note name in the standard chromatic scale. Floating point key numbers are rounded to their closest integer key number unless channel tuning is enabled in the destination midi stream. If the value is less than zero or the symbol r (rest) then the event will not be output to the destination.
- ```:duration number``` The duration of the note.
- ```:amplitude number``` A floating-point logical amplitude 0.0-1.0, or an integer velocity 0-127. If the value is zero then the event will not be output to the destination. The default value is 64.

#### Midi CC Events
```common-lisp
(new midi-control-change &keys time channel controller value)
```

Keys are as follows:
- ```:time number``` - The start time of the object.
- ```:channel integer``` - A MIDI channel number. The default value is 0.
- ```:controller integer``` - A MIDI controller value 0-127.
- ```:value integer``` - A MIDI control value 0-127.

#### Real-Time MIDI output
Real time MIDI output requires incudine to be installed and running. (see opening section for more details).

Once you have started `cm-incudine`, then `output` `event` and `sprout` can be used to send MIDI events to the real time MIDI scheduler:

```common-lisp
(output (new midi)) ;; you should hear a note on your synthesizer if this is setup correctly

(sprout
 (loop for x below 10
    collect (new midi :time (* x 0.5) :keynum (between 60 71))))

;; Note that here we had to use *rts-out* to tell it where to send these events as we're collecting them.
(events
 (loop for x below 10
    collect (new midi :time (* x 0.5) :keynum (between 60 71)))
 *rts-out*)

;; here we're using process, but we still have to use *rts-out* as we need to tell it what to do witht the output
(events
 (process
   repeat 5
   output (new midi :time (now) :keynum (between 60 71))
   wait 0.5)
 *rts-out*)

;; But here we use sprout, and we don't need to worry
(sprout
 (process
   repeat 5
   output (new midi :time (now) :keynum (between 60 71))
   wait 0.5))
```

#### Generating a MIDI file
To generate a MIDI file, use events and a filename with the extension `.mid`. You can make sure it goes to the right directory by using `pwd` to check the current directory, and by using `cd` to change directories:

```common-lisp
(cd "~/Music/Composition") ;; You may have to modify this on windows.

(events (new midi :time 0
:keynum 60 :duration 2) "temp.mid") ;; you can also use a full filename if you prefer
```

This should generate a midi file in the correct format. See the dictionary for the full array of possibilities.

You can also set a hook once this is generated using `(set-output-midi-hook!)`, though given real-time output will work just fine, not sure there's much benefit to this.

### Musical Notation
*Work in Progress*. This uses fomus. I have not tried this yet, but it requires using different event types. You can also do it using common music notation, but not currently clear to me if there is a working version of that around.

### OSC
*TODO*
```common-lisp
(sprout
 (process
   repeat 2
   output (new osc :time (now) :types "iiii" :message '(1 2 3 4))
   wait 1))
```

### Incudine
*TODO*
