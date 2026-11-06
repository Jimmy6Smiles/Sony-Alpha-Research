# Sony Alpha Research

[简体中文](README.zh-CN.md)

How to access and modify internal camera files, execute native code, develop extensions, and load them from an SD card. Tested target: **Sony α1 II / ILCE-1M2, firmware 4.00**.

The loading chain has been demonstrated on a physical camera. This publication focuses on reusable mechanisms, rather than a specific photography feature. A stable general-purpose extension SDK is not claimed. This first publication contains documentation and curated observations, without an installer, firmware images, authentication material, or experimental MOD binaries.

## Demonstrated chain

```text
Windows MSC connection
  -> factory service authentication
  -> Jiritsu file channel and fixed-slot library execution

Cold startup
  -> persistent dynamic-loader configuration
  -> process-scoped bootstrap in a factory helper
  -> isolated worker and native interprocess request
  -> library execution inside the camera main process
  -> native filesystem reads of an SD-card payload
  -> resident extension and native menu experiment
```

The SD card is the extension's storage location. The demonstrated startup trigger is a small installation in writable internal storage; this is **not** evidence that an unmodified camera executes an SD-card file by itself.

## Findings and their limits

| Finding | Evidence | Limit |
| --- | --- | --- |
| Windows service entry and authenticated file operations | Physical camera transactions | Initial USB handoff and service authentication are separate stages |
| Code execution through the fixed Jiritsu library slot | Static call path plus hardware execution records | Uploading an arbitrary filename does not establish execution |
| Writable internal configuration survives power removal | Exact 53-byte marker before/after a battery-removal cold start | Not a test of factory reset or firmware-update survival |
| Automatic execution through the original dynamic loader | Absent-before/present-after constructor receipt and original boot records | Global preload has broader scope than the constructor's process guard |
| Boot worker can cause main-process library execution | Separate successful worker and main-process receipts | A mode-query reply alone is insufficient |
| Card-backed native menu payload | SD bootstrap builds, main-process observer and user-visible menu | No universal physical-slot-to-native-root mapping is claimed |
| Native capability integration | UI dispatch, graphics and live-view preview experiments | Each interface needs independent ABI, thread and lifetime validation |
| Stable runtime across all camera states | **Not established** | Object replacement, input ownership and resume reattachment remain unfinished |

## Read the research

1. [Firmware identity and service execution](docs/01-firmware-and-service.md)
2. [Persistent startup and card loading](docs/02-startup-and-card-loading.md)
3. [Developing a native extension](docs/03-extension-development.md)
4. [Evidence index and reproducibility boundaries](docs/04-evidence.md)
5. [References and attribution](docs/05-references.md)

Machine-readable material: [firmware identities](evidence/firmware-identities.json), [curated observations](evidence/observations.json).

## Compatibility and recovery boundary

The tested body/firmware combination is ILCE-1M2 4.00. Addresses, native object layouts, protocol assumptions and loader behavior require independent verification on another build. Installation/readback success does not establish menu stability or successful photography.

Persistent startup configuration survives battery removal. If an invalid library prevents normal startup and service access, an independent recovery route has **not** been demonstrated in this research. The notes describe experiments rather than a supported installation procedure.

Snapshot: **2026-11-05**. Evidence here describes the named experiments, not the camera's current installation or connection state.
