# Evidence index and reproducibility boundaries

This publication is a curated research snapshot. The original workspace contains experiment notes, decoded records, firmware extraction results and private device traces. They were not copied wholesale.

## Evidence levels

- **Static:** observations in an identified firmware file; independently checkable with a separately obtained copy.
- **Offline execution:** actual target code exercised with emulated/stubbed dependencies; proves only the tested conditions.
- **Device record:** completion/readback from the tested body, decoded under that experiment's record format.
- **Visual/user observation:** what appeared on the camera and whether native controls/shutdown worked.

Each level answers a different question. Build flags such as `hardware_tested: false` are build-time metadata and were not silently rewritten as later experimental acceptance.

## Curated record index

| ID | Result | Public evidence |
| --- | --- | --- |
| E01 | Initial authentication led to service PID 0336 | Sanitized observation and retained source-note digest |
| E02 | 53-byte `/etc` marker survived a battery-removal cold start | Selected readback fields; separate successful removal observation |
| E03 | Startup constructor receipt absent before, present after cold start | Selected transaction fields and startup-note provenance |
| E04 | Automatic worker completed native delivery; main process executed the canary | Separate worker/main-process acceptance fields |
| E05 | SD bootstrap configuration and resident menu execution | Selected build identity and resident observer fields; user confirmed visible disconnected menu |
| E06 | Original-style full page displayed/hidden during extended observation | Selected native page result fields; user visual feedback recorded in narrative |
| E07 | Later runtime candidate failed lifecycle/navigation acceptance | Brief limitation summary and retained source-note digest |

Structured observations are in [observations.json](../evidence/observations.json). Firmware identities are in [firmware-identities.json](../evidence/firmware-identities.json).

## What can be reproduced from this repository

The pinned firmware hashes and service landmarks support independent static review. The notes explain the loading stages, ownership checks, rejected assumptions and the distinction between boot and resume.

This snapshot does **not** provide a self-contained hardware reproduction kit. Raw transport traces, authentication inputs, original firmware binaries, native payload binaries and experimental deployment tools are withheld. The public selected fields and source digests document retained observations; a source digest alone does not independently prove a hardware claim. External researchers need their own authorized device access and experiments to reproduce those claims.

Source hashes bind each curated item to the retained local record as read for publication. No original source file is linked as if it were included here. Historical statements about a test configuration being installed or removed are not statements about the current camera state.

## Excluded material

No camera serial number, Windows user path, network identity, credential, authentication table, challenge/digest transcript, photo, firmware image, complete disassembly, USB capture or unfinished MOD binary is included. Sony firmware and upstream code have not been relicensed or redistributed as part of this documentation snapshot.
