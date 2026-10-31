# Tape implementation tradeoffs

These decisions follow from the recording workflow: audio should continue while the application reads files, aligns new takes and lets the musician reverse edits.

## Why use fixed-size queues?

A queue lets one thread pass work to another. Preallocating a fixed number of slots avoids growing a container while processing audio. Each queue has one producer and one consumer, so there is no ambiguity about which side may write each index.

The cost is a capacity limit. Callers must handle a rejected push. For playback, unavailable data becomes silence while the timeline advances. For recording, the gap must be reflected in later placement so the remaining audio does not slide earlier. The final recording chunk is retried when necessary so stopping does not immediately discard it just because the queue is full.

A larger queue can tolerate a longer interruption, but it also uses more memory. It does not solve sustained storage overload.

## Why measure recording delay?

Device-reported timing and the audio's actual round trip can differ. Comparing a known emitted signal with its recorded return measures the complete path being used. Tape finds their relative shift and expresses the correction in audio frames.

This measurement needs a usable return signal. When a clear match cannot be established, calibration fails explicitly. Applying an uncertain offset could make alignment worse.

## Why preserve takes and store revisions?

Copying a complete audio file for every edit would be expensive, while overwriting it would make undo difficult. Tape keeps the takes and changes the arrangement that refers to them. A revision describes the layers on the tracks, so undo can select an earlier arrangement without rewriting the sound.

Retaining takes uses disk space. A future cleanup policy must check whether a file is still referenced by history before removing it.

## Why separate the engine from the desktop app?

The macOS application needs JUCE controls and device APIs. The engine's recording and mixing behavior can be expressed independently of them. Keeping those responsibilities separate allows deterministic engine tests and makes platform dependencies easier to identify.

The additional interfaces take work to maintain. A future hardware port will also need its own CPU, memory and storage measurements; a portable interface is a starting point, not proof that the target device is fast enough.
