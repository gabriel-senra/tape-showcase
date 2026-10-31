# Engineering decisions worth explaining

These are selected, externally understandable tradeoffs, not a full reproduction of private design documents.

## 1. Separate real-time audio from storage

**Problem:** Filesystem I/O and general heap allocations can have unpredictable timing.

**Choice:** Bounded, preallocated messaging between a deadline-sensitive audio callback and background workers.

**Tradeoff:** Bounded queues can fill. The code must specify what happens on overflow rather than pretending infinite buffering is available.

**How it is tested:** Empty/full queue behavior, message order, concurrent producer/consumer behavior, and storage edge cases.

## 2. Align overdubs by measured latency

**Problem:** Recording devices and software introduce delay between playback and capture. Untreated, new takes can sound late.

**Choice:** Measure round-trip delay with a known signal and cross-correlation; place takes on a frame-based timeline.

**Tradeoff:** Calibration depends on a clear return signal and should signal failure rather than invent a plausible offset.

## 3. Prefer non-destructive revisions

**Problem:** A musician should be able to undo a recording or clear operation without losing the original take.

**Choice:** Preserve take files and store arrangement revisions. Save history across sessions.

**Tradeoff:** Revision metadata consumes additional memory/storage and requires a clear policy for cleanup and history growth.

## 4. Keep the portable engine separate from the desktop layer

**Problem:** The Mac app uses desktop APIs, but a future embedded instrument may not.

**Choice:** Separate a C++ audio core from device adapters, UI and desktop filesystem implementations.

**Tradeoff:** More explicit interfaces, but clearer test boundaries and portability.

## Claims we should NOT make without fresh evidence

- Zero audio glitches or underruns on every device.
- A specific latency or CPU performance improvement.
- A complete commercial hardware implementation.
- Passing GitHub CI in the original private repo.
- Full product feature completion based solely on roadmap text.
