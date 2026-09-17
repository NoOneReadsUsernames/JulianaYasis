// ============================================================
//  12x8 Green LED Matrix — mbed OS 6 / LPC1768
//  Anode columns (PNP, active LOW) x12
//  Cathode rows  (NPN, active HIGH) x8
// ============================================================

#include "mbed.h"

// ------------------------------------------------------------
// Pin assignments — adjust to your wiring
// ------------------------------------------------------------
DigitalOut colPins[12] = {
    DigitalOut(p5),  DigitalOut(p6),  DigitalOut(p7),  DigitalOut(p8),
    DigitalOut(p9),  DigitalOut(p10), DigitalOut(p11), DigitalOut(p12),
    DigitalOut(p13), DigitalOut(p14), DigitalOut(p15), DigitalOut(p16)
};

DigitalOut rowPins[8] = {
    DigitalOut(p17), DigitalOut(p18), DigitalOut(p19), DigitalOut(p20),
    DigitalOut(p21), DigitalOut(p22), DigitalOut(p23), DigitalOut(p24)
};

// ------------------------------------------------------------
// Frame buffer  [row 0..7][col 0..11]   1 = ON, 0 = OFF
// ------------------------------------------------------------
volatile uint8_t framebuffer[8][12] = {0};

// ------------------------------------------------------------
// Display ISR — called by Ticker every 1 ms
// Scans one row per call → full frame every 8 ms (~125 Hz)
// ------------------------------------------------------------
int activeRow = 0;

void refreshISR() {
    // 1. Blank all rows first (prevents ghosting)
    for (int r = 0; r < 8; r++) rowPins[r] = 0;

    // 2. Set column data for the active row
    //    PNP is active LOW: 0 = LED on, 1 = LED off
    for (int c = 0; c < 12; c++) {
        colPins[c] = framebuffer[activeRow][c] ? 0 : 1;
    }

    // 3. Enable the active row (NPN active HIGH)
    rowPins[activeRow] = 1;

    activeRow = (activeRow + 1) % 8;
}

// ------------------------------------------------------------
// Helper: copy a symbol into the frame buffer
// ------------------------------------------------------------
void loadSymbol(const uint8_t sym[8][12]) {
    for (int r = 0; r < 8; r++)
        for (int c = 0; c < 12; c++)
            framebuffer[r][c] = sym[r][c];
}

// ------------------------------------------------------------
// Helper: clear the frame buffer
// ------------------------------------------------------------
void clearDisplay() {
    for (int r = 0; r < 8; r++)
        for (int c = 0; c < 12; c++)
            framebuffer[r][c] = 0;
}

// ============================================================
//  SYMBOL DEFINITIONS  (8 rows × 12 cols)
// ============================================================

const uint8_t SYM_HEART[8][12] = {
    {0,1,1,0,0,0,0,1,1,0,0,0},
    {1,1,1,1,0,0,1,1,1,1,0,0},
    {1,1,1,1,1,1,1,1,1,1,1,0},
    {1,1,1,1,1,1,1,1,1,1,1,0},
    {0,1,1,1,1,1,1,1,1,1,0,0},
    {0,0,1,1,1,1,1,1,1,0,0,0},
    {0,0,0,1,1,1,1,1,0,0,0,0},
    {0,0,0,0,1,1,1,0,0,0,0,0},
};

const uint8_t SYM_SMILEY[8][12] = {
    {0,0,1,1,1,1,1,1,1,1,0,0},
    {0,1,0,0,0,0,0,0,0,0,1,0},
    {1,0,0,1,1,0,0,1,1,0,0,1},
    {1,0,0,1,1,0,0,1,1,0,0,1},
    {1,0,0,0,0,0,0,0,0,0,0,1},
    {1,0,1,0,0,0,0,0,0,1,0,1},
    {0,1,0,1,1,1,1,1,1,0,1,0},
    {0,0,1,1,1,1,1,1,1,1,0,0},
};

const uint8_t SYM_ARROW_RIGHT[8][12] = {
    {0,0,0,0,0,1,0,0,0,0,0,0},
    {0,0,0,0,0,1,1,0,0,0,0,0},
    {1,1,1,1,1,1,1,1,0,0,0,0},
    {1,1,1,1,1,1,1,1,1,1,1,0},
    {1,1,1,1,1,1,1,1,1,1,1,0},
    {1,1,1,1,1,1,1,1,0,0,0,0},
    {0,0,0,0,0,1,1,0,0,0,0,0},
    {0,0,0,0,0,1,0,0,0,0,0,0},
};

