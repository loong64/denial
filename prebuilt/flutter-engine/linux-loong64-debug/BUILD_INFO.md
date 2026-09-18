# Flutter Engine — Linux LoongArch64 debug/JIT

`libflutter_engine.so` is the JIT-capable LoongArch64 engine used by
`denial-ui-development` for live shell development, hot reload, Flutter
Inspector integration, and the authenticated loopback VM service. Native
engine code remains optimized; only the Dart runtime mode is `develop`.

The shared library is ignored by Git. This directory tracks the generated GN
arguments, coupled source revisions, checksum, and the license files needed by
the development package.

## Source identity

The authoritative source input is
[`SOURCE_LOCK.json`](../SOURCE_LOCK.json). The generated `args.gn` and the
adjacent revision files record the derived build and ABI identities without
duplicating mutable lock values in this document.

The Flutter fork's DEPS file pins the matching Skia fork and Dart source. The
LoongArch64 prebuilt Dart SDK is installed at
`engine/src/flutter/prebuilts/linux-loong64/dart-sdk` and is passed to GN as a
prebuilt SDK; the build does not attempt to download the unavailable official
LoongArch64 CIPD package.

## LoongArch64 build

Build natively on a LoongArch64 GNU/Linux host with the fork's clang toolchain
and Debian trixie sysroot:

```sh
./flutter/tools/gn \
  --runtime-mode=debug \
  --enable-fontconfig \
  --target-os=linux \
  --linux-cpu=loong64 \
  '--gn-args=build_analyze_snapshot=true host_cpu="loong64"' \
  --target-dir=denial_host_debug
/usr/bin/ninja -C out/denial_host_debug libflutter_engine.so
```

`build_analyze_snapshot=true` selects the LoongArch64 snapshot-analysis target
implemented by the fork, while `host_cpu="loong64"` makes GN resolve the
native sysroot and toolchain correctly. `tools/denial-flutter-engine` supplies
these arguments, compares the complete generated arguments with `args.gn`,
canonicalizes the stripped ELF GNU build ID, and verifies
`libflutter_engine.so.sha256`.

The canonical incremental builder is:

```sh
tools/denial-flutter-engine build
```

Exact cache hits skip source synchronization, GN, Ninja, and linking. Source
changes retain `out/denial_host_debug`, allowing Ninja to rebuild only
invalidated targets. If local validated fork checkouts are used, set both
`DENIAL_FLUTTER_SOURCE_ROOT` and `DENIAL_SKIA_SOURCE_ROOT` to absolute paths.

The engine must remain JIT-capable and export
`FlutterEngineGetProcAddresses`,
`FlutterEngineRunsAOTCompiledDartCode`, and
`DenialFlutterEngineScheduleFrameForExternalTextures`.

Linux builds enable Flutter's Fontconfig backend. The shipped engine therefore
requires `libfontconfig.so.1` and resolves fonts through the host's Fontconfig
configuration.

## Licensing

Flutter Engine is BSD 3-Clause and bundled third-party components retain their
upstream licenses. The development package includes the release engine's
Flutter and third-party license material together with the pinned Dart SDK
license.
