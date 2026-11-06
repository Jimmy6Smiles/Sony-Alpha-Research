# Developing a native camera extension

This workflow explains how the research pieces fit together. This snapshot does not include a turnkey installer, authentication implementation or a stable general-purpose camera SDK.

## 1. Establish the exact target

Record the model, firmware version and hashes of the libraries whose interfaces will be used. Confirm ELF class, machine, dynamic interpreter, symbol versions and dependencies. The tested environment is AArch64 Linux with the original glibc 2.25 loader, rather than the Android application environment of older PlayMemories cameras.

Compile a position-independent AArch64 shared object against a compatible target runtime. Inspect the ELF instead of assuming that any ARM64 library is compatible. Avoid requiring newer host glibc symbols, unavailable C++ runtime versions, unsupported relocations or incorrect OS constants. AArch64 and x86 Linux do not assign every file-open flag the same numeric value.

Useful offline inspection commands, assuming the corresponding cross tools are installed:

```sh
aarch64-linux-gnu-readelf -h -l -d extension.so
aarch64-linux-gnu-readelf --dyn-syms --version-info extension.so
aarch64-linux-gnu-objdump -d extension.so
```

The exact library paths and link dependencies must come from the identified target environment; these commands do not constitute a complete SDK.

## 2. Prove the smallest load

The verified fixed-slot path reaches the original dynamic loader. A shared library initializer can serve as a minimal execution probe; this differs from implementing the separate Jiritsu script-object ABI.

Use a bounded, process-guarded initializer that performs one identifiable observation and returns. A receipt should distinguish not started, entered, completed and failed states, and bind itself to the exact build/experiment. It must be absent before the test, otherwise an old file can be mistaken for a new execution.

Read back and validate both the deployed library and the completed result. Separate device execution from host decoding. Simulation can verify layouts and error paths, but a mocked dynamic loader does not prove the camera accepted a library.

The original handler may close its library handle after loading. If an extension installs callbacks or retains native objects, its executable code must remain valid for their entire lifetime. Deleting the file, returning from initialization or restoring a callback slot does not establish that all existing executions/references have ended.

## 3. Separate loader, transport and runtime

A useful organization is:

```text
bootstrap/          minimal startup trigger and process guard
transport/          bounded native IPC/file delivery
runtime/            main-process module identity and state machine
platform/           verified native API adapters
feature/            extension-specific behavior
diagnostics/        build identity, progress and completion records
```

The factory helper used for startup is not the camera UI host. Its address space cannot be treated as the main program's. The demonstrated bridge uses native interprocess services to deliver a main-process stage. Validate the destination identity and readiness before initializing the runtime.

Keep expensive work out of UI callbacks. Native wrappers must reflect observed calling conventions, return values, thread requirements and object ownership. Finding a symbol or string does not establish a complete callable ABI.

## 4. Add one native capability at a time

For each interface adapter, establish:

- How an object/handle is obtained and which process owns it.
- Which thread may call an operation and how completion is reported.
- Buffer bounds, allocation space, reference counts and release order.
- Partial failure, cancellation and native object replacement behavior.
- How original camera controls remain usable when extension context is invalid.

The research demonstrated UI-thread dispatch, native graphics resources, original-style controls, live-view JPEG acquisition and onboard preview processing. These are primitives, not a universal stable API.

One graphics experiment showed why allocation space matters: a private image retained from a screen bitmap's allocation space could prevent that bitmap's retirement during format replacement. A retained address may also refer to a newly created object of a different class. Identity and lifetime require fresh checks.

## 5. Move the payload to the card

Use the original native filesystem abstraction. The investigated removable roots are namespace candidates, not assumed POSIX mounts or universal slot numbers. Wait for a verified readable namespace and validate the complete payload before loading it.

Bind the bootstrap to an expected name, maximum/actual length, digest and supported runtime version. Keep previous artifacts separately for rollback. A configurable plugin format and a fixed hash-pinned research payload are different designs; the latter does not automatically provide arbitrary replacement or hot reload.

Load the module into the destination process, then handle normal events from memory. Ordinary menu scrolling does not require reading the entire extension from the card again.

## 6. Enable persistence after manual execution works

Prove writable-file retention separately from the consumer that executes it. The demonstrated route uses the original loader's persistent preload configuration and a process-guarded bootstrap; its scope is described in [startup and card loading](02-startup-and-card-loading.md).

Write/verify the library before enabling configuration. Validate ownership before modifying an existing path. Disable matching configuration before removing dependencies. Keep readback/removal verification independent of the installer response.

Test battery-removal cold startup, ordinary OFF/ON, sleep/wake and playback separately. A snapshot-restored main program may require reattachment even while the library is mapped. A startup receipt does not establish reliable callback reacquisition.

## 7. Define acceptance independently of installation

Require completed execution records and user-visible behavior where appropriate. Repeated navigation, original controls, touch, suspend/resume, object recreation and normal shutdown must remain usable. The experimental runtime still showed failures in these areas, so no stable SDK or consumer MOD is claimed.

These limits define the work between a demonstrated execution primitive and a reliable extension. They do not invalidate the proven service, file, boot and card-loading mechanisms.
