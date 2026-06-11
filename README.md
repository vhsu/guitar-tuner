# Guitar Tuner (Enhanced YIN Algorithm)

A **web-based guitar tuner** that uses the **YIN pitch detection algorithm** for real-time **monophonic** note analysis. Originally written with [ChatGPT](https://openai.com/blog/chatgpt/), later redesigned and rewritten with [Claude](https://claude.ai). It provides:

1. **Accurate pitch detection** across the full guitar range, including the low E string around 82 Hz — verified against synthetic harmonic tones to within fractions of a cent.
2. **Six tunings** — Standard, Drop D, E♭ Standard (half-step down), DADGAD, Open G, and Open D.
3. **Chromatic mode** — detect any note rather than snapping to the nearest string. Useful for capos, intonation checks, bass, or unusual tunings.
4. **String lock** — tap a string to lock the tuner onto it, so a badly flat low E isn't mistaken for a nearly-in-tune D.
5. **Adjustable reference pitch** — calibrate A4 anywhere from 432 to 446 Hz (default 440).
6. **Analog-style needle gauge** showing how many cents sharp or flat you are, with a large note display readable from across the room.
7. **Per-string progress** — a string's lamp turns green once you hold it in tune for a second.
8. **Reference tones** — tap any string button to hear its target pitch at the current calibration.
9. **Audible confirmation** — a short beep when you're in tune (±5 cents), with a mute toggle for phone speakers.
10. **Screen wake lock** — the display stays awake while the tuner runs (on supporting browsers).

---

## Live Demo

[**https://guitar-tuner-online.pages.dev/**](https://guitar-tuner-online.pages.dev/)

---

## How It Works

1. **Microphone Access**
   - The app requests microphone access through the browser, with echo cancellation, noise suppression, and auto gain disabled (these otherwise distort guitar signals).
   - Browsers only allow mic access over **HTTPS** or on `localhost` (the demo site is served via HTTPS).

2. **AnalyserNode + YIN**
   - Audio is read from an **`AnalyserNode`** (8192-sample frames) inside a `requestAnimationFrame` loop — no deprecated `ScriptProcessorNode`.
   - Each frame is analyzed with the **YIN algorithm**, which is more robust than naive autocorrelation. The search is restricted to the relevant frequency range for speed, with parabolic interpolation for sub-sample accuracy. (This version also fixes a subtle bug in earlier YIN implementations of this project: the dip walk-down compared against not-yet-normalized values, biasing every reading slightly sharp.)
   - Readings are smoothed with a **median filter** (resistant to outlier frames), and the last reading is held briefly after the note decays so the display doesn't flicker.

3. **Target Selection**
   - All pitches are derived from MIDI note numbers and the adjustable A4 reference, so every tuning and the chromatic scale stay correct at any calibration.
   - In **Strings** mode the detected frequency is matched to the nearest string of the selected tuning using **logarithmic (cents) distance** rather than raw Hz, so high strings aren't unfairly matched to low ones. Locking a string overrides auto-detection entirely.
   - In **Chromatic** mode the target is simply the nearest semitone.

4. **Cents Off & Feedback**
   - The gap between detected and target pitch is shown in **cents** (1 semitone = 100 cents) on a needle gauge clamped to ±50 cents.
   - Within ±5 cents: the note turns **green**, the hint reads "In tune ✓", and a short beep plays.
   - Outside that range: the hint tells you which way to adjust ("Too low — tighten the string ↑" / "Too high — loosen the string ↓").

5. **Per-String Progress**
   - Hold a string in tune for one second and its button gains a green dot, so you can track which strings are done.

---

## Usage Instructions

1. **Open the Live Demo** (or host the code yourself over HTTPS / `localhost`).
2. **Pick your setup** — choose a tuning, switch between Strings and Chromatic mode, adjust the A4 reference if needed, and mute the beep if you're on phone speakers.
3. **Press Start tuner** and allow microphone access when prompted.
4. **Pluck one string at a time.** The tuner shows the nearest target, the detected frequency, and how far off you are. If a string is so far off that auto-detect grabs the wrong target, tap its button to lock onto it.
5. **Adjust tuning** until the needle centers and the note turns green — hold it there a second and the string's lamp turns green.
6. **Repeat for all six strings**, then press **Stop tuner** to release the microphone.

---

## Tweaks & Customizations

All constants live at the top of the script:

- **Tunings** — add or edit entries in the `TUNINGS` table; each is just a label plus six MIDI note numbers, so frequencies follow the calibration automatically.
- **Tuning tolerance** — `IN_TUNE_CENTS` (default ±5) controls how strict "in tune" is.
- **Calibration range** — `A4_MIN` / `A4_MAX` bound the reference-pitch stepper.
- **Beep** — `BEEP_GAP_S` sets the minimum gap between beeps; frequency and envelope are in `beep()`.
- **Sensitivity** — lower `YIN_THRESHOLD` (default 0.15) for more sensitive detection (more false positives), raise it for stricter detection. `RMS_THRESHOLD` sets how loud the input must be before analysis runs.
- **Smoothing** — `SMOOTHING` sets the median window size; `HOLD_MS` controls how long the last reading stays on screen after the note decays; `DETECT_GAP_MS` throttles how often detection runs.
- **Tuned indicator** — `TUNED_HOLD_MS` sets how long a string must stay in tune before it's marked done.

---

## Browser Support

Works in current Chrome, Edge, Firefox, and Safari. Requires `getUserMedia` and the Web Audio API, which means HTTPS (or `localhost`) is mandatory.
