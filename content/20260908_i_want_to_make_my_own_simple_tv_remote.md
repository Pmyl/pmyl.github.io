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
| Universl | 🚫 too many or too few | ✅ good                       | 🚫 none                  |
| Mine     | ✅ perfect amount      | ✅ as good as I want it to be | ✅ lots                  |

### Goal

I have a Philips TV and an Apple TV plugged into it. The tv remote has to work with both, these are the buttons need:

- **Apple TV**: UP, DOWN, LEFT, RIGHT, SELECT, MENU, HOME, VOL+, VOL-, MUTE, PLAY/PAUSE
- **Philips TV**: UP, DOWN, OK, SOURCES

**Battery life**: at least 1 month of usage without charging \
**Battery**: rechargeable with USB TYPE C \
**Buttons**: silent \
**Usage**: from under the sheets

{% admonition(type="info") %}
Most of it can be achieved with the Apple TV remote, but few things are missing: there is no SOURCES button, and I couldn't for the life of me configure it so that I can press OK from the Apple TV remote when the Philips TV asks me if I'm still watching. Sometimes I'm sleeping and I want the tv to turn off, sometimes I'm watching and I want to press OK.
{% end %}

### Plan

Use Rust to write embedded code, use Bluetooth to connect to Apple TV, use IR to command the Philips TV, learn to use resistors, transistors, capacitors, leds, IR and Bluetooth protocols.

Use the _Raspberry Pico W_ I have for the testing phase, buy a better chip for the final result.

Use the 3D printer to print the shell.

### Where do I start?

That's the question. I need to learn embedded programming and electronics and I only have an extremely basic knowledge of both from school and personal projects.

So, blinking RPico led!

`embassy` supports the RPico through the `embassy-rp` crate, super easy to setup:

```rs

```
