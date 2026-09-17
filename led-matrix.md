# LPC1768 12×8 Green LED Matrix Display

**Embedded Systems • mbed OS 6 • LPC1768 • C • LED Matrix**

## Overview

This project implements a multiplexed 12×8 LED matrix display driven by an mbed LPC1768 microcontroller. The display supports static symbol rendering, transition animations, and real-time scrolling text using a full 5×8 ASCII font — all running on bare-metal embedded C++ with mbed OS 6.

## Technologies

- LPC1768
- C
- Embedded systems
- LED matrix
- GPIO

---

## Implementation

This project implements a multiplexed 12×8 LED matrix display driven by an mbed LPC1768 microcontroller. The display supports static symbol rendering, transition animations, and real-time scrolling text using a full 5×8 ASCII font — all running on bare-metal embedded C++ with mbed OS 6.

---

### Hardware Design

#### Matrix Architecture
The matrix uses a **column-anode, row-cathode** configuration with multiplexed scanning. Rather than driving each of the 96 LEDs individually (which would require 96 GPIO pins), the matrix is wired so that:

- **12 columns** share a common anode line each (driven by PNP transistors)
- **8 rows** share a common cathode line each (driven by NPN transistors)

Only one row is active at any given time. By cycling through all 8 rows fast enough (~125 Hz total), persistence of vision makes all LEDs appear continuously lit.

#### Transistor Driver Circuit
The LPC1768 GPIO pins can only source/sink ~4 mA — insufficient to drive a full LED row or column. Two transistor stages are used:

| Side | Component | Logic | Purpose |
|------|-----------|-------|---------|
| Column (anode) | PNP transistor (2N2907) + 1 kΩ base resistor | Active LOW | Sources current into the column |
| Row (cathode) | NPN transistor (2N2222) + 1 kΩ base resistor | Active HIGH | Sinks current from the row to GND |

A **68 Ω current-limiting resistor** on each column anode limits LED current to a safe level. The lower resistance (compared to a static ~120 Ω calculation) compensates for the reduced duty cycle introduced by multiplexing (1/8 on-time per row).

#### Parts List

| Component | Quantity |
|-----------|----------|
| mbed LPC1768 | 1 |
| Green LEDs | 96 |
| NPN transistor (2N2222 or equivalent) | 8 |
| PNP transistor (2N2907 or equivalent) | 12 |
| 68 Ω resistor (column current limiting) | 12 |
| 1 kΩ resistor (transistor base) | 20 |
| 100 nF decoupling capacitor | Per IC |
| 3.3 V power supply | 1 |

---

### Firmware Architecture

#### Display Refresh — Ticker ISR
The display is refreshed using an mbed `Ticker` interrupt that fires every **1 ms**. Each ISR call:

1. Blanks all row pins (prevents ghosting during column switching)
2. Writes the current row's pixel data to the column pins (PNP active LOW)
3. Enables the active row pin (NPN active HIGH)
4. Advances to the next row

This yields a **~125 Hz full-frame refresh rate** (8 rows × 1 ms/row), which runs entirely in the background via interrupt — leaving the main thread free for animation and display logic.

```cpp
void refreshISR() {
    for (int r = 0; r < 8; r++) rowPins[r] = 0;          // blank all rows
    for (int c = 0; c < 12; c++)
        colPins[c] = framebuffer[activeRow][c] ? 0 : 1;  // PNP: LOW = on
    rowPins[activeRow] = 1;                                // enable row
    activeRow = (activeRow + 1) % 8;
}
```

#### Frame Buffer
All display content is written to a shared `volatile uint8_t framebuffer[8][12]` array. Each cell holds `1` (LED on) or `0` (LED off). The ISR reads from this buffer every millisecond, so any write to the buffer is reflected on the display within one full scan cycle (~8 ms).

#### Scrolling Text — 5×8 ASCII Font
A full printable ASCII font (characters 0x20–0x7E) is stored as a lookup table of 5-byte column bitmaps. Each byte encodes 8 vertical pixels, with bit 7 at the top.

Scrolling works by:
1. Expanding the input string into a wide pixel buffer (5 data columns + 1 gap column per character)
2. Shifting a 12-column window across the buffer one column per step
3. Writing each window position into the framebuffer

Scroll speed is controlled by the delay between column shifts (default 40 ms/step).

```cpp
// Scroll any ASCII string across the display
scrollString("HELLO WORLD!", 40);   // 40 ms per column step
```

#### Symbol Display
Symbols are defined as `uint8_t[8][12]` pixel maps — readable at a glance in source code as a grid of 1s and 0s. Loading a symbol is a single function call:

```cpp
loadSymbol(SYM_HEART);
```

Built-in symbols include: heart, smiley face, right arrow, checkmark, cross, and diamond.

#### Animations
Three animation functions are provided for transitions between display states:

| Function | Effect |
|----------|--------|
| `animateWipeLeft()` | Reveals symbol column-by-column from left to right |
| `animateFadeIn()` | Fills rows outward from the vertical centre |
| `blinkSymbol()` | Flashes the current symbol N times with configurable on/off timing |

---

### Key Design Decisions

- **ISR-based refresh over polling** — Decouples display timing from application logic. Animation delays and `sleep_for()` calls in the main thread do not cause flicker.
- **Active-LOW column polarity** — Required by the PNP transistor topology. The firmware inverts framebuffer values before writing to column pins, keeping the framebuffer itself intuitive (`1` = on).
- **Row blanking before column switching** — Eliminates ghosting artifacts that occur when column data changes while a row is still enabled.
- **Heap-allocated scroll buffer** — The scroll buffer is dynamically allocated proportional to string length, then freed after scrolling completes, keeping static RAM usage low.

---

### Repository Structure

```
led-matrix-lpc1768/
├── src/
│   └── led_matrix.cpp      # Full firmware source
├── docs/
│   └── schematic.svg       # Circuit schematic (3×3 representative view)
└── README.md
```

---

### Skills Demonstrated

- Embedded C++ on bare-metal ARM Cortex-M3 (mbed OS 6)
- Hardware interrupt programming (`Ticker` ISR)
- Transistor-level driver circuit design (NPN/PNP switching)
- Memory-efficient font rendering and bitmap scrolling
- Multiplexed display timing and ghosting prevention

## Results

The completed system successfully displayed the programmed
patterns on the LED matrix.

## What I Learned

This project gave me experience with microcontroller I/O,
hardware/software interaction, and debugging embedded systems.

## Demonstration

[add working video here](images/It’s upside down gimme a bit to edit.mp4)

## Source Code

[GitHub repository link](ledmatrixcode.c)
