+++
title = "I want to make my own simple tv remote"
date = "2026-09-08"
[taxonomies]
tags = ["rust", "code", "electronics", "hardware", "tv", "remote"]
+++

I want to make my own simple tv remote. I've been thinking of making one for years, I only use a handful of buttons and I need it to coopeate with multiple devices at the same time. In this article I'll show how I made it and what I learned, ending with the answer to the question: does it check all the boxes?

### Why now?

I slept with the Apple TV remote in my hand and it got in the bed sheets... that my wife put in the washing machine. She felt so guilty, even though it was my fault.

So after ordering sushi and ensuring my wife was happy again, I felt so energized, it was time for our family to have our own customized tv remote!

### Addressing the elephant in the room: Universal remotes

Universal remotes are a no go, the reason is easy to plot into a table:

| Remote    | Buttons                | Enjoyment of the process |
| --------- | ---------------------- | ------------------------ |
| Universal | 🚫 too many or too few | 🚫 none                  |
| Mine      | ✅ perfect amount      | ✅ lots                  |

### Goal

I have a Philips TV with an Apple TV plugged into it. The tv remote has to work with both, these are the buttons need:

- **Apple TV**: POWER, UP, DOWN, LEFT, RIGHT, SELECT, MENU, HOME, VOL+, VOL-, MUTE, PLAY/PAUSE
- **Philips TV**: POWER, UP, DOWN, OK, SOURCES

**Battery life**: at least 1 month of usage without charging \
**Battery**: rechargeable with USB TYPE C \
**Buttons**: silent \
**Usage**: mostly from under the sheets

{% admonition(type="info") %}
Most of it can be achieved with the Apple TV remote, but few things are missing: there is no SOURCES button, and I couldn't for the life of me configure it so that I can press OK from the Apple TV remote when the Philips TV asks me if I'm still watching. Sometimes I'm sleeping and I want the tv to turn off, sometimes I'm watching and I want to press OK.
{% end %}

### Plan

Use Rust to write embedded code, use Bluetooth to connect to Apple TV, use IR to command the Philips TV, learn to use resistors, transistors, capacitors, leds, IR and Bluetooth protocols along the way.

Use the spare _Raspberry Pico W_ for the testing phase, buy a better chip for the final result.

Use the 3D printer to print the shell.

### Where do I start?

That's the question. I need to learn embedded programming and electronics and I only have an extremely basic knowledge of both from school and personal projects.

I'll start from something that requires me to learn a bit of both: turning off a led by the press of a button. I'll use `embassy` in the code to control the RPico and I only have the wiring left to do.

`embassy` supports the RPico through the `embassy-rp` crate, super easy to setup:

{% code_with_image(image_top="22%") %}
```rust
#[embassy_executor::main]
async fn main(spawner: Spawner) {
    let p = embassy_rp::init(Default::default());
    let mut button = Input::new(p.PIN_13, Pull::Down);
    let mut led = Output::new(p.PIN_15, Level::High);

    loop {
        button.wait_for_high().await;
        led.set_low();

        button.wait_for_low().await;
        led.set_high();
    }
}
```
<br>
%%%
![Press button -> light LED](/i_want_to_make_my_own_simple_tv_remote/button-led.gif)
{% end %}

The resistor is there to lower the current in the circuit to protect the LED, otherwise the circuit would have low resistance and the LED would die from the high current from the RPico.

{% admonition(type="info") %}
**I = V / R** (Amps = Volts / Ohms) \
When **R** is low, **I** is high
{% end %}

This was simple! I'm sure the rest of the steps will follow suit ;)

### 3 Rs: Read, Record, Replay - The setup

Most TVs have remotes that use infrared (IR) to communicate, and Philips TVs are no different. But how and what should we send to the TV?

I could have found the specs online, _but that's not fun_.

{% two_columns() %}
So I bought an **IR Receiver**, specifically a *TSOP4838*. The plan is to turn it on, press the button on the Philips TV remote, *read* what it sends, *record* it and *replay* it when pressing the button of my new tv remote.
%%%
<img src="/i_want_to_make_my_own_simple_tv_remote/tsop.png" alt="TSOP4838" style="height: 130px; margin: 0;">
{% end %}

After some wiring, learning what a Low Pass RC Filter is and how pull up/down work, I can start testing it with my multimeter and... it's not built correctly?

![Receiver OUT 1.28V](/i_want_to_make_my_own_simple_tv_remote/receiver_128v.png)

The above should show a value slightly lower than 3.3V, not 1.28V!

The voltage in input to the receiver is 3.3V minus whatever voltage is across the resistor, and the resistor is only a 100 Ohm, stealing only a small part of the voltage. From my measurements it's 0.03V, leaving 3.27\~V in input to the receiver.
Then, without any IR received, the receiver should be **pulled up** and output the same input voltage, so where are the remaining 3.27-1.28=**2\~V**?

*Found them!* The receiver's output is wired to a GPIO that I want to use as an input to read the signal, but that means that the voltage is affected by it, specifically by its resistor that becomes part of the circuit when the GPIO is **pulled down**. And we definitely all know that that's the default state right? *Surely there is no need to waste 20 minutes struggling with it.*

In any case, for our purposes we want to use that pin to just be an observer of the voltage running across it, for that there is an easy fix:

```rs
// Neither pulled down nor pulled up, just an observer
let ir_rx = Input::new(p.PIN_15, Pull::None);
```

![Receiver OUT 3.27V](/i_want_to_make_my_own_simple_tv_remote/receiver_327v.png)

Now we're talking.

{% admonition(type="info") %}
Please go read how Low Pass RC Filter and pull up/down/none work, they are fascinating mechanisms!
{% end %}

### 3 Rs: Read, Record, Replay - Read

Next, let's use this bad boy to read the IR emitted by the Philips TV remote. This means knowing how the IR receiver works: there is a chip inside that **pulls down** when it sees an IR signal of around 38kHz, and **pulls up** when it doesn't. The practical effect is that when the Philips TV remote sends the signal, the receiver's output has 0V across it, otherwise it has 3.3~V, and since that's connected to the RPico, we can see in code if the pin reads high or low.

// let's read, read randomly

// adjust by waiting for a pause

// identify that the lead is around 2700 micro seconds, matching RC6 protocol, link to https://www.sbprojects.net/knowledge/ir/rc6.php

### 3 Rs: Read, Record, Replay - Record

// decode signal

// some panics (damn I don't have the debug probe, but looking at the code was enough), when the signal ends in 0 we don't really notice it, we have to manually add it

// variance and errors, only pick the output if it works 3 times in a row

### 3 Rs: Read, Record, Replay - Replay

// Do it first, then write it
