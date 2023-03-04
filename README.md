![](../../workflows/gds/badge.svg) ![](../../workflows/docs/badge.svg)

An ASIC design for [Tiny Tapeout](https://tinytapeout.com) 03.

This implements a very simple 7-segment [cellular automaton](https://en.wikipedia.org/wiki/Cellular_automaton) from ~200 basic logic gates, using the [Wokwi web-based logic editor](https://tinytapeout.com/digital_design/wokwi/). When the Tiny Tapeout ASIC is fabricated and mounted on its standard carrier PCB (with 8 dip switch inputs and a 7 segment display as output), it should run the below behavior in hardware.

## Behavior

I played around with a few different rulesets and this one seemed the most interesting:

At each time step, each of the seven segments is set to "alive" or "dead" based on a simple set of rules:
1) If a segment was "alive" in the previous time step and has exactly one living neighbor, it survives.
2) If a segment was "dead" (or "empty") in the previous time step and has exactly two living neighbors, if becomes alive ("gives birth")

A "neighbor" is any segment it touches, tip to tip. This means that the top and bottom segments only have two neighbors, while the side segments have three neighbors and the center segment has four neighbors.

### Example + Explanation

From the initial state shown in Step 0 below (where thin lines represent dark/dead segments, and solid blocks represent lighted/living segments), the automaton will evolve through each of the following states:
```
Step 0         Step 1         Step 2         Step 3         Step 4

 ███            ███            ---            ---            --- 
█   |          █   █          |   |          |   |          |   |          
█   |          █   █          |   |          |   |          |   |   
 ---            ███            ---            ███            --- 
|   █          |   |          █   █          |   |          █   █
|   █          |   |          █   █          |   |          █   █
 ---            ---            ---            ███            ---
```
To determine the state in Step 1:
* The two connected segments in the upper left of Step 0 survive (as they each have one living neighbor)
* The disconnected segment in the lower right of Step 0 dies (zero neighbors)
* Two segments are born in previously-empty locations: the upper right (it has the top segment and lower right segment in Step 0 as two living neighbors), and the middle segment (the upper left and lower right are its two living neighbors).

During Step 2, all the four segments in Step 1 die (two neighbors), but two new segments are born.

During Step 3, two segments die and two more are born.

During Step 4, two segments die and two more are born, and we realize it's in an infinite loop between this state and the previous state.

## Running

On power-up, the segments will be initialized to an unpredictable state.
You can also set them to a specific pattern as noted below in "Setup".

To run the automaton step by step, first set dip switch #4 to on ("Run" mode).

Ensure dip switches #5,6,7 are off (these are used to enable a clock divider when running in automatic mode, see below, but interfere with manual stepping).

Set the PCB clock slide switch to the "manual clock" position (not "system clock"), and then toggle dip switch 1 on to advance the automaton one step. Toggle dip switch 1 off and on again to step the simulation forward.

### Free Running

If you have dip switch #4 on but instead have the PCB clock slide switch to the "system clock" position, the automaton will advance every system clock cycle.

This will run it too fast to see by eye. To slow it down, first configure the PCB-level clock divider to its slowest speed, which I believe should set the clock to 6250Hz/256 ~ 24Hz. Then use dip switches 5, 6, and 7 to further slow it down. Turning on dip switch 5 divides the clock by 8, dip switch 6 divides the clock by 4, and dip switch 7 divides the clock by 2. So turning on dip switches 5 and 7 should give you a clock of ~1.5Hz.

## Setting Initial Pattern

You can set the initial state to a specific pattern by using dip switches #2 and #3 to shift "alive" or "dead" states into memory:
* First, set dip switch #4 to off ("Load" mode). The period in the 7-segment display should come on to remind you you're in Load mode.
* If you cycle switch #2 to on, it loads a 0 ("dead") into segment A (the top segment), and shifts the values of all other segments to the following segments, in the A->G order shown below:
* If you cycle switch #3 to on, it loads a 1 ("alive") into segment A (the top segment), and shifts the valuers of all other segments as above.

Don't forget to set switch #4 back to on ("Run") when you want to run the automaton!

```
 AAA
F   B 
F   B 
 GGG
E   C
E   C
 DDD
```

### Example

```
Initial        After cycling        After cycling        After cycling
state:         switch #3            switch #3 again      switch #2
 ---               ███                  ███                  ---           
|   |             |   |                |   █                |   █           
|   |             |   |                |   █                |   █           
 ---               ---                  ---                  ---                 
|   |             |   |                |   |                |   █                       
|   |             |   |                |   |                |   █                       
 ---               ---                  ---                  ---                 
```

## Exercises For The Reader

* How many unique initial states are there, disregarding equivalent mirrored/rotated states? (there are 2^7 = 128 possible initial states but many are equivalent)

* What fraction of these initial states survive? (i.e. don't eventually die out)

* What fraction settle into a static living pattern vs an infinite cycle between multiple different patterns?

* What is the longest sequence of unique states a pattern travels through (stop counting once it reaches a previously-visited state, beginning an infinite loop)?

* What is the longest cycle of unique states that repeats in a loop? (there's an example of a period 2 cycle in the example above)

## Related Resources

Copied from the [TinyTapeout](https://tinytapeout.com) template repo:
* [TinyTapeout FAQ](https://tinytapeout.com/faq/)
* [Digital design lessons](https://tinytapeout.com/digital_design/)
* [Learn how semiconductors work](https://tinytapeout.com/siliwiz/)
* [Discord community](https://discord.gg/rPK2nSjxy8)

## TODO List

No blocking items, ready to submit...

**Low Priority**
* Learn [how to do HDL testing](https://tinytapeout.com/hdl/testing/) on the Verilog generated by the toolset
* Visual cleanup: replace not+and+and+or outputs of life/death testing with muxes? (doesn't affect behavior, will be optimized out either way)

**Documentation-related, can do later:**
* Ask in discord about slowdown where Wokwi locks up when editing JSON for text labels...
* Maybe add some graphics to this README at some point:
  * Screenshot of logic gate layout
  * Snapshot of GDS 3D render
  * Replace text 7 segment ascii art with images? Or, is this good enough?


