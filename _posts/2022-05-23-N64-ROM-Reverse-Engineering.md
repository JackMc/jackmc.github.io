---
title: N64 ROM Reverse Engineering (NorthSec 2022)
date: 2022-05-23 16:23:00
layout: post
---

This weekend I participated in the [NorthSec 2022 CTF](https://nsec.io/competition/)
with a bunch of friends who I either presently or used to work with.
Overall, the CTF was really well run and incredibly fun! Today I wanted
to talk about one of the challenges which was outside of my comfort zone
that we ended up solving after a lot of work! 
I worked on this challenge with my teammates 
Mariana and Erin, which made it even more fun! 

## The Challenge

This challenge was called The Legend of Shiitakoin. I clicked into it on the
challenge list to see that one of the attached files was an N64 ROM! It seemed
like the main point of the challenge was going to be extracting flags via playing
and reverse engineering the attached ROM. There was also an attached `.out` file,
which at the time we were not sure of the format of. 

Our first step was to run the ROM. Initially, my friend who I was working on the
challenge with tried it in the [Mupen64](https://mupen64plus.org/) emulator, and
for some reason we got a blank screen. At this point, we tried running it in a second
emulator, [SixtyForce](https://sixtyforce.com/). This popped up a screen dripping in
late-90s nostalgia:

![A picture of the SixtyForce N64 emulator running the Legend of Shiitakoin ROM. There's red mushrooms lining the bottom of the screen, and the text "Legend of Shiitakoin\nSTART: Change Stage" is centred.](/images/sixtyforce_n64.png)

## Stage 1

Pressing Start as instructed brought us to a screen with four hex numbers that, after a 
bit of fiddling with the controls, we realized could be incremented and decremented with
the A and B buttons. We also learned that the Z button switched
between the numbers.

![A picture of the SixtyForce emulator running the ROM. There are yellow mushrooms lining the bottom of the screen, and the text "00 00 00 00" is centred.](/images/sixtyforce_n64_stage1.png)

And pressing the right trigger (`s` in SixtyForce), the text "INCORRECT" would be overlaid on top of 
the numbers. Like any good hackers, this made us wonder "well, what's the correct code" 😁

### Disassembling the binary

At this point, our attention turned to how to look at the ROM under the hood. 

This lead us to the `.out` file which was included with the ROM. Running `file` on it, we got this 
output:

```
legend_of_shiitakoin.out: ELF 32-bit MSB executable, MIPS, MIPS-I version 1 (SYSV), statically linked, not stripped
```

This let us know it's an ELF binary for the MIPS instruction set. This is great, since a quick Google
told us that was the N64 instruction set! We tried the binary in [Ghidra](https://ghidra-sre.org/).
Unhelpfully, it prompted us to tell it what compiler and variant was used. To be honest, I had no idea
the answer to either of these questions! After guessing, we received a blank project.[^1] This made us try
it in a different reversing tool, Binary Ninja. I'm usure why, but for this reason we continued with
Binary Ninja.

### Finding stage 1 in the code

The first thing that stood out to me was that the `.out` file had symbols included, which was a relief!
This means most things would already have names. We easily found the main function, `mainproc` (abridged and simplified for space):

```c
int32_t mainproc() {
    ...
    stage = 0;
    while (true) {
        // Weird numbers thing
        if (stage == 1) {
            initStage01();  // Set some variables (mostly display stuff)
            nuGfxFuncSet(0x80025e5c); // stage01 function
            nuGfxDisplayOn();
            *gspS2DEX2_fifoTextEnd = 0;
            first_render = 1;
        }
        // Code for start screen and stage2
        ...
    }
}
```

This is (mostly) normal C code. It loops forever,
then checks the current stage number. If it's 
1, it calls an initialization function. Then
it calls `nuGfxFuncSet`. After a bit of Googling,
we realized this was actually part of the N64 
graphics API! The docs are [here](http://n64devkit.square7.ch/nusystem/nu_f/graphics/nuGfxFuncSet.htm). It sets a
function that is called roughly every frame 
(I think, some of the docs are a bit unclear).

The address passed in (`0x80025e5c`) is for the
function `stage01`. Clicking through, it seems 
like this function calls two functions: `makeDL01` 
and `updateGame01`. We reversed the majority of 
`makeDL01` before realizing it was all graphics
code and likely not part of the solution. I'm 
happy for this since I learned a lot about the
graphics APIs for the N64! [^3]

### The core stage 1 logic

`updateGame01` contains the code for handling
controller inputs, i.e. the "logic" of the stage. 
In this function, `nuContDataGetEx` is called, which
[according to the homebrew docs](https://n64squid.com/homebrew/n64-sdk/type-definitions/#NUContData)
returns a non-specified structure containing
the data from the controller. From this structure,
the function extracts the `button` bitmap, and
uses it to check if buttons are pressed. For example, the right trigger is checked with:

```c
if (((contdata + 6) & 0b10000) != 0)
{
    interactiveScreenShow();
}
```

From my understanding, `(contdata + 6)` is the
`button` element of the controller data struct
(it may be the second byte of the 16-bit integer).
The `& 0b10000` takes only the 5th bit, which 
represents the right trigger. 

`a` `b`, `z`, start, and the right trigger are all handled similarly.
`interactiveScreenShow` is the function that 
checks if we got the numbers correct, so let's 
jump in. 

### What are the right numbers?

`interactiveScreenShow` copies a weird base64 
string beginning with `GQMXGTN` into a buffer,
then calls `evaluationRoutine`. This function
contains a bunch of rules about what our four
numbers can be. For example:

```c
// XOR first number with second number.
char xor_of_one_and_two = input_bytes[0] ^ input_bytes[1];
int32_t return_value;
if (((uint32_t)(xor_of_one_and_two & 0xf0)) != 0xf0)
{
    // Displays INCORRECT message
    return_value = 0xffffffff;
}
else if (((uint32_t)(xor_of_one_and_two & 0xf)) != 0xd)
{
    return_value = 0xffffffff;
}
```

The rules vary, but for numbers 1, 2, and 3 they
are mostly various forms of XORs and ANDs.[^2] I 
wrote these rules into our shared Discord instance,
kind of dreading how we would actually resolve 
all of them. Then another friend suggested we 
bruteforce the numbers! It's only 32 bits total
(4 8 bit numbers), so a small C program could do 
it easily! So that's what we did. Source code for
that program can be found [here](https://gist.github.com/JackMc/a8b96a176c17412e0f763f7aae7a5979).

Compiling and running this code we get:

```bash
$ ./legend_of_scoin
a3 5e 8d 3f
```

Entering this into the game, we get the flag!!

![The stage 1 screen from above. The text FLAG-HZQC is on the left of the screen. The remainder of the flag is redacted.](/images/sixtyforce_n64_stage1flag.png)

## Stage 2

After pressing start again, we got to another screen:

![A picture of the ROM running in an emulator. There are orange mushrooms lining the bottom of the screen, a yellow mushroom in the top-left corner, and the text "X = 0 Y = 0" appears to the centre-left of the screen.](/images/sixtyforce_n64_stage2.png)

On this screen, we found out that the left DPad 
(the i, j, k, and l keys on SixtyForce) moved around the X and Y of the mushroom. 

Stage 2 is very similar to stage 1. The X and Y 
positions are put through a long series of rules
to determine which values are allowable. This 
time the rules tested each bit of each coordinate
individually (see screenshot for some examples).

![A screenshot of Binary Ninja. Several rules of the form `if ((other_y_pos & 0b1) & 0b1) != 0)` are displayed in a control flow graph.](/images/binaryninja_stage02_rules.png)

We wrote [another program](https://gist.github.com/JackMc/095a5c6c83ae24d6225615eff9cda009) to bruteforce these
rules. We ran it:

```
$ ./legend_of_scion_2
a7 4e
```

Inputted it into the program... and it was wrong!
Not going to lie, we were pretty dejected at this
moment. We didn't have an easy way to debug the 
ROM. Thankfully, after reexamining the challenge
description, it mentioned [Project64](https://www.pj64-emu.com/).
This emulator has a built in debugger, which is
exactly what we need! It only runs on Windows, 
but one of our teammates lent us their Linux 
machine with wine. 

I don't have screenshots, but we ran through the 
evaluation function `proceduralOperation` one
instruction at a time. We found that for the 
third least-significant bit, the check failed!
It seems the binary ninja decompilation was 
flipped. We change that bit to a `0`, giving us
`a3`, inputted it and got the flag!

![The stage 2 screen from above. The yellow mushroom is now at the coordinates a3, 4e. The text FLAG-HZQCFXWF is displayed in the center-bottom left of the screen](/images/skyforce_n64_stage2flag.png)

## Conclusion

All in, this challenge probably took us 8 hours.
As someone who isn't super experienced
in reverse engineering, I feel like this was 
right at the limit of do-ability for me. 

Thanks to the NorthSec team for the CTF, to my
teammates, and to Zack Deveau for putting the
challenge together! Cheers.

[^1]: We learned after-the-fact that there's a [Ghidra N64 plugin](https://www.retroreversing.com/n64-decompiling) which solves our Ghidra issues.
[^2]: `makeDL01` (and other `makeDL`s) are called conditionally based on a parameter which we found out is called `gfxTaskNum`. I couldn't find solid docs on what decides this number, if anyone knows please reach out!
[^3]: the fourth number is calculated in an interesting way. The program loops four times and binary-ORs the loop counter into it (plus some constants). This threw me through for a loop (no pun intended) because in Binary Ninja it looked like the loop counter itself was being dereferenced, at addresses 0-3. This lead to a rabbit hole about the [catridge ROM header](http://en64.shoutwiki.com/wiki/ROM#Cartridge_ROM_Header) that's at that address. Turns out Binary Ninja's decompiler was just wrong.

