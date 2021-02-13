# AsciiSeq

> **?** This name sucks. Need something better.

This is an idea for a text based midi sequencer inspired by SonicPi and Orca

**NO CODE HAS BEEN WRITTEN YET** This is still in the early design phase.

## Core idea

Sequences of notes can be defined easily, for example

`1 = a,c,f,g,e,a` and `2 = e,e,e,a,f,g`

then these sequnces can be chained with simple rules

```
1 -> 2
2 -> 1
```

This would just play 1 and 2 in a loop forever.

## Chaining Sequences
Note, that any sequence entry point that is the `number ->`  must be unique, that is, each sequence destination can only be defined once.

Basic sequence chaining:
```c
1 -> 2
2 -> 3
```

Probabilistic Sequence Chaining. Probability is in increments of 10%. Not having a total of 100% will cause the remaining chance to be recursive.
```c
3 -> 5:4~5:5 		
// 50% chance to move to 4, 50% chance to move to 5
4 -> 1:5
// this would have 90% chance of repeating, 10% to go to seq 5
4 -> 1:5~9:4
// would be exactly equivilent. Note that having both these lines
// would be invalid as 4 can't be defined twice
```

variables aa-zz can store complex ops to used elsewhere. All lines must have a go-to (->) even if it results in an infinite loop

> **?** `0→_` can stand in for immediately redirecting to a new decision

note, that when using a variable in the definion of another varibale the go-to of the inner variable will be overridden.

```c
a = 5x4 -> 2
// repeat a sequence, here seq 4 is repeated 5 times, then goto 2
b = 3o4 -> 2
// the number of notes of a sequence to play (3 notes of seq 4)
c = 1u4 -> 2
// play the sequence shifted up one octave
d = 1d4 -> 2
// or down an octave
e = 2s4 -> 2
// s for slow, 1/2 speed
f = 3s4 -> 2
// or 1/3 speed, etc.
g = 2f4 -> 2
// f for fast, 2x speed
h = 1u2s3x3o4 -> a
// 1 octave up, repeat 3x, at half speed, the first 3 notes of sequence 4
i = 0 -> 3:5xb~7:5  // 30% chance to play the sequnce at b 5 times, 70% to go to seq 5
zz = ...
// variables can be up to two leters in length
5 -> A|5
// until statements, to wait on external input A-Z, think a BARS 5
6 -> 5:g~3:h
// 50% chance of going to g's definiton, 30% → h, 20% chance to 
// repeat sequence 6 before deciding again
7 -> 2 -> 4
//the redirect to sequence 3 from sequnce to 2 is overridden, to go to 4 instead
8 -> 0r1
//play sequence 1 in reverse, note the 0 because 
// reverseing doesn't have any options
// this just keeps syntax consistant
```

| Operator | Name   | Description                                              |
| -------- | ------ | -------------------------------------------------------- |
| ->       | GoTo   | What sequence to play after the source sequence finishes |
| x        | Repeat | Number of Times to repeat the sequence                   |
| o        | Of     | Number of notes to play of the sequence.                 |
| u        | Up     | Number of Octaves Up to play sequence                    |
| d        | Down   | Number of Octaves Down to play sequence                  |
| s        | Slow   | slow down by division                                    |
| f        | Fast   | Speed up by multiple                                     |
| :        | Chance | ~ to chain                                               |
| \|       | Bar    | wait for input                                           |

## Making A Sequence

avail notes are c,c#,d,d#,e,f,f#,g,g# and n where n is no note. notes can also be defined as {40}, for example, to play a midi note number.

> **?** flat would be nice, but I think having syntax like eb is very confusing and I don't see a better symbol? maybe e♭ with unicode, but that's a bitch to type and fucks up the otherwise ascii format

> **?** syntax for chords in scale?

> **?** syntax highlighting for notes that are out of scale?

CC's can be set wit {CCnum=val} or {CCLnum=val}

where the L(ock) variant will default back to the default value set for that CC in the config after the note off event

> **?** it would be nice to do time, like with PB, but that would depend on knowing the current CC value or at least the last value. Not sure if this is possible. Might need to be a per-CC option?

Other sequences can be referenced in a sequence with [seqnum]

> **?** This may require special attention to recursion, what would 4 = [4],a,c do? should it be invalid?

Pitch Bend can be set with {PB=±cents,time}. Assumes a PB range of ±2 semitones unless overridden and time is a 10% fraction of the gate time for the bend to complete. 0 for instant.

> **?** if more than one bend at a time is possible, this will require MPE. That could be rather complicated

Octave shifts can affect an entire line or a particular note

`1 = a,b@+1,c,d,e,f,g @-1`

@-1 at the end means to take the entire sequence down 1 octave. If the @-1 *didn't* have a space before it, it would only apply to the g before it, as is the case with the b@+1

> **?** should grouping an octave shift be possible? maybe `1 = {a,b}@+1,c,d,e,f,g @-1` to take the first two notes up an octave?

polyphony is possible using ';' to separate chords

`2 = a;c;e,a;d;f,a;c;f;{CC34=127}`
where the this would play a c and e together, followed by a d and f, then a c and f along with a CC change. Note the CC is done polyphony to avoid the CC taking a step of its own.

> **?** this syntax is a bit hard to read, should there be better grouping symbols? maybe `2 = {a;c;e},{a;d;f},{a;c;f;{CC34=127}}`



