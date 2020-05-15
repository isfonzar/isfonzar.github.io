---
layout: post
title:  "Building an 8-bit Computer from Scratch - Part 1"
date:   2020-05-15 08:25:31 +0200
categories: hardware electronics
---

This is the first post of what I intend to be a series of posts on my project to build an 8-bit computer from scratch. 

The motivation to start this project came from [Ben Eater's videos](https://www.youtube.com/user/eaterbc) and design.

## Architecture

The architecture of the computer is the **SAP-1** (Simple as Possible), introduced by Albert Paul Malvino in his book Digital Computer Electronics.

![](/assets/8-bit-computer/computer-architecture-malvino.png)

_(SAP-1 architecture from Digital Computer Electronics)_

The SAP-1 computer features:

- 8 bit W bus.
- Program counter – initialized to 0000 to incremented up to 1111 for program execution.
- Memory Address Register (MAR) to store memory addresses. 
- 16 x 8 static TTL RAM that will hold our programs and data.
- Instruction register.
- Controller-Sequencer.
- Accumulator to store intermediate answers during a computer run.
- Adder-Subtracter.
- B register to be used in arithmetic operations.
- Output register to be loaded by the accumulator at the end of a computer run.
- And lastly, a binary display made of eight light-emitting diodes (LEDs).

## Instruction Set

SAP-1's instruction set is fairly simpel and contains only the following 5 instructions:

- **LDA**: load the accumulator
- **ADD**: adds the contents of memory location to the accumulator
- **SUB**: subtract the contents of memory location from the contents of the accumulator
- **OUT**: transfer the contents of the accumulator to the output register.
- **HLT**: stands for halt, tells the computer to stop processing data.

## Computer Clock

I started by buying a simple electronic kit that already came with a Breadboard, a power supply and a few components I would be using later on.

<a href="/assets/8-bit-computer/001-electronics-kit.jpeg" target="_blank">
    <img src="/assets/8-bit-computer/001-electronics-kit.jpeg" alt="elegoo-electronics-starter-kit" width="250" height="300"/>
</a>

<a href="/assets/8-bit-computer/002-electronics-kit.jpeg" target="_blank">
    <img src="/assets/8-bit-computer/002-electronics-kit.jpeg" alt="elegoo-electronics-starter-kit" width="250" height="250"/>
</a>

I also bought some 555 integrated circuits (chip). The 555 timer IC. The datasheet for the purchased 555 can be found [here](https://www.ti.com/lit/ds/symlink/se555m.pdf?HQS=TI-null-null-alldatasheets-df-pf-SEP-wwe).

<a href="/assets/8-bit-computer/003-555-ic.jpeg" target="_blank">
    <img src="/assets/8-bit-computer/003-555-ic.jpeg" alt="elegoo-electronics-starter-kit" width="250" height="300"/>
</a>

## Building the Clock module

The clock synchronizes all of the components in our computer and it's the first component in the project.
In this project, it is split into 3 different modules:

- Astable clock
- Monostable clock
- Bistable

### Astable 555 timer

In astable mode, the 555 timer acts as an oscillator that generates a square wave.

The frequency of this wave, or how many pulses it sends can be changed by adjusting the values of the two resistors.



### References

- [eater.net](https://eater.net/8bit/clock)
- [555 timer astable mode](https://www.circuitbasics.com/555-timer-basics-astable-mode/)