# Guitar Tuner (Enhanced YIN Algorithm)

A **web-based guitar tuner** that uses the **YIN pitch detection algorithm** for real-time **monophonic** note analysis. Originally written with [ChatGPT](https://openai.com/blog/chatgpt/), later redesigned and rewritten with [Claude](https://claude.ai). It provides:

1. **Accurate pitch detection** across the full guitar range (60–500 Hz), including the low E string around 82 Hz.
2. **Analog-style needle gauge** showing how many cents sharp or flat you are, with a large note display readable from across the room.
3. **String detection** that lights up the matched string (E2, A2, D3, G3, B3, E4) and marks it green once you hold it in tune for a second.
4. **Reference tones** — tap any string button to hear its target pitch.
5. **Audible confirmation** — a short beep when you're in tune (±5 cents), at most once every 2 seconds.

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
   - Each frame is analyzed with the **YIN algorithm**, which is more robust than naive autocorrelation. The search is restricted to the guitar's frequency range for speed, with parabolic interpolation for sub-sample accuracy.
   - Readings are smoothed with a **median filter** (resistant to outlier frames), and the last reading is held briefly after the note decays so the display doesn't flicker.

3. **Nearest String Calculation**
   - The detected frequency is compared against the six standard string frequencies: E2 (82.41 Hz), A2 (110 Hz), D3 (146.83 Hz), G3 (196 Hz), B3 (246.94 Hz), E4 (329.63 Hz).
   - Matching uses **logarithmic (cents) distance** rather than raw Hz, so high strings aren't unfairly matched to low ones.

4. **Cents Off & Feedback**
   - The gap between detected and target pitch is shown in **cents** (1 semitone = 100 cents) on a needle gauge clamped to ±50 cents.
   - Within ±5 cents: the note turns **green**, the hint reads "In tune ✓", and a short beep plays.
   - Outside that range: the hint tells you which way to adjust ("Too low — tighten the string ↑" / "Too high — loosen the string ↓").

5. **Per-String Progress**
   - Hold a string in tune for one second and its button gains a green dot, so you can track which strings are done.

---

## Usage Instructions

1. **Open the Live Demo** (or host the code yourself over HTTPS / `localhost`).
2. **Press Start tuner** and allow microphone access when prompted.
3. **Pluck one string at a time.** The tuner shows the nearest string, the detected frequency, and how far off you are.
4. **Adjust tuning** until the needle centers and the note turns green — you'll hear a confirmation beep.
5. **Repeat for all six strings.** Tap a string button anytime to hear its reference pitch.
6. **Press Stop tuner** when finished to release the microphone.

---

## Tweaks & Customizations

All constants live at the top of the script:

- **Tuning tolerance** — `IN_TUNE_CENTS` (default ±5) controls how strict "in tune" is.
- **Beep** — `BEEP_GAP_S` sets the minimum gap between beeps; frequency and envelope are in `beep()`.
- **Sensitivity** — lower `YIN_THRESHOLD` (default 0.15) for more sensitive detection (more false positives), raise it for stricter detection. `RMS_THRESHOLD` sets how loud the input must be before analysis runs.
- **Smoothing** — `SMOOTHING` sets the median window size; `HOLD_MS` controls how long the last reading stays on screen after the note decays.
- **Tuned indicator** — `TUNED_HOLD_MS` sets how long a string must stay in tune before it's marked done.
- **Detection range** — `MIN_FREQ` / `MAX_FREQ` bound the YIN search (useful if you adapt this for bass or alternate tunings; you'd also update the `STRINGS` table).

---

## Browser Support

Works in current Chrome, Edge, Firefox, and Safari. Requires `getUserMedia` and the Web Audio API, which means HTTPS (or `localhost`) is mandatory.