```c
3 = ^(a,d,f,a),d,f,a,e
// where ^() is an arpegio forced to fit in the time, so here would be a 4 note arp a,d,f,a
// ^_() will try force a constant gate (no retriggers)
4 = 5:a~5:c,e,5:a~5:!n,c
// 50% chance a, 50% chance c, then e, then a 50% chance of a or no note, then c
5 = [4],a,c
// play sequence 4, followed by a,c
6 = [4d1,4u1],a,c
// play sequence 4 shifted down an octave, followed by seq 4 up 1oct, followed by a,c
7 = [f],a,c
// play sequence at var f, ignoring the go-to, followed by a,c
8 = [1&2]
// play sequence 1&2 in parallel, using polyphony
9 = [0r3],a,c,f
// play seq 3 in reverse
10 = c,!e,f
// ! doesn't send a gate with the note... (how would this work with midi?)
11 = c;{PB=-50,5},e,g
// pitch bend down 50 cents, taking 50% of the note gate time to do so.
12 = $c,d,e$,g,a,c
// start holding the sustain pedal before C is played, release it before g is played
13 = !n,!n,!n,!n
// play no note for 4 notes. Note, that without the '!', this would still emit a gate if
// the last note can have it's gate sustained,
// on that note
14 = c,!n,n
// is equivilent to c,!n,c, as to send a gate (note on) signal requires some note, so the last 
// played note or group of notes is used.
15 = c,{PB=0,5}e,f
// will attempt to pitchbend from c to e in 50% of the gate of the e
15 = c,e{PB=+100,1},f
// will attempt to pitchbend from c to e+100cents in 10% of the gate of the e
// note, if using polyphony and multiple pitch bends are used I could either do MPE
// (probably very, very hard) or just take the first/last pitchbend
// orrrr just throw a syntax error
// note the difference here of the pitchbend not getting it's own note channel, that is
// before the pitch bend was being done polyphonically and in it's own spot
// where this one has a note before it.
16 = c...,e...,f...
// is the same as c,c,c,c,e,e,e,e,f,f,f,f UNLESS the gate lengths have been defined to be under a note
// long, this is a forced gate hold for the sake of operators that are dependent on gate time, 
// the length of the total gate is what the percentage is of, so 
17 = c,e...{PB=0,1},f
// will take 10% of the time of the 4 note gate, not of just one note. This allows for long glides
18 = ^(a,a,a,a),d,f,a,e
// ^() can be used for ratchets
19 = ^(a...)
// when the ^() sytax is used with an extended gate, a ratchet is implied, note that adding more '.'s
// further divides the time, making the ratchet faster
20 = ^(c.......)
// would be a twice as fast ratchet (8 notes in the time of 1)
21 = ^(d*12})
// is the same as ^(d,d,d,d,d,d,d,d,d,d,d,d) is the same as ^(d...........)
22 = c*3,e*97
// this syntax can be used anywhere, but it's a bit gross and easy to mistake for sharps
// still useful for when a note needs to be repeated many times
23 = c#,c#*3,e*97
// I mean it's not that bad, but it's a bit symbol-y
// note that *0 will play indefinitely.
24 = $c,{PB=0,5}e..$,^(f,a,c#);[1&2],[4d1],e,f,[1u2s3x3o4];{CCL34=127},g
// and with that we have a sytax that can get comically gross.
// note on polypony
25 = c;e...,a,d
// the first note will define the length of the note group, so here the c,e... cluster will
// only take 1 note space, and the e... will sustain over the a and d, as well as into the
// first note space of the next pattern
// technically, this could be used to make a drone
26 = !n;e*0
```

## Configuration Options

support for some global statements at top of each MIDI channel or top of file (global). Some are Global only.

this will require tracking some stats, such as how many times a sequence is played.

```c
MAX_PLAYS = 20
MAX_GOTO  = 1
// prevent any sequence from playing more than 20 times consequtively
MAX_NOTE = {70}
MIN_NOTE = {8}
// constrain note pitch
PBrange = 24
// set pitchbend range to 24 semitones. Assumes symmetric, so this is ±12 semitones.
CLK_IN = FALSE
// ignore the input midi clock from an external source
CLK_RATE = 120
// BPM if no external source
TRANSPOSE = -12
// Transpose everything down 1 octave
TRANSPOSE_U = +100
// Transpose Everything up 100 cents
RAND_SEED = whatever
// seed for random stuff. If not set, a new seed is chosen on each clock start/stop
CC24 = 0
CC25 = 64
// set the default values for CC's
SCALE = minor
// set the scale used to highlight out-of-scale notes
LAZY_CMAJOR = minor
// sets the notes of the major scale to just be indicators in a different scale
// this is super gross, but for example in C minor
// c stays c, d stays d, e becomes d#, f stays f, g stays g, a becomes a#, and b becomes b#
// this lets noobs like me treat everything like it's major for writing it. This will throw a 
// warning if you try to use a note off the major scale
// combine with transpose to get different keys
// options should be major (no change, but warn about non-major keys), minor, minor_mel,
// minor_harm, penta, and blues. not sure what to do with those with more than 7 notes.
// might let the notes availble expand, so for blues it could be
// a is a, b is a#, c is b, d is c, e is d, f is d#, g is e, then go into non-notes, so
// h is f, i is f#, and j is g
DRUMS = true
// use k,s,h,t,c as kick, snae, hat, tom, clap etc. assuming the standard midi drum mapping.
```
## Multiple Channels

Midi supports 16 channels. To take advantage of this, code can be surrounded with tags. This would only include the pattern chaining, as the sequences can go up to arbitrary numbers. This would allow for sharing sequences between midi channels, while still having different sequence chains. Per channel settings can be defined in these blocks as well.

```c

---1---
1 -> 2
2 -> 1
---1---
    
---2---
MAX_PLAYS = 3
MAX_GOTO  = 1
1 -> 2
2 -> 3
3 -> 5:1~5:3
---2---

---16---
TRANSPOSE = -12
1043 -> 234
...
---16---
```