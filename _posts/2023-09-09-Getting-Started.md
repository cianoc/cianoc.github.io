# CommonMusic Reference
This post assumes that you have some experience with CommonMusic. 
If you don't, that's fine, I'll introduce in a later post how to install and run it.

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
- ```\*loudest\*``` 
- ```\*softest\*```
- ```\*power\*```

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
- **:hz boolean** - If true then a numerical freq is interpreted as a Hertz value rather than as a key number.
- **:in tuning** - The tuning to return the Hertz value from. The default value is \*scale\*.
- **:through { tuning | mode}** Filters freq through tuning or mode and returns the Hertz value of the closest Hertz values.

### keynum
```common-lisp
(defun keynum (freq &key (hz nil) (from *scale*) in? to through))
```
Returns the key number of freq, which can be a note name, key number, Hertz value or list of the same. 

If freq is in Hertz then the return value will be a floating point keynum where the fractional portion times 
100 represents that may cents above the integer keynum.  For example, the floating point keynum 60.5 stands 
for the Hertz value 50 cents above Middle C, or 269.291 Hz.

keynum supports the following keyword arguments:

- **:hz boolean** - If true then a numerical freq is interpreted as a Hertz value rather than as a key number.
- **:from { tuning | mode}** - The tuning or mode to return the key number from. The default value is *scale*. If mode is specified then freq is a modal key number and the value returned is the equivalent key number in the mode's tuning. if tuning is specified then freq is an integer key number and the value returned is the corresponding, possibly floating point, key number in the standard chromatic scale.
- **:in? { tuning | mode}** - Tests if freq references a key number in tuning or mode. Returns the key number or false.
- **:to mode** - Forces freq to the closet key number in mode.
- **:through { tuning | mode}** - Filters freq through tuning or mode and returns the closest integer tuning keynum.

### note
```common-lisp
(defun note (freq &key hz in in? through))
```

Returns the note name of freq, which can be a note name, key number, Hertz value or list of the same.

note supports the following keyword arguments:

- **:hz boolean** - If true then a numerical freq is interpreted as a Hertz value rather than as a key number.
- **:in { tuning | mode}** - The tuning or mode to return the note from. The default value is *scale*. If a mode is specified then freq must be a modal key number.
- **:in? { tuning | mode}** - Tests if freq references a note name in the specified tuning or mode. Returns the note name or false.
- **:through mode** - Filters freq through mode and returns the closest note.

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
