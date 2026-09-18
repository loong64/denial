# Flutter Engine — Linux LoongArch64 profile

`libflutter_engine.so` is Denial's optimized LoongArch64 profiling engine. It
retains the authenticated loopback VM service required by Flutter DevTools
without using a debug or JIT runtime.

The shared library is ignored by Git. This directory tracks the generated GN
configuration, coupled source revisions, checksum, and license files needed to
reproduce and package the artifact.

## Source identity

The authoritative source input is
[`SOURCE_LOCK.json`](../SOURCE_LOCK.json). Generated `args.gn` and the adjacent
revision files record the derived build and ABI identities without duplicating
mutable lock values in this document.

The Flutter fork's DEPS file pins the matching Skia fork and Dart source. The
prebuilt LoongArch64 Dart SDK is installed at
`engine/src/flutter/prebuilts/linux-loong64/dart-sdk`. The current build uses
validated local fork checkouts when official LoongArch64 CIPD packages are not
available.

## LoongArch64 build

The equivalent direct engine commands are:

```sh
./flutter/tools/gn \
  --runtime-mode=profile \
  --enable-fontconfig \
  --target-os=linux \
  --linux-cpu=loong64 \
  '--gn-args=build_analyze_snapshot=true host_cpu="loong64"' \
  --target-dir=denial_host_profile
/usr/bin/ninja -C out/denial_host_profile libflutter_engine.so
```

The revision-keyed incremental builder is preferred:

```sh
tools/denial-flutter-engine build
```

It verifies the source lock and DEPS resolutions, compares generated GN
arguments with `args.gn`, canonicalizes the stripped library's GNU build ID
from the shipped ELF content, and performs full-file SHA-256 verification.
Exact cache hits do not synchronize, configure, compile, or link. For local
validated fork checkouts, set both `DENIAL_FLUTTER_SOURCE_ROOT` and
`DENIAL_SKIA_SOURCE_ROOT` to absolute paths.

`build_analyze_snapshot=true` enables the LoongArch64 snapshot-analysis target
provided by the fork, and `host_cpu="loong64"` ensures that GN uses the native
LoongArch64 sysroot and toolchain. The generated engine must report AOT mode
and export `FlutterEngineGetProcAddresses`,
`FlutterEngineRunsAOTCompiledDartCode`, and
`DenialFlutterEngineScheduleFrameForExternalTextures`.

Linux builds enable Flutter's Fontconfig backend. The shipped engine therefore
requires `libfontconfig.so.1` and resolves fonts through the host's Fontconfig
configuration.

The coupled performance and hardware results are retained in the
[engine validation report](../../../docs/flutter-engine/3.44.7/VALIDATION.md).

## Licensing

Flutter Engine is BSD 3-Clause. The coupled Flutter and third-party licenses
are recorded in `../linux-loong64-release/` and included by the development
package.
