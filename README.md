# Tape

Tape is a four-track recording prototype for macOS, written in C++ with a JUCE interface. It explores a simpler way to capture musical ideas while handling the difficult parts underneath: moving audio between threads, keeping overdubs aligned, and letting a musician undo a recording without destroying the original take.

**C++17 · JUCE · WAV/PCM · CMake · Catch2**


## Engineering overview

| Problem | Approach in the source | Evidence reviewed |
| --- | --- | --- |
| Disk access cannot stall the audio callback | Separate audio processing and background storage using bounded messages | Engine and storage worker implementations; threaded file-streaming test |
| New takes arrive later than playback | Estimate round-trip delay and place recordings on an integer-frame timeline | Calibration implementation and known-delay / overdub tests |
| Recording over a part should be reversible | Keep takes and move between arrangement revisions | Revision-history implementation; layering tests |
| Fast UI changes should not become abrupt audio changes | Advance gain targets through sample-stepped ramps | Smoother implementation and ramp tests |

## What is implemented

The reviewed source contains four-track recording and mixing, sound-on-sound layers, per-track faders and mutes, calibration, monitoring, project revisions, and undo/redo. The repository also contains effects, bounce and export code. The focus here is the recording architecture, where the evidence is strongest.

These are source-inspection findings, not a claim that the current application was built or auditioned in this review environment. Automated tests exist for exact-frame placement, queue overflow, missing storage data, allocation checks and real threaded WAV recording/playback. The private application's test suite has not been rerun here.

## Technical documentation

- [Architecture and thread boundaries](docs/architecture.md)
- [Engineering decisions and tradeoffs](docs/engineering-decisions.md)
- [Verified snapshot and development status](docs/development-timeline.md)
- [Runnable educational examples](https://github.com/gabriel-senra/engineering-examples): bounded queue and parameter ramp; prepared separately from the product


## Current limitations

Tape is a desktop prototype for a possible future instrument. This showcase does not claim an embedded hardware port, a measured MCU budget, universal glitch-free operation, or a finished sampler. Device-specific latency and reliability need real hardware testing. A finite queue can fill; a design must account for rejected messages and storage gaps rather than hide them.

The full engine, session orchestration, project storage, calibration implementation and hardware product plans remain private.