const uint8_t SYM_CROSS[8][12] = {
    {1,1,0,0,0,0,0,0,0,0,1,1},
    {1,1,1,0,0,0,0,0,0,1,1,1},
    {0,1,1,1,0,0,0,0,1,1,1,0},
    {0,0,1,1,1,0,0,1,1,1,0,0},
    {0,0,1,1,1,0,0,1,1,1,0,0},
    {0,1,1,1,0,0,0,0,1,1,1,0},
    {1,1,1,0,0,0,0,0,0,1,1,1},
    {1,1,0,0,0,0,0,0,0,0,1,1},
};

const uint8_t SYM_CHECKMARK[8][12] = {
    {0,0,0,0,0,0,0,0,0,0,1,1},
    {0,0,0,0,0,0,0,0,0,1,1,0},
    {0,0,0,0,0,0,0,0,1,1,0,0},
    {1,1,0,0,0,0,0,1,1,0,0,0},
    {1,1,1,0,0,0,1,1,0,0,0,0},
    {0,1,1,1,0,1,1,0,0,0,0,0},
    {0,0,1,1,1,1,0,0,0,0,0,0},
    {0,0,0,1,1,0,0,0,0,0,0,0},
};

const uint8_t SYM_DIAMOND[8][12] = {
    {0,0,0,0,0,1,1,0,0,0,0,0},
    {0,0,0,0,1,1,1,1,0,0,0,0},
    {0,0,0,1,1,1,1,1,1,0,0,0},
    {0,0,1,1,1,1,1,1,1,1,0,0},
    {0,0,1,1,1,1,1,1,1,1,0,0},
    {0,0,0,1,1,1,1,1,1,0,0,0},
    {0,0,0,0,1,1,1,1,0,0,0,0},
    {0,0,0,0,0,1,1,0,0,0,0,0},
};

