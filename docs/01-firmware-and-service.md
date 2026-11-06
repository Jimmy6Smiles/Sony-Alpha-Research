# Firmware identity and service execution

## Target and static analysis

The investigated target is Sony ILCE-1M2 firmware 4.00. The locally analyzed official update has 1,194,332,648 bytes and SHA-256:

```text
f6f8db5735fd15584dfc0118f84e537876c3aaeba72a9971388c21334ce34614
```

This is a locally computed identity, not a claim that Sony published that digest. Selected extracted ELF identities are in [firmware-identities.json](../evidence/firmware-identities.json). Obtain firmware from Sony; no firmware is distributed here.

The service libraries are ELF64/AArch64. The original `ma1co/fwtool.py` version used in the investigation recognized the update container but lacked a working decryptor for it. A complementary unpacker from `q19032/Sony-firmware-unpacker` completed extraction after retaining two candidate entries that otherwise shared the same dictionary key. The successful candidate's tool label is not independent evidence of a physical chip model. Container extraction does not establish a firmware-update signature bypass or a complete body backup.

## Two USB stages

The tested Windows route retained the native USB mass-storage driver and used a signed device-scoped libusb-win32 filter for control access. Only the three identified camera storage children were temporarily paused during quiet service communication, then restored after disconnection. This is a description of the tested setup, not a recommendation to replace arbitrary system drivers.

The initial service handshake caused a handoff from normal MSC to Sony service USB PID `0336`. That handoff is not the authenticated service-command session: the re-enumerated interface required its own listener/authentication sequence before Jiritsu operations worked.

An endpoint-zero STALL by itself did not establish firmware rejection. Host driver binding, concurrent storage traffic, command completion and the resulting USB identity had to be assessed separately. A command being sent, or a host trace tool failing to stop, did not by itself determine the camera transaction's result.

Authentication tables were compared with the public Sony-PMCA-RE implementation during local research. No table bytes, challenge/digest exchanges, or usable authentication material are included in this publication.

## Exact file-to-code branch

In the pinned `libJiritsu.so`, the investigated file handlers contain explicit dynamic-load branches for the exact target:

```text
/tmp/jiritsu/libJit.so
```

For this exact ELF only, static landmarks include:

| Symbol/path | ELF virtual address | Observation |
| --- | --- | --- |
| `adj_cmd_set_file` | `0x9008` | Writes/synchronizes the file; exact-path branch reaches `dlopen` at `0x9230` and `dlclose` at `0x9274` |
| `adj_cmd_set_file2` | `0x983c` | Another file-access implementation; exact-path branch reaches `dlopen` at `0x9c1c` and `dlclose` at `0x9c3c` |
| `adj_cmd_set_file_path` | `0x8cc8` | Performs length/character checks and path handling |
| `get_access_filename` | `0x95fc` | Selects/formats the request path; selectors and path conditions matter |

These are build-specific landmarks. They are not portable patch addresses, a complete wire-protocol specification, or permission to overwrite an occupied slot. Static path analysis was followed by separate hardware tests; later execution evidence, rather than the presence of a `dlopen` string, supports the code-execution finding.

## Verification contract

Experiments used separately identified payloads and receipts. Installation was accepted only after reading back the expected bytes. Execution was accepted only after decoding a matching completion record. Unknown slot contents stopped a transaction rather than being deleted.

File removal and library unloading are separate facts. In particular, a module registered with retained native callbacks may remain mapped even after its on-disk file is deleted. Restoring an entry-point pointer also does not prove that an already executing callback has returned.

## Internal storage is not the exposed PC volume

Extracted configuration mounts `nflasha15` at `/usr` as an ordinarily read-only ext2 filesystem and uses `nflasha10` for writable `/tmp`, `/etc` and `/var` mappings. A path named `/tmp` must therefore not be assumed to be disposable RAM on this system.

These internal filesystems are distinct from the SD card and the small USB presentation volume exposed to a computer. Firmware extraction established the configured layout; the later marker experiment separately established actual cold-start retention for one `/etc` file. It did not establish arbitrary partition-write safety.
