# Tape

Tape is a four-track recorder prototype for macOS, built in C++ with a JUCE interface. It supports recording and layering parts, mixing tracks, undoing edits, and saving a project to continue later. The desktop application is also a way to develop the audio engine for a future standalone instrument.

**C++17 · JUCE · WAV/PCM · CMake · Catch2**

## Recording workflow

Record a part on one track, play it back while recording another, and adjust each track's level or mute. Recording onto an occupied track adds a layer rather than replacing the existing audio. Undo and redo move through arrangement revisions, while the original takes remain available on disk.

The prototype also includes monitoring, latency calibration, effects, bounce and export. The application and its full source are private; this repository explains the implementation and its tradeoffs.

## Engineering challenges and how they are handled

### Keeping disk access out of the audio callback

The audio device needs each block on time, while file reads and writes can take an unpredictable amount of time. Tape separates the audio engine from a background storage worker. Fixed-capacity queues carry commands, status and audio chunks between those responsibilities without asking the callback to wait for a file operation.

That separation introduces another problem: storage can fall behind. Playback continues with silence for missing data and records the gap in its status. Recording gaps are counted so later chunks retain their correct timeline position. A final recording chunk that cannot be queued immediately is retained for another attempt.

### Keeping an overdub aligned with the part being played

Audio takes time to travel through playback and recording devices. Placing new audio at its arrival time would make an overdub late. Tape measures the round-trip delay using a known signal and cross-correlation, then corrects take placement in audio frames. An inconclusive calibration returns failure rather than a guessed delay.

The timeline uses integer frames so positions remain tied to individual samples. Tests exercise known recording delays, take placement at the beginning of the timeline, and overdubs against an existing track.

### Making recording and clearing a track reversible

Destructively replacing a file would make an accidental recording or clear operation difficult to undo. Tape stores recordings as takes and represents the arrangement as revisions containing layers. Undo and redo select a revision; they do not need to rewrite the recorded audio.

This preserves the original material, but take files and history consume storage. Cleanup and crash recovery remain part of the hardening work.

### Handling a failed take without corrupting the session

A write failure must not make an incomplete take appear to be a successful recording. Storage failures are reported, and failed takes are not added to the arrangement. Read failures produce silence and an error event so playback can retain its position while the problem is reported.

The private tests include failed reads and writes, stalled storage, a full recording queue, and a new recording after a failed take.

### Developing the engine independently of the desktop interface

The JUCE application handles desktop controls and devices. The audio engine uses standard C++ interfaces, with storage and device adapters outside the core. This lets the same engine logic be tested without opening the application and keeps a future hardware port from depending on the desktop UI.

An embedded port still needs separate CPU, memory and storage-throughput measurements; portable interfaces alone do not establish hardware readiness.

## Testing

The private repository contains Catch2 tests for queue boundaries and ordering, exact-frame recording placement, layering, storage failure, callback allocation checks, and recording/playback through real threads and WAV files. These tests cover the engine and integration behavior; a fresh macOS device run is still needed for the current demonstration.

## Further documentation

- [Audio, control and storage boundaries](docs/architecture.md)
- [Implementation tradeoffs](docs/engineering-decisions.md)
- [Project status](docs/development-timeline.md)

## Current status

The project is a macOS prototype. The standalone hardware, sampler and additional reliability work remain in development. Full production source and hardware designs are private.
