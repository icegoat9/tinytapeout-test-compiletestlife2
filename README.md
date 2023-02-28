![](../../workflows/gds/badge.svg) ![](../../workflows/docs/badge.svg)

An ASIC design for [Tiny Tapeout](https://tinytapeout.com) 03. **Work in progress: see running TODO**

This implements a very simple 7-segment [cellular automaton](https://en.wikipedia.org/wiki/Cellular_automaton) from ~200 **(TBD)** logic gates, using the [Wokwi web-based logic editor](https://tinytapeout.com/digital_design/wokwi/). When the Tiny Tapeout ASIC is fabricated and mounted on its standard carrier PCB (with 8 dip switch inputs and a 7 segment display as output), it should run the below behavior in hardware.

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
You can set them to a specific pattern as noted below in "Setup".

Then, to run the automaton, you set dip switch #4 to 1 ("Run").

Set the clock slide switch to the stepping position (not "system clock"), and then toggle dip switch 1 on to advance the automaton one step.
(in theory, you could also press the "Step" pushbutton to supply one clock pulse and advance the automaton one step, but unless it is debounced, you will instead advance multiple steps). Toggle dip switch 1 off and on repeatedly to step the simulation forward.

### Free Running, Clock Divider

If you set the clock slide switch to the clock position, the automaton will advance every system clock cycle, but this will advance too fast to see. You can use the combination of the PCB clock divider and an application clock divider configured by dip switches 5-8 to slow it down to a visible speed.

*TODO:* Add clock divider and documentation.

## Setup

You can set the initial condition to a specific pattern by using dip switches #2 and #3 to shift "alive" or "dead" states into memory:
* First, set switch #4 to off ("Load" mode). The dot in the 7-segment display should come on to indicate you're in Load mode.
* If you cycle switch #2 to on and then off, it loads a 0 ("dead") into segment A (the top segment), and shifts the values of all other segments to the following segment, in the A->G order shown below:
* If you cycle switch #3 to on and then off, it loads a 1 ("alive") into segment A (the top segment), and shifts the valuers of all other segments as above.

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

## Puzzles

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

Other links I found along the way (mostly relevant to future HDL work):
* https://zerotoasiccourse.com/resources/
* https://nandland.com/
* https://8bitworkshop.com/
* https://www.joelw.id.au/FPGA/CheapFPGADevelopmentBoards
* https://www.crowdsupply.com/1bitsquared/icebreaker-fpga#products (2019)

## TODO List

* ~~Get working in Wokwi, clear out previous TODOs~~
* ~~Move README to github~~
* ~~Add 7seg period as indication of Load mode~~
* ~~Set up github project, test running build Actions~~
* Check if I'm using correct Wokwi template (8 inputs, or 10 inputs with separate clk and rst pins?)
* Design report generated by Action lists a surprisingly small # of gates and muxes, is part of this being optimized out?
  * Get someone who's done this before to look at the metrics and give input
* (WIP) Add clock divider for input (clock ~50Hz on slowest input setting, at least in tt02)
  * Redo clock divider to be smart powers of 2 (one mux cuts out 1 flipflop, another cuts out 2, 4, 16? )
    * Should be able to divide by wide range of speeds
  * Ask in discord for tt03 clock divider docs (maybe can remove most of what I have)
  * Edit diagram.json to simulate slower input clock in Wokwi
* Ask in discord about Buffers seen in some peoples' designs (but not in dropdown?) -- any need?

**Documentation-related:**
* Ask in discord about error where Wokwi locks up when editing JSON for text labels...
  * Add more Wokwi text labels
* Update docs re: clock divider (once working)
* Update info.yaml documentation (clock, how to test, how it works, etc)
* Add Wokwi link
* Graphics in README once design is done:
  * Screenshot of logic gate layout
  * Snapshot of GDS
  * Replace text 7 segment sketches with images? Or, good enough?


