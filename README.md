# Typedrummer — Documentation

Version: 1.0  
Source: typedrummer.html

## Overview / Purpose
Mini Typedrummer maps typed letters to synthesized percussion and sequences typed text as rhythmic patterns. It uses only the Web Audio API (no audio samples). Intended as a lightweight, editable synth-driven drum/sequencer demo.

## Project layout
Single-file app:
- typedrummer.html — UI, styles and full JavaScript implementation (AudioContext, instruments, mapping, playback, UI).

This Markdown documents the code, behavior, extension points, troubleshooting and implementation notes.

## High-level structure
- CSS: visual layout (left editor + controls, right mapping panel).
- HTML: textarea for typing, mapping grid, controls (BPM, volume, kit select), buttons.
- JS:
  - Audio setup and helpers
  - Instrument implementations (Electronic / World / Acoustic)
  - Registry and kits
  - Letter → instrument mapping and UI rendering
  - Immediate keydown triggers
  - Playback engine (steps through textarea characters at a musical interval)
  - Shortcuts and small UX helpers

## Core audio setup
- ensureAudio()
  - Creates AudioContext and masterGain, connects masterGain → ctx.destination.
  - Reads initial volume from #volume slider.
- now()
  - Returns ctx.currentTime (used for scheduling).
- User gesture unlock
  - A click on document body calls ensureAudio() and resumes suspended context (modern browsers require user gestures).

Volume control:
- HTML input#volume changes masterGain.gain in real time.

## Utilities
- makeNoise(duration)
  - Creates mono AudioBuffer filled with white noise for transient & sustained noise-style instruments.
- Small helper functions to centralize time and audio creation patterns.

## Instruments
Three families implemented (all generate synthesized sound via OscillatorNode, GainNode, BiquadFilter, AudioBufferSourceNode):

1. Electronic / FX
   - fxHat, fxShaker, laser, zap, blip, noiseHit, glitch, subBoom, reverseNoise, percTick, etc.
   - Mix of short noise bursts, bandpass/highpass filtering, oscillators with pitch sweeps.

2. World percussion approximations
   - tablaHi/tablaLo, djembe, agogoHi/agogoLo, guiroUp/guiroDown, blockHi/blockLo, cowbell, clave, timbale, castanet, bongoHi/bongoLo.

3. Acoustic (synth-based drum kit)
   - acKick, acSnare, acHatClosed/acHatOpen, acRide, acCrash, acTomLow/acTomMid/acTomHigh, acRim, acWood, acTamb, acShake, acTri, acCow.
   - These use sine/triangle/square oscillators and noise buffers to emulate drum/tom/snare characteristics.

Each instrument:
- Ensures ctx exists (guard).
- Creates nodes, sets frequencies/gain/time envelopes.
- Connects to masterGain, starts and stops sources promptly.

## Instrument registry & kits
- instruments: object mapping string keys → function references.
- kits: two arrays (`electronic`, `acoustic`) listing instrument keys in order.
- remapLetters()
  - Cyclically maps letters "a"–"z" to instruments from the selected kit.
  - Updates letterMap (letter → instrument key).
- renderMap()
  - Renders a two-column grid showing letter → instrument.

Note: Some registry keys have slightly different names/prefixes (e.g., `'ackick'` maps to `acKick`). The registry normalizes lookup to available functions.

## Text interaction & UI
- #typeArea (textarea)
  - Input updates overlay via renderOverlay() which converts the text into per-character spans for highlighting during playback.
- Keydown (immediate trigger)
  - On keydown while focused, printable letters (length 1) are converted to lowercase, looked up in letterMap and immediately trigger the mapped instrument (fn()).

## Playback engine
- schedulePlayback()
  - Ensures audio is available, stops existing playback, then starts a step loop via setTimeout.
  - Steps through textarea characters in order. For each step:
    - Convert char → instrument → call instrument function.
    - Update overlay to highlight current index.
    - Compute next interval: (60 / BPM) * 0.25 → a sixteenth-note step at chosen BPM.
  - Uses setTimeout repeatedly (playTimer).
- stopPlayback()
  - Clears timer, resets state and overlay.

Controls & keyboard shortcuts:
- Buttons: Play, Stop, Clear.
- Shortcuts:
  - Ctrl + Enter => Play
  - Escape => Stop
  - Ctrl + Backspace => Clear
- BPM slider updates #bpmVal display.

## Extending the app
1. Add new instrument
   - Implement function fn() that uses ctx nodes, connects to masterGain, schedules envelope, and stops sources.
   - Add `instruments['myKey'] = myFunction;`
   - Add the key to a kit in `kits` to include it in remapping.

2. Add new kit
   - Add an array to `kits`:
     kits.myKit = ['instA','instB', ...];
   - Call remapLetters() or select via UI.

3. Change mapping strategy
   - remapLetters currently cycles kit instruments across letters; modify as needed to provide fixed maps or per-letter selection UI.

4. Improve scheduling
   - For tighter timing, convert the setTimeout loop to a lookahead scheduler using ctx.currentTime and periodic scheduling windows (preferred for accurate audio timing).

## Performance & safety
- Stop oscillators / buffer sources shortly after start to avoid leaks.
- Keep noise buffers reasonably short.
- Avoid creating high numbers of long-running nodes per interval.
- For complex sequences, schedule nodes ahead of time (AudioContext time) instead of relying on setTimeout for sample-accurate timing.

## Troubleshooting
- No audio:
  - Ensure you clicked page once to unlock/resume AudioContext.
  - Check console for exceptions (e.g., ctx undefined).
- Timing jitter:
  - Browser timers (setTimeout) can be imprecise. Use a lookahead scheduler for professional timing.
- High CPU:
  - Reduce polyphony, shorter buffers, or simpler envelopes.

## Browser compatibility
- Requires Web Audio API (AudioContext, OscillatorNode, AudioBufferSourceNode, BiquadFilterNode).
- Modern Chromium, Firefox and Safari are supported. Mobile browsers require user gesture for AudioContext startup.

## API surface (internal)
- ensureAudio(): init ctx & masterGain
- makeNoise(duration): AudioBuffer
- instruments: { key: function } — call functions to play
- remapLetters(), renderMap(), renderOverlay(idx)
- schedulePlayback(), stopPlayback()

## Quick usage examples
- Type "the quick brown fox" in textarea, then:
  - Click once anywhere (unlock audio), press Ctrl+Enter to play the sequence.
  - Press individual letters while focused to trigger instantaneous hits.
  - Switch kits to change instrument mapping instantly.

## File map
- typedrummer.html — single-file app (UI + JS + CSS)
- typedrummer.md — this documentation (suggested)

## License & attribution
This demo synthesizes percussion for educational/creative use. Confirm reuse/redistribution terms before commercial use.

## Minimal changelog / notes
- Instruments are implemented as concise JS functions using Web Audio primitives.
- Mapping is cyclical; letter→instrument mapping is recomputed when kit changes.

---
