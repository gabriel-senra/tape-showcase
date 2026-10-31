# Tape project status

Tape is currently a macOS recording prototype. The desktop app is the working platform for developing a possible future standalone recorder.

| Area | Implemented | Next validation or development |
| --- | --- | --- |
| Recording and mixing | Four tracks, overdubbing, monitoring, levels and mute | Broader audio-device testing and a current demonstration |
| Timing | Placement in audio frames and round-trip delay calibration | Calibration and listening checks on each device setup |
| Editing | Recording layers, saved projects and undo/redo revisions | Unused-take cleanup and crash recovery |
| Sound and output | Effects, track merging and export | Further listening and workflow testing |
| Automated testing | Engine, storage-failure and threaded WAV tests in the private repository | A fresh full-application run for the current build |
| Embedded hardware | Engine separated from desktop-specific code | Porting, memory/CPU measurements and storage-throughput testing |

The sampler remains planned. This showcase contains documentation, not an application download or the production source.
