# Tape

Tape is a four-track recorder prototype for macOS. It is built for capturing a musical idea, adding another part, and continuing to play without managing a large recording session. It uses C++ for audio processing and JUCE, a C++ audio application framework, for the desktop interface.

For example, a musician can record a piano part, play it back while adding bass, and balance the two tracks. Adding another recording to an occupied track creates a new layer. Undo removes that edit from the arrangement while preserving the original audio.

**C++17 · JUCE · WAV audio · CMake · Catch2**

## What the prototype supports

- Four-track recording, playback, level control and mute.
- Sound-on-sound recording, where a new layer plays alongside the earlier layers on the same track.
- Undo/redo and saved projects for returning to a session later.
- Input monitoring, recording-delay calibration, effects, track merging and audio export.

The macOS prototype also provides a way to develop the engine for a future standalone instrument. This repository documents the project; the application source and hardware designs remain private.

## Engineering challenges and solutions

### Keeping audio moving while files are being read and written

An audio device repeatedly asks the application for a small block of sound. That function is the **audio callback**, and it needs to finish before the next block is due. Waiting for a file operation inside it could interrupt playback.

Tape moves WAV-file reads and writes to a background storage worker. Fixed-size queues act as mailboxes between the engine and the worker, letting each side exchange prepared chunks without making the callback wait for disk access.

A mailbox can still fill, or storage can fall behind. Missing playback data becomes silence while the timeline advances. Recording gaps are counted so later audio retains its intended position. The final chunk of a take is kept for another attempt if it cannot be queued immediately. These policies make overload visible; they do not make slow storage unlimited.

### Making a new recording line up with the part being played

Playback and recording introduce delay. Without compensation, an overdub can land later than the musician played it.

Tape sends a known test signal through the audio path and compares its recorded return with the original. **Cross-correlation** finds the time shift where the signals match most closely. That measured delay is used to correct where a new take is placed.

Positions use whole **audio frames**—one sample instant across the audio channels—rather than rounded wall-clock timestamps. If calibration cannot identify a clear match, it returns failure instead of a guessed correction.

### Letting a musician undo a recording without losing the take

Overwriting an audio file would make an accidental recording hard to reverse. Tape keeps take files and stores **arrangement revisions**: descriptions of which layers play on each track and where they begin. Undo and redo select a different revision instead of rewriting recorded sound.

A new edit after undo replaces the redo branch in the arrangement history. The old take files remain on disk, so this approach preserves material at the cost of additional storage. Cleanup and crash recovery remain part of the reliability work.

### Recovering from a failed recording

A failed disk write must not turn an incomplete take into a successful edit. The storage worker reports the failure, and the failed take is not added to the arrangement. A read failure produces silence and an error event while retaining the timeline position.

Tests cover failed reads and writes, a full recording queue, stalled storage, and starting another take after a failure. This matters because a recorder has to handle an unsuccessful operation as carefully as a successful one.

## Testing and current status

The private repository contains Catch2 automated C++ tests for message ordering, recording placement, layers, storage failures, memory-allocation checks in the audio callback, and recording/playback through real threads and WAV files. Those are engine and integration tests; they do not replace listening and device testing on macOS.

The desktop prototype is implemented. The embedded hardware port, sampler and further reliability work are still in development. Device-specific calibration and a current demonstration are still needed.

For a deeper explanation, see [architecture](docs/architecture.md), [implementation tradeoffs](docs/engineering-decisions.md), and [project status](docs/development-timeline.md).
