# Tape implementation notes

## Queue storage

The queues reserve a fixed number of slots before audio processing starts. Each has one producer and one consumer, and a push is rejected when the queue is full. This avoids growing a container or waiting for disk access during the audio callback.

If playback data is unavailable, the engine outputs silence while advancing the timeline. Recording gaps are counted so subsequent chunks keep their intended position. The final chunk of a take is retained for another attempt if it cannot be queued immediately.

Increasing queue capacity uses more memory and absorbs longer interruptions. It cannot compensate for storage that is consistently slower than the recording rate.

## Recording delay

Calibration measures the actual return of a known test signal rather than relying only on the device's reported timing. The matching offset is expressed in audio frames and applied when placing a take. A measurement without a clear match is rejected.

## Take files and arrangement revisions

Recorded audio stays in separate take files. An arrangement revision holds the references and positions needed to play those files together. This keeps undo independent of file rewriting and avoids copying a full recording for every edit.

Preserved takes use disk space. Cleanup will need to check references from the arrangement history before removing a file.

## File errors

The worker reports failed reads and writes to the engine. A failed recording is not added to the arrangement. A read failure produces silence and an error event while preserving the timeline position.

Tests cover these paths, full queues, stalled storage and recording another take after a failure. Worst-case callback timing and storage throughput still require measurements on each device setup.
