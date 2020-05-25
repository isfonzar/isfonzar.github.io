---
layout: post
title:  "Building an 8-bit Computer - Part 1: Architecture and Clock Module"
date:   2020-05-23 22:00:00 +0200
modified_date: 2020-05-25 21:00:00 +0200
categories: hardware electronics
---

This is the first post of what I intend to be a series of posts on my journey to build an 8-bit computer from scratch. 

The motivation to start this project came from [Ben Eater's videos](https://www.youtube.com/user/eaterbc), as well from my desire to better understand how a computer works internally and to brush up my skills on electronics.

## Computer Architecture - SAP-1

The architecture of the 8-bit computer I'm going to build is the **SAP-1** (Simple as Possible), introduced by Albert Paul Malvino in his book Digital Computer Electronics.

[![](/assets/8-bit-computer/computer-architecture-malvino.png)](/assets/8-bit-computer/computer-architecture-malvino.png)

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

### Instruction Set

SAP-1's instruction set is fairly simple and contains only the following 5 instructions:

- **LDA**: load the accumulator
- **ADD**: adds the contents of memory location to the accumulator
- **SUB**: subtract the contents of memory location from the contents of the accumulator
- **OUT**: transfer the contents of the accumulator to the output register.
- **HLT**: stands for halt, tells the computer to stop processing data.

## First module: Computer Clock

The clock is the module that is going to synchronize all the operations of the computer. Ben Eater builds 3 different clocks and by doing so, introduces the 3 different ways of using the 555 timer IC: an integrated circuit (also known as chip) that outputs pulses of electrical current for certain amounts of time.

[![](/assets/8-bit-computer/555-ic-thumb.jpg)](/assets/8-bit-computer/555-ic.jpeg)

### Astable 555 timer

With the 555 timer built in the Astable configuration, the timer will behave like an ocillator, producing pulses of electrical current for a certain amount of time, then stop sending this pulses for some time and then continuing.

![](/assets/8-bit-computer/astable-555-timer.gif)

The frequency of the oscillator can be customized by changing the value of the two resistors and the capacitor, and it's defined mathematically in the [555 datasheet](http://www.ti.com/lit/ds/symlink/lm555.pdf):


The charge time (output high) is given by:

t<sub>high</sub> = 0.693 (R<sub>A</sub> + R<sub>B</sub>) C

The discharge time (output low) by:

t<sub>low</sub> = 0.693 (R<sub>B</sub>) C

However, as Ben Eater did, I also changed one of the resistors for a potentiometer, an component in which the resistance can be adjusted, allowing me to control the frequency of the pulses by turning the dial:

![](/assets/8-bit-computer/astable-555-timer-potentiometer.gif)

### Monostable 555 timer

In this mode, the 555 timer outputs only a single pulse of electrical current. 

The duration of the pulse can also be adjusted by modifying either the resistors or the capacitor, but in this case, since the duration is not important, it's kept short.

By connecting it with a button, we can make the 555 timer only emit the pulse when the button is pressed:

![](/assets/8-bit-computer/monostable-555-timer.gif)

It's useful to have the clock in this configuration for debugging purposes, since we can send one pulse of current at a time and observe the behavior of the computer.

### Bistable 555 timer

Bistable mode means that our clock will alternate between two stable modes. Unlike the two previous modes, this configuration does not involve timing, it's just alternating between `on` and `off` states.

![](/assets/8-bit-computer/bistable-555-timer.gif)


### Putting it all together

To finish the clock module, I connected the outputs of the `Astable clock` and the `Monostable clock` to some logical gates that will feed the `Bistable clock`.

![](/assets/8-bit-computer/computer-clock-1.gif)

#### Next Steps

Building the clock module as presented by Ben Eater was a great way to learn the 3 possible configurations of the 555 timer.

In the next post, I am going to assemble the registers of the SAP-1 computer.

#### References

- [eater.net](https://eater.net/8bit/clock)
- [555 timer astable mode](https://www.circuitbasics.com/555-timer-basics-astable-mode/)
- [555 timer monostable mode](https://www.circuitbasics.com/555-timer-basics-monostable-mode/)
- [555 timer bistable mode](https://www.circuitbasics.com/555-timer-basics-bistable-mode/)