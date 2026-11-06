# Persistent startup and card loading

## Separate the milestones

A persistent file, a consumer that reads it, code execution in a helper, and code execution in the camera main process are four different findings. They were tested separately.

1. **Retention:** an exclusively created 53-byte ordinary marker in `/etc` was synchronized and read back. Following user-confirmed battery removal and cold startup, a read-only probe found identical bytes. Exact-content removal and absence checks completed afterward.
2. **Loader consumer:** the original `ld-2.25.so` has an access/read/parse path for `/etc/ld.so.preload`. A private QEMU chroot with original target libraries confirmed the mechanism offline. The actual camera loader/helper identities were then compared with those files.
3. **Startup trigger:** a minimal constructor library acted only when `/proc/self/exe` identified `/usr/bin/btproperty.elf`. Its receipt was absent before the cold-start test and present afterward. Original boot records and `startBTCore.sh` provided supporting startup context. The host did not explicitly launch that helper.
4. **Main-process delivery:** an isolated worker used native interprocess communication and native file transfer to request loading of a minimal library in `/usr/bin/im.elf`. Separate worker and main-process completion receipts both passed verification.

The receipts establish the recorded executions. They do not encode precise startup latency; where timing was not recorded, no precise completion time is claimed.

## Bootstrap architecture

```text
persistent /etc/ld.so.preload
        |
        v
small library mapped by original dynamic loader
        |
        +-- unrelated process: guarded constructor returns
        |
        +-- expected factory helper: starts bounded isolated worker
                                      |
                                      v
                           original IPC/file-transfer services
                                      |
                                      v
                             /usr/bin/im.elf executes
                             a main-process stage
                                      |
                                      v
                          native Fsys reads card payload
                                      |
                                      v
                           resident extension / UI dispatch
```

A helper's successful load is not sufficient to register UI in a snapshot-restored main process. UI readiness and callback/resource ownership must be validated in the destination process. The worker provides transport; the extension owns its runtime and UI state.

## Native queue semantics

An early main-process transfer test exited before delivery because its queue gate misinterpreted native status words. Static replay and hardware records distinguished asynchronous listener counts from the synchronous callback pointer. The observed accepted service and DMM status tuples differed; an asynchronous DMM listener was not evidence of a pending message or a missing synchronous recipient.

The revised test obtained a completed transfer receipt and a separate execution receipt from the main process. This is why the research distinguishes a successful mode query from successful library delivery/execution.

USB read polling also mattered operationally: an earlier query finished according to its later recovered receipt despite a host read timeout. A later comparison left a 30-second quiet interval before one read and completed successfully. This supports avoiding competing diagnostic requests during a synchronous worker operation; it does not prove a universal 30-second startup requirement or a single root cause for every timeout.

## SD-card access

The working family uses Sony's native `Fsys` interfaces through the camera main process. Investigated removable namespace roots include:

```text
/nondev/rm1
/nondev/rm2
/nondev/rm3
```

These names are native filesystem namespaces, not proof of Linux POSIX mount points or a universal `rm1 == Slot 1` rule. Treat card presence, namespace readiness and file identity as explicit conditions. A later bootstrap build scanned the native removable candidates and the user observed its menu after a disconnected cold start with the card in Slot 1.

The internal bootstrap and card payload were paired by expected filename, size, digest and build-specific native anchors. Changing a payload requires updating the corresponding identity contract. A general-purpose arbitrary-plugin ABI, hot reload and update compatibility have not been established.

Once loaded, the resident module handles menu events from memory. This architecture does not reload the entire SD file for each menu scroll. UI disappearance during navigation is not evidence of insufficient card throughput. It requires separate inspection of native redraw, object lifetime and callback handoff.

## Cold start, ordinary OFF/ON and sleep/wake

The platform has a snapshot/resume startup path. Battery-removal cold startup, ordinary power-switch OFF/ON, display blanking and sleep/wake must be recorded separately. Seeing a constructor execute after one cold start does not prove automatic reinstallation on every resume.

Native menu/input objects may be destroyed and recreated across those transitions. A still-mapped library can have stale object pointers or inactive hooks. Thus boot loading and reliable menu reattachment are distinct engineering problems.

## Transaction order and recovery limits

The test installer refused preexisting target configuration and unrelated files. It wrote/verified the library before enabling configuration. Removal validated ownership, disabled the matching startup configuration first, synchronized the directory, then removed matching libraries and receipts. An independent read checked absence.

Configuration is global to newly executed dynamic programs, even when the constructor acts only in a single helper. A crashing or invalid library can affect startup before its own process guard provides useful protection. Battery removal does not delete configuration. A separate recovery channel that works when normal startup/service entry is unavailable has not been verified.

No claim is made here that removing the SD card safely disables every bootstrap version, that deleting a card file uninstalls the internal loader, or that these settings survive firmware updates/factory reset. Those require their own tests.
