# Flutter Engine — Linux LoongArch64 release

`libflutter_engine.so` is Denial's optimized raw Flutter Embedder library.
The official Linux artifact is a GTK embedding library, so Denial builds and
packages this raw AOT embedder target directly for LoongArch64.

The shared library is ignored by Git. This directory tracks the expected
checksum, GN configuration, coupled source revisions, and Flutter/third-party
license files needed to reproduce and redistribute the artifact.

## Source identity

[`SOURCE_LOCK.json`](../SOURCE_LOCK.json) is the sole source of truth for the
engine source. The generated `args.gn` records the Flutter, Skia, Dart, and
content identities used by GN. `ENGINE_REVISION` and `FLUTTER_REVISION` record
the upstream ABI revisions used by the embedder bindings.

The Flutter source is the locked LoongArch64-capable fork revision. Its DEPS
file pins the matching Skia fork and Dart source revisions. The prebuilt Dart
SDK is supplied at `engine/src/flutter/prebuilts/linux-loong64/dart-sdk`; it is
not bootstrapped from an unavailable LoongArch64 CIPD package.

## LoongArch64 build

The artifact is built natively on a LoongArch64 GNU/Linux host with Flutter's
clang toolchain, the locked Debian trixie sysroot, and the prebuilt Dart SDK.
The build uses the same GN/Ninja engine target as the other Linux platforms.
The LoongArch64 target arguments are required because the fork's GN wrapper
otherwise defaults the host and target CPU values to x64:

```sh
./flutter/tools/gn \
  --runtime-mode=release \
  --enable-fontconfig \
  --target-os=linux \
  --linux-cpu=loong64 \
  '--gn-args=build_analyze_snapshot=true host_cpu="loong64"' \
  --target-dir=denial_host_release
/usr/bin/ninja -C out/denial_host_release libflutter_engine.so
```

`build_analyze_snapshot=true` enables the LoongArch64 snapshot-analysis target
provided by the fork. `tools/denial-flutter-engine` supplies these arguments,
verifies the complete generated `args.gn`, canonicalizes the stripped ELF GNU
build ID, and checks the resulting file against
`libflutter_engine.so.sha256`.

Use the revision-keyed incremental builder for normal builds:

```sh
tools/denial-flutter-engine build
```

The builder hashes the source lock, every mode's GN arguments, and expected
artifact checksums. An exact cache hit skips synchronization, configuration,
compilation, and linking. On a cache miss it retains the GN/Ninja output so
subsequent builds remain incremental. When local validated fork checkouts are
used instead of a managed checkout, set both
`DENIAL_FLUTTER_SOURCE_ROOT` and `DENIAL_SKIA_SOURCE_ROOT` to absolute paths.

The generated engine must export
`FlutterEngineGetProcAddresses`,
`DenialFlutterEngineRequestFrameForExternalTextures`, and
`DenialFlutterEngineScheduleFrameForExternalTextures`.

Linux builds enable Flutter's Fontconfig backend. The shipped engine therefore
requires `libfontconfig.so.1` and resolves fonts through the host's Fontconfig
configuration rather than assuming they live below `/usr/share/fonts`.

The standard Rust ABI is generated from the pristine official embedder header
at the upstream compatibility revision:

```sh
tools/generate-flutter-embedder-bindings
tools/generate-flutter-embedder-bindings --check
```

Denial's versioned extension is loaded and typed separately in
`compositor/flutter-engine/src/lib.rs`.

## Licensing

Flutter Engine is BSD 3-Clause; bundled third-party code retains its upstream
licenses. Ship `LICENSE.flutter` and `LICENSE.third_party` with every engine
package.
