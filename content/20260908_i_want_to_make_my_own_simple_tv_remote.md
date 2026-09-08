+++
title = "I want to make my own simple tv remote"
date = "2026-09-08"
[taxonomies]
tags = ["rust", "code", "electronics", "hardware", "tv", "remote"]
+++

I want to make my own simple tv remote. I've been thinking of making one for years, I only use a handful of buttons and I need it to coopeate with multiple devices at the same time. In this article I'll show how I made it and what I learned, ending with the answer to the question: does it check all the boxes?

### Why now?

I slept with the Apple TV remote in my hand and it got in the bed sheets... that my wife put in the washing machine. She felt so guilty, even though it was my fault, but I felt so energized.

So after ordering sushi and ensuring my wife is happy again, it's time for our family to have our own customized tv remote!

### Addressing the elephant in the room: Universal remotes

Universal remotes are a no go, the reason is easy to plot into a table:

| Remote   | Buttons                | Functionality                 | Enjoyment of the process |
| -------- | ---------------------- | ----------------------------- | ------------------------ |
| Universl | 🚫 too many or too few | ✅ great                      | 🚫 none                  |
| Mine     | ✅ perfect amount      | ✅ as good as I want it to be | ✅ lots                  |

### Goal

I have a Philips TV with an Apple TV plugged into it. The tv remote has to work with both, these are the buttons need:

- **Apple TV**: UP, DOWN, LEFT, RIGHT, SELECT, MENU, HOME, VOL+, VOL-, MUTE, PLAY/PAUSE
- **Philips TV**: UP, DOWN, OK, SOURCES

**Battery life**: at least 1 month of usage without charging \
**Battery**: rechargeable with USB TYPE C \
**Buttons**: silent \
**Usage**: mostly from under the sheets

{% admonition(type="info") %}
Most of it can be achieved with the Apple TV remote, but few things are missing: there is no SOURCES button, and I couldn't for the life of me configure it so that I can press OK from the Apple TV remote when the Philips TV asks me if I'm still watching. Sometimes I'm sleeping and I want the tv to turn off, sometimes I'm watching and I want to press OK.
{% end %}

### Plan

Use Rust to write embedded code, use Bluetooth to connect to Apple TV, use IR to command the Philips TV, learn to use resistors, transistors, capacitors, leds, IR and Bluetooth protocols.

Use the _Raspberry Pico W_ I have for the testing phase, buy a better chip for the final result.

Use the 3D printer to print the shell.

### Where do I start?

That's the question. I need to learn embedded programming and electronics and I only have an extremely basic knowledge of both from school and personal projects.

I'll start from turning on a led by the press of a button. I'll use `embassy` in the code to control the RPico and I only have the wiring left to do.

`embassy` supports the RPico through the `embassy-rp` crate, super easy to setup:

```rs

```

# TODO: also show the result as a gif, it would be cool to put it next to the code if it fits

The resistor is there to offer resistance, taking on some voltage and lower the current, otherwise the led I bought, that is designed for 2~ Volt, dies from the 3.3 Volts from the RPico.

{% admonition(type="info") %}
I know the GPIO reduces the voltage and yada yada, I don't really understand it so let's assume it doesn't do that.
{% end %}

This was simple! Unfortunately, this is the first and last simple concept.

### 3 Rs: Read, Record, Replay - The setup

Most TVs have remotes that use infrared (IR) to send commands to them, and Philips TVs are among those. But what and how should we send to the TV?

I could have found the specification online, _but that's not fun_.

So I bought an IR Receiver, called TSOP. The plan is to turn it on, press the button on the Philips TV remote, read what it sends, record it and replay it when pressing the button of my new tv remote.

# TODO: show an image of the tsop on the right next to the above paragraph

After some wiring, learning what a Low Pass Filter is, I can start testing it with my multimeter and hope I built everything correctly... well, it's not?

# TODO: show the image of the multimeter showing 1.x when measuring the TSOP due to the GPIO's shenanigans

The above should show 3.3V! Without any IR received, the TSOP should output the same input voltage (3.3V from the RPico 36th pin), so where did the leftover voltage go?

The TSOP output was wired to a GPIO that I wanted to use as an input to read the values, but that meant that the voltage gets affected by it. Apparently there is a concept of "pull" for the GPIOs, to define what GND means for them, in case of `Pull::DOWN` the GPIO uses the internal resistor, and as I learned before, resistors reduces the voltage in the circuit.

Easy fix:

```rs
// gpio with Pull NONE
```

# TODO: show image of 3.3~ V reading

Now we're talking.

{% admonition(type="info") %}
Above I mentioned the Low Pass Filter, it's used for
{% end %}

# TODO: conclude the above for Low Pass Filter

### 3 Rs: Read, Record, Replay - The maths

Next, let's use this bad boy to read the IR emitted by the Philips TV remote. This means understanding how the TSOP works. The IR receiver transform
