# Guitar Tuner (Enhanced YIN Algorithm)

This is a **web-based guitar tuner** that uses the **YIN pitch detection algorithm** for **monophonic** note analysis in real time. The code and interface were written by [ChatGPT](https://openai.com/blog/chatgpt/) and tweaked to provide:

1. **Accurate pitch detection** (including the low E string around 82 Hz).  
2. **Visual feedback**: color-coded readouts and a meter showing how many cents off you are from the nearest standard guitar string.  
3. **String detection**: clearly shows which string is recognized (E2, A2, etc.).  
4. **Optional beep** when you’re in tune (±5 cents) to give audible feedback.

---

## Live Demo

Access the live demo here:  
[**https://guitar-tuner-online.pages.dev/**](https://guitar-tuner-online.pages.dev/)

---

## How It Works

1. **Microphone Access**  
   - The app requests access to your microphone through the browser.  
   - Make sure you run it over **HTTPS** or on `localhost` (the provided demo site is served via HTTPS).

2. **ScriptProcessor + YIN**  
   - We create a **ScriptProcessorNode** that receives small audio buffers from the mic.  
   - Each audio frame is analyzed using the **YIN algorithm**, which is generally more robust than naive autocorrelation.  
   - YIN computes the fundamental frequency of the note being played.

3. **Nearest String Calculation**  
   - We compare the detected frequency with the six standard guitar string frequencies:
     - E2 (82.41 Hz), A2 (110 Hz), D3 (146.83 Hz), G3 (196 Hz), B3 (246.94 Hz), E4 (329.63 Hz).  
   - The tuner shows which of these strings you’re closest to.

4. **Cents Off & Color-Coded Display**  
   - The difference between your detected pitch and the target frequency is converted to **cents** (1 semitone = 100 cents).  
   - If you’re within ±5 cents, the text turns **green** and you’ll hear a **short beep** (once every 2 seconds).  
   - ±10 cents displays **yellow**, and anything more shows **red**, indicating you need further adjustment.

5. **Meter**  
   - The `<meter>` element visually represents how sharp or flat you are, clamped between –50 and +50 cents.

---

## Usage Instructions

1. **Open the Live Demo**  
   - Go to [**https://guitar-tuner-online.pages.dev/**](https://guitar-tuner-online.pages.dev/).  
   - Alternatively, download and host the code locally over HTTPS.

2. **Allow Microphone Access**  
   - When prompted, grant permission for the site to access your microphone.

3. **Pluck a String**  
   - Play one guitar string at a time.  
   - The tuner displays the **nearest string name** and how many cents off you are.

4. **Adjust Tuning**  
   - Tighten or loosen the string until the **meter** is near the center (0 cents) and the text turns **green**.  
   - A short beep indicates you’re in an acceptable range.

5. **Repeat for All Strings**  
   - Tune each string (E, A, D, G, B, E) until your guitar is fully in tune.

---

## Tweaks & Customizations

- **Beep Settings**  
  - Change the beep’s frequency (`BEEP_FREQUENCY`) or duration (`BEEP_DURATION`) in the code.  
  - Adjust `BEEP_DEBOUNCE_SECONDS` to control how often the beep can repeat.

- **Sensitivity**  
  - `YIN_THRESHOLD` can be raised or lowered if you see too many false positives (lower means more sensitive).  
  - `RMS_THRESHOLD
