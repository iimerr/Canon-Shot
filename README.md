# Canon Shot 🎯

A two-player, two-board competitive arcade shooting game built on the **Nuvoton NuMicro NUC140** learning board.

> RBB3023: Embedded and IoT Systems — May 2026 Semester, Universiti Teknologi PETRONAS
> Group: **Emir&Harith Enterprise.co**

## Overview

Each player controls a cannon that moves left and right along the bottom of a 128×64 graphic LCD and fires bullets upward to destroy five types of geometric shapes — **Star, Diamond, Circle, Triangle, and Cross** — that move unpredictably across the screen, each worth a different point value (10–300 points). Each round lasts 30 seconds, tracked live on the on-board 7-segment display.

The defining feature of the project is genuine **inter-board communication**: when a round ends, each board transmits its final score to the other over a UART serial link, and both boards independently compare scores to determine and display the winner. The outcome is a shared result between the two boards, not two independent single-player sessions.

## Hardware

| Component | Interface / Pins | Role |
|---|---|---|
| 128×64 Graphic LCD | On-board parallel LCD, `init_LCD()` | Main game display: cannon, bullets, shapes, score, game-state messages |
| 4-Digit 7-Segment Display | GPIO Port E (segments), Port C4–C7 (digit select) | Live 30-second round countdown, refreshed every frame |
| 3×3 Matrix Keypad | On-board GPIO keypad, `ScanKey()` | Key '2' fires a bullet; any key confirms title/result screens |
| Variable Resistor (VR1) | ADC Channel 7 (GPA, 12-bit, continuous scan) | Controls the cannon's left/right position on screen |
| Piezo Buzzer | GPIO Port B, Pin 11 | Audio beep feedback when a shape is destroyed |
| UART0 | Pin 32 (RX0) / Pin 33 (TX0), 9600 baud, 8N1 | Serial link between the two boards for score exchange |
| Timer1 | Internal hardware timer, HCLK source, 2 Hz interrupt | Drives the 30-second countdown independently of game-loop timing |

## Functional Requirement Coverage

| Spec Requirement | Implementation |
|---|---|
| GPIO as switch input (must) | 3×3 keypad read via `ScanKey()` controls firing and menu navigation |
| 7-segment / LCD display (must) | Both used simultaneously: LCD for the game field, 7-segment for the countdown |
| Timer | Timer1 hardware interrupt (2 Hz) drives the round countdown independently of frame rate |
| ADC | ADC Channel 7 (VR1) continuously read to position the cannon |
| Other (UART/SPI/I2C/CAN/WiFi) | UART0 implements a two-board score-exchange protocol at the end of every round |

## Software Flow (high level)

1. Initialize system clock, LCD, keypad, buzzer, Timer1, ADC7 (VR1), UART0.
2. Show title/instructions screen — wait for any keypress.
3. Reset score, start 30 s countdown, init shape positions and UART state.
4. **Main game loop**: draw cannon/bullets/shapes → refresh 7-segment countdown → read ADC for cannon position → check fire key → move bullets/shapes → check collisions/scoring → repeat until all shapes cleared or timer hits 0.
5. Show "YOU WON" / "TIME'S UP", then repeatedly transmit own score over UART0 while listening for the opponent's score; display WIN / LOSE / TIE once both scores are known.
6. Wait for keypress → reset round state → back to main loop.

## Two-Board Communication Protocol

At the end of a round, each board formats its score as an 8-byte ASCII packet: `SCddddd` (2-character prefix + 5-digit zero-padded score), and transmits it repeatedly over UART0 while showing "Comparing scores…". A UART receive interrupt on the other board buffers and parses incoming packets as they arrive. There is no fixed timeout — each board keeps re-announcing its score and re-checking for the opponent's until a reply is received, updating the result live.

## How to Play

### Setup
1. Power on both NUC140 boards.
2. Cross-connect UART0 lines: Board A TX0 (pin 33) → Board B RX0 (pin 32), and Board A RX0 (pin 32) → Board B TX0 (pin 33). Connect a common ground between boards.
3. Each board's LCD shows the title screen with instructions.

### Controls
- Turn **VR1** left/right to move the cannon horizontally.
- Press keypad key **'2'** to fire a bullet upward.
- Press **any key** to dismiss the title screen and the end-of-round result screen.

### Objective
Destroy as many shapes as possible in 30 seconds. Shape values: Star (300), Diamond (30), Circle (25), Triangle (20), Cross (10). A round ends early if all shapes are cleared, or automatically when the countdown hits zero.

### End of Round
1. LCD shows "YOU WON" or "TIME'S UP".
2. Both boards exchange final scores over UART0 and display: "YOU WIN THE MATCH!", "OPPONENT WINS THE MATCH!", or "IT'S A TIE!".
3. Press any key to start a new round — all state resets automatically.

## Known Limitations & Future Improvements

- Only a single ADC channel (VR1) is used; a second potentiometer could add an independent fire-power or difficulty control.
- The UART protocol assumes a direct wired link between exactly two boards; no support for more than two players.
- Shape movement is randomized per-frame but doesn't scale in difficulty over a round; a future version could increase shape speed as the countdown runs low.

## Repository Contents

> 🚧 Source code to be added.

```
canon-shot/
├── README.md
└── src/            <- firmware source (to be added)
```

## Demo Video

[_Link to be added._](https://www.youtube.com/watch?si=MekqNhcINxoYTO1x&v=WZRVseMsXPs&feature=youtu.be)

## Authors

- Muhammad Harith Iskandar Bin Mahathir — 24003426
- Emir Azimil Akbar Bin Mohd Fauzi — 24003510
