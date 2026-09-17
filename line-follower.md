---
layout: default
title: Analog Line Follower Robot
---

Electrical Engineering Student | Embedded Systems & Electronics

[View the Project on GitHub NoOneReadsUsernames/JulianaYasis](https://github.com/NoOneReadsUsernames/JulianaYasis)

# Analog line follower robot

**Analog Circuits Lab • Op-Amps • Comparators • 555 Timers • DC-DC Conversion**

*Team project — final lab assignment*

A line-following robot built entirely from analog circuitry, with no
microcontroller. The robot detects a black line on a white floor
using IR reflectance sensing, and uses discrete analog logic —
rather than code — to decide when to turn.

---

## Overview

The robot uses IR reflectance sensing to detect a black line against
a white floor, comparator circuits to convert that sensor signal into
a clean on/off turning decision, and a pair of 555 timer circuits
wired as flip-flops to hold that turning state per side of the robot.
Because the sensing circuit's output voltage was too small to
reliably drive the rest of the system, a voltage boost circuit was
added to step it up from the battery pack.

---

## Circuit Design

### Line sensing — IR LED and IR sensor pair

Each side of the robot uses an IR LED paired with an IR sensor. Since
the line was black and the floor was white, the circuit works by
checking how much IR light reflects back to the sensor — white floor
reflects more, black line reflects less. The brightness of the IR
LEDs was tuned with a potentiometer to get a reliable difference
between the two surfaces.

### Comparator circuit — turning decision

The IR sensor's output feeds into a comparator circuit, which
converts the analog reflectance signal into a clean on/off signal —
essentially deciding whether that side of the robot is currently
"on the line" or not. This was one of the circuits I built and
understand in depth.

### Dual 555 timer flip-flops — per-side turning logic

Two 555 timer ICs, each wired as a bistable (flip-flop) circuit, hold
the turning state for the left and right sides of the robot
independently — referencing the standard 555 bistable configuration
([reference](https://electronzap.com/electronic-components-list-and-links/555-timer-ic-integrated-circuit-electronic-component/)).
This is the second circuit I built myself, and the one I'm most
confident explaining end-to-end.

### Voltage boost circuit

The sensing circuit's output voltage was too small to reliably drive
the rest of the system, so the team built a boost circuit to draw
additional voltage from the battery pack. This circuit took the
longest to get working — about two weeks — and was a collaborative
build with a teammate, so I have a solid working understanding of
what it does and why it was needed, though I'm less confident
walking through its exact component-level design than the two
circuits above.

*Note: exact resistor/capacitor values from this project weren't
preserved in my own notes, so they're left out here rather than
guessed at.*

---

## Video Demonstration

<iframe width="100%" height="500" src="https://www.youtube.com/embed/nLQ0FiSh2AY" title="Analog line follower robot demonstration" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

---

## What I Learned

Building the sensing and turning-decision circuits myself gave me a
solid feel for how a physical analog system makes what looks like a
"decision" without any code at all — it's really just voltage
thresholds and timing propagating through discrete components. The
voltage boost circuit was the harder lesson: getting a supporting
circuit to reliably deliver enough voltage to drive the rest of the
system took much longer than the sensing logic itself, which was a
good reminder that power delivery is often the least glamorous and
most failure-prone part of an analog build.

## Skills Demonstrated

- IR reflectance sensing and phototransistor-based detection
- Comparator circuit design for analog-to-digital decision logic
- 555 timer bistable (flip-flop) circuit design
- Potentiometer-based sensor tuning
- Collaborative debugging on a multi-stage analog power circuit

---

[← Back to projects](https://noonereadsusernames.github.io/JulianaYasis/)

This project is maintained by [NoOneReadsUsernames](https://github.com/NoOneReadsUsernames)

Hosted on GitHub Pages — Theme by [orderedlist](https://github.com/orderedlist)