// ============================================================
//  FONT: 5×8 pixel font (printable ASCII 0x20–0x7E)
//  Each character is stored as 5 column bytes (top=MSB).
//  A 1-pixel gap column is added automatically during scroll.
// ============================================================
// fmt: 5 bytes per char, each byte = 8 vertical pixels (bit7=top)
static const uint8_t FONT5x8[][5] = {
    {0x00,0x00,0x00,0x00,0x00}, // ' ' (0x20)
    {0x00,0x00,0x5F,0x00,0x00}, // '!'
    {0x00,0x07,0x00,0x07,0x00}, // '"'
    {0x14,0x7F,0x14,0x7F,0x14}, // '#'
    {0x24,0x2A,0x7F,0x2A,0x12}, // '$'
    {0x23,0x13,0x08,0x64,0x62}, // '%'
    {0x36,0x49,0x55,0x22,0x50}, // '&'
    {0x00,0x05,0x03,0x00,0x00}, // '''
    {0x00,0x1C,0x22,0x41,0x00}, // '('
    {0x00,0x41,0x22,0x1C,0x00}, // ')'
    {0x14,0x08,0x3E,0x08,0x14}, // '*'
    {0x08,0x08,0x3E,0x08,0x08}, // '+'
    {0x00,0x50,0x30,0x00,0x00}, // ','
    {0x08,0x08,0x08,0x08,0x08}, // '-'
    {0x00,0x60,0x60,0x00,0x00}, // '.'
    {0x20,0x10,0x08,0x04,0x02}, // '/'
    {0x3E,0x51,0x49,0x45,0x3E}, // '0'
    {0x00,0x42,0x7F,0x40,0x00}, // '1'
    {0x42,0x61,0x51,0x49,0x46}, // '2'
    {0x21,0x41,0x45,0x4B,0x31}, // '3'
    {0x18,0x14,0x12,0x7F,0x10}, // '4'
    {0x27,0x45,0x45,0x45,0x39}, // '5'
    {0x3C,0x4A,0x49,0x49,0x30}, // '6'
    {0x01,0x71,0x09,0x05,0x03}, // '7'
    {0x36,0x49,0x49,0x49,0x36}, // '8'
    {0x06,0x49,0x49,0x29,0x1E}, // '9'
    {0x00,0x36,0x36,0x00,0x00}, // ':'
    {0x00,0x56,0x36,0x00,0x00}, // ';'
    {0x08,0x14,0x22,0x41,0x00}, // '<'
    {0x14,0x14,0x14,0x14,0x14}, // '='
    {0x00,0x41,0x22,0x14,0x08}, // '>'
    {0x02,0x01,0x51,0x09,0x06}, // '?'
    {0x32,0x49,0x79,0x41,0x3E}, // '@'
    {0x7E,0x11,0x11,0x11,0x7E}, // 'A'
    {0x7F,0x49,0x49,0x49,0x36}, // 'B'
    {0x3E,0x41,0x41,0x41,0x22}, // 'C'
    {0x7F,0x41,0x41,0x22,0x1C}, // 'D'
    {0x7F,0x49,0x49,0x49,0x41}, // 'E'
    {0x7F,0x09,0x09,0x09,0x01}, // 'F'
    {0x3E,0x41,0x49,0x49,0x7A}, // 'G'
    {0x7F,0x08,0x08,0x08,0x7F}, // 'H'
    {0x00,0x41,0x7F,0x41,0x00}, // 'I'
    {0x20,0x40,0x41,0x3F,0x01}, // 'J'
    {0x7F,0x08,0x14,0x22,0x41}, // 'K'
    {0x7F,0x40,0x40,0x40,0x40}, // 'L'
    {0x7F,0x02,0x04,0x02,0x7F}, // 'M'
    {0x7F,0x04,0x08,0x10,0x7F}, // 'N'
    {0x3E,0x41,0x41,0x41,0x3E}, // 'O'
    {0x7F,0x09,0x09,0x09,0x06}, // 'P'
    {0x3E,0x41,0x51,0x21,0x5E}, // 'Q'
    {0x7F,0x09,0x19,0x29,0x46}, // 'R'
    {0x46,0x49,0x49,0x49,0x31}, // 'S'
    {0x01,0x01,0x7F,0x01,0x01}, // 'T'
    {0x3F,0x40,0x40,0x40,0x3F}, // 'U'
    {0x1F,0x20,0x40,0x20,0x1F}, // 'V'
    {0x3F,0x40,0x38,0x40,0x3F}, // 'W'
    {0x63,0x14,0x08,0x14,0x63}, // 'X'
    {0x07,0x08,0x70,0x08,0x07}, // 'Y'
    {0x61,0x51,0x49,0x45,0x43}, // 'Z'
    {0x00,0x7F,0x41,0x41,0x00}, // '['
    {0x02,0x04,0x08,0x10,0x20}, // '\'
    {0x00,0x41,0x41,0x7F,0x00}, // ']'
    {0x04,0x02,0x01,0x02,0x04}, // '^'
    {0x40,0x40,0x40,0x40,0x40}, // '_'
    {0x00,0x01,0x02,0x04,0x00}, // '`'
    {0x20,0x54,0x54,0x54,0x78}, // 'a'
    {0x7F,0x48,0x44,0x44,0x38}, // 'b'
    {0x38,0x44,0x44,0x44,0x20}, // 'c'
    {0x38,0x44,0x44,0x48,0x7F}, // 'd'
    {0x38,0x54,0x54,0x54,0x18}, // 'e'
    {0x08,0x7E,0x09,0x01,0x02}, // 'f'
    {0x0C,0x52,0x52,0x52,0x3E}, // 'g'
    {0x7F,0x08,0x04,0x04,0x78}, // 'h'
    {0x00,0x44,0x7D,0x40,0x00}, // 'i'
    {0x20,0x40,0x44,0x3D,0x00}, // 'j'
    {0x7F,0x10,0x28,0x44,0x00}, // 'k'
    {0x00,0x41,0x7F,0x40,0x00}, // 'l'
    {0x7C,0x04,0x18,0x04,0x78}, // 'm'
    {0x7C,0x08,0x04,0x04,0x78}, // 'n'
    {0x38,0x44,0x44,0x44,0x38}, // 'o'
    {0x7C,0x14,0x14,0x14,0x08}, // 'p'
    {0x08,0x14,0x14,0x18,0x7C}, // 'q'
    {0x7C,0x08,0x04,0x04,0x08}, // 'r'
    {0x48,0x54,0x54,0x54,0x20}, // 's'
    {0x04,0x3F,0x44,0x40,0x20}, // 't'
    {0x3C,0x40,0x40,0x40,0x7C}, // 'u'
    {0x1C,0x20,0x40,0x20,0x1C}, // 'v'
    {0x3C,0x40,0x30,0x40,0x3C}, // 'w'
    {0x44,0x28,0x10,0x28,0x44}, // 'x'
    {0x0C,0x50,0x50,0x50,0x3C}, // 'y'
    {0x44,0x64,0x54,0x4C,0x44}, // 'z'
    {0x00,0x08,0x36,0x41,0x00}, // '{'
    {0x00,0x00,0x7F,0x00,0x00}, // '|'
    {0x00,0x41,0x36,0x08,0x00}, // '}'
    {0x10,0x08,0x08,0x10,0x08}, // '~'
};

// ------------------------------------------------------------
// Build a wide scroll buffer from a text string.
// Each char = 5 cols of pixel data + 1 blank gap col.
// Buffer is allocated on the heap — caller must free[].
// Returns the total column count via outCols.
// ------------------------------------------------------------
uint8_t* buildScrollBuffer(const char* text, int& outCols) {
    int len = strlen(text);
    outCols = len * 6;              // 5 data cols + 1 gap per char
    uint8_t* buf = new uint8_t[8 * outCols]();  // zero-initialised

    for (int ci = 0; ci < len; ci++) {
        char ch = text[ci];
        if (ch < 0x20 || ch > 0x7E) ch = 0x20;    // replace unknowns with space
        const uint8_t* glyph = FONT5x8[ch - 0x20];

        for (int col = 0; col < 5; col++) {
            uint8_t colByte = glyph[col];
            int bufCol = ci * 6 + col;
            for (int row = 0; row < 8; row++) {
                // bit7 = top row, bit0 = bottom row
                buf[row * outCols + bufCol] = (colByte >> (7 - row)) & 1;
            }
        }
        // gap col (col index ci*6+5) stays 0 from zero-init
    }
    return buf;
}

