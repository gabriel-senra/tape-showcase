# Implementation tradeoffs

## Bounded queues and overload

A fixed-size queue avoids allocating when a producer sends a message, but it cannot absorb unlimited work. Both producer and consumer must have clear ownership, and callers must handle failed pushes and pops. Tape's command/status queues and audio streaming paths use this boundary to keep control and disk work separate from audio processing.

Playback starvation and recording overflow need different handling. Missing playback data can become silence while the timeline advances; dropped recording data must preserve the placement of later audio. Completing a take also needs special care, so a final chunk can be retried when the queue is full.

## Measured delay and calibration failure

Device and software delay makes capture time different from the time the musician intended to play. A round-trip measurement gives take placement a concrete correction in frames. It also depends on hearing a clear return signal. The calibration therefore needs a failure result, and the application must not treat a failed measurement as a valid offset.

## Revisions and disk usage

Arrangement revisions refer to preserved take files. This makes recording and clearing reversible without copying or rewriting the audio for every edit. The cost is retained material and history metadata. A cleanup policy must respect revisions that still refer to those files.

## Shared logic and platform adapters

Separating the C++ engine from JUCE makes engine behavior independently testable. It also requires explicit interfaces for devices and storage. Those interfaces are useful for a future hardware port, but the target device still needs its own performance and reliability measurements.
