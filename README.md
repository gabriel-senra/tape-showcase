# Tape

Tape is a four-track recorder prototype for macOS, built with C++ and JUCE. The idea is to capture a musical phrase and add parts without setting up a large recording session. The desktop app is also the starting point for a possible standalone recorder.

A session has four tracks with individual levels and mute controls. You can record a piano part, listen back while adding bass, then balance them. Recording onto an occupied track adds a layer alongside the existing audio. Sessions can be saved, edits undone or redone, and tracks merged or exported. The prototype also includes input monitoring, effects and recording-delay calibration.

## How it works

### Audio and file storage

JUCE connects the app to the audio device and provides the desktop interface. The device repeatedly requests a block of sound from the engine through an audio callback. The engine mixes prepared playback data with the incoming audio.

A background worker reads and writes the WAV files. Fixed-size queues carry recording chunks, playback requests and prepared audio between the worker and engine. File access stays outside the callback because disk operations can take longer than the device allows for a block of audio.

### Recording placement

Take positions are stored in audio frames: one sample instant across all channels. Calibration sends a known signal through the audio path and compares its recorded return with the original. Cross-correlation finds the offset where the two signals match most closely, and that delay corrects the placement of new recordings. Calibration reports failure if there is no clear match.

### Layers and undo

Each take is stored separately. An arrangement records which layers belong to each track and where they start. Undo and redo select earlier or later arrangement revisions without rewriting the recorded audio. A new edit after undo replaces the redo branch; the underlying take files remain on disk.

## Current version

The desktop prototype is implemented. Its private repository contains Catch2 tests for recording placement, layers, message ordering, callback allocations, storage failures and threaded WAV recording/playback.

Broader device testing and a current demo are pending. The embedded port and sampler are not complete; unused-take cleanup and crash recovery also remain to be developed. Application source and hardware designs remain private.

More detail: [architecture](docs/architecture.md), [implementation notes](docs/engineering-decisions.md), [status](docs/development-timeline.md).
