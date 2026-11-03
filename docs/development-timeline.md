# Tape status

The current platform is a macOS desktop prototype. It includes four-track recording and playback, overdubbing, monitoring, levels, mute, effects, track merging, export, saved projects and undo/redo.

Recording placement uses audio frames and round-trip delay calibration. The private test suite covers engine behavior, storage failures and threaded WAV recording/playback. A fresh full-application run, broader device checks and a current demo are pending.

Work remaining includes unused-take cleanup, crash recovery, the sampler and an embedded hardware port. The port will need its own CPU, memory and storage measurements.

This repository contains project documentation. The application source and hardware designs remain private.
