# Tape

A four-track recorder prototype for macOS. Record a musical idea, play it back and add another part in the same session.

**Built with:** C++ and JUCE · **Current form:** desktop prototype

## From an idea to a layered recording

1. **Record a part.** Capture a piano phrase, guitar line or another sound on one of four tracks.
2. **Play along.** Listen to the recording while adding another part. Recording onto an occupied track adds a layer alongside what is already there.
3. **Shape the mix.** Adjust track levels, mute parts, apply effects or merge tracks.
4. **Keep working.** Undo an edit, return to a saved session or export the audio.

The aim is a small recording workflow for building up musical ideas. The macOS app is also the starting point for a possible standalone recorder.

## What happens behind the controls

**The engine mixes the sound.** JUCE connects the app to the audio device and provides its interface. The engine supplies sound in small blocks, mixing the prepared playback with incoming audio.

**A separate worker handles the files.** Recording and playback data move through fixed-size buffers between the engine and storage worker. This keeps disk reads and writes out of the time-sensitive audio processing.

**Calibration places new parts on the timeline.** The app sends a test signal through the audio path and measures how late it returns. That measured delay adjusts where a new recording starts. If the signal cannot be matched clearly, calibration reports failure.

**Undo changes the arrangement.** Takes stay in separate audio files. A session records which layers play and where they begin, so undo can restore an earlier arrangement without rewriting the sound. Preserving takes also uses more disk space.

## Where it stands

The desktop prototype includes recording, playback, monitoring, track levels, mute, effects, merging, export, saved projects and undo/redo. The private test suite covers recording placement, layers, storage failures and recording/playback across threads and WAV files.

A current demo and broader device testing are pending. The standalone hardware port, sampler, unused-take cleanup and crash recovery remain unfinished.

## Further reading

[Audio architecture](docs/architecture.md) · [Implementation notes](docs/engineering-decisions.md)

Application source and hardware designs remain private.