// ------------------------------------------------------------
// Scroll a pre-built buffer across the display.
// scrollDelay_ms: pause between each column shift (speed).
// The function blocks until the full string has scrolled off.
// ------------------------------------------------------------
void scrollText(const uint8_t* buf, int totalCols, int scrollDelay_ms) {
    // Start with text just off the right edge, scroll left
    for (int offset = -12; offset < totalCols; offset++) {
        for (int r = 0; r < 8; r++) {
            for (int c = 0; c < 12; c++) {
                int srcCol = offset + c;
                if (srcCol >= 0 && srcCol < totalCols)
                    framebuffer[r][c] = buf[r * totalCols + srcCol];
                else
                    framebuffer[r][c] = 0;
            }
        }
        ThisThread::sleep_for(scrollDelay_ms);
    }
}

// Convenience wrapper: build + scroll + free
void scrollString(const char* text, int scrollDelay_ms = 40) {
    int totalCols;
    uint8_t* buf = buildScrollBuffer(text, totalCols);
    scrollText(buf, totalCols, scrollDelay_ms);
    delete[] buf;
}

// ============================================================
//  ANIMATION HELPERS
// ============================================================

// Fade in: gradually illuminate row by row from the centre out
void animateFadeIn(const uint8_t sym[8][12], int stepDelay_ms = 60) {
    clearDisplay();
    // rows outward from centre: 3,4,2,5,1,6,0,7
    const int order[] = {3,4,2,5,1,6,0,7};
    for (int i = 0; i < 8; i++) {
        int r = order[i];
        for (int c = 0; c < 12; c++) framebuffer[r][c] = sym[r][c];
        ThisThread::sleep_for(stepDelay_ms);
    }
}

// Wipe in from left to right, one column at a time
void animateWipeLeft(const uint8_t sym[8][12], int stepDelay_ms = 30) {
    clearDisplay();
    for (int c = 0; c < 12; c++) {
        for (int r = 0; r < 8; r++) framebuffer[r][c] = sym[r][c];
        ThisThread::sleep_for(stepDelay_ms);
    }
}

// Blink a symbol N times
void blinkSymbol(const uint8_t sym[8][12], int times = 3, int onMs = 300, int offMs = 200) {
    for (int i = 0; i < times; i++) {
        loadSymbol(sym);
        ThisThread::sleep_for(onMs);
        clearDisplay();
        ThisThread::sleep_for(offMs);
    }
}

// ============================================================
//  MAIN
// ============================================================
int main() {
    // Initialise all pins to safe (off) state
    for (int r = 0; r < 8;  r++) rowPins[r] = 0;
    for (int c = 0; c < 12; c++) colPins[c] = 1;   // PNP off = HIGH

    // Start the display refresh ticker (every 1 ms)
    Ticker refreshTicker;
    refreshTicker.attach(&refreshISR, 1ms);

    // --------------------------------------------------------
    // Demo sequence — edit this section to suit your project
    // --------------------------------------------------------
    while (true) {

        // 1. Scroll a greeting message
        scrollString("HELLO WORLD! ", 40);

        // 2. Wipe in heart, hold, blink, clear
        animateWipeLeft(SYM_HEART, 30);
        ThisThread::sleep_for(1500ms);
        blinkSymbol(SYM_HEART, 3, 250, 150);
        clearDisplay();
        ThisThread::sleep_for(300ms);

        // 3. Fade in smiley, hold, clear
        animateFadeIn(SYM_SMILEY, 60);
        ThisThread::sleep_for(2000ms);
        clearDisplay();
        ThisThread::sleep_for(300ms);

        // 4. Cycle through remaining symbols with a wipe animation
        const uint8_t* symbols[] = {
            &SYM_ARROW_RIGHT[0][0],
            &SYM_CHECKMARK[0][0],
            &SYM_CROSS[0][0],
            &SYM_DIAMOND[0][0],
        };
        for (int i = 0; i < 4; i++) {
            animateWipeLeft((const uint8_t (*)[12]) symbols[i], 25);
            ThisThread::sleep_for(1200ms);
            clearDisplay();
            ThisThread::sleep_for(200ms);
        }

        // 5. Scroll a number string
        scrollString("1234567890 ", 40);
    }
}
