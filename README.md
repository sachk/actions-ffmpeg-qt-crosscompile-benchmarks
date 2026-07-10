# webOS cross-compile benchmarks

This repository compares GitHub-hosted x86-64 and Arm64 runners as host machines for building Qt6.11. Both jobs cross-compile for the same `arm-webos-linux-gnueabi` target (32-bit Arm, Cortex-A53/Thumb); the matrix changes the build host, not the target architecture.

## Workflow

[`webos-static-lto-bench.yml`](.github/workflows/webos-static-lto-bench.yml):

- builds the native Qt host tools and one static target Qt installation with LTO;
- builds a static FFmpeg 8.1 installation with LTO;
- records timings, caches dependency prefixes, and uploads logs and packaged prefixes.

## Corrected results

GitHub Actions run [29063959878](https://github.com/sachk/actions-ffmpeg-qt-crosscompile-benchmarks/actions/runs/29063959878), 2026-07-10, used the matching `openlgtv/buildroot-nc4` `webos-a38c582` SDK (GCC 14.2) on both host architectures:

| Host runner              | Qt `all` time | FFmpeg time | Outcome |
| ------------------------ | ------------: | ----------: | ------- |
| `ubuntu-24.04` x86-64    |     2:12:36.6 |      29.0 s | Success |
| `ubuntu-24.04-arm` Arm64 |     2:18:59.1 |      20.5 s | Success |

The Qt timer wraps one `build-qt6-611.sh all` invocation. That invocation runs two sequential phases: a native host Qt/tooling build, then **one** static target Qt build. It is not two static target builds. Across the two matrix jobs, those independent host-plus-target sequences ran concurrently.

In this single run, x86-64 finished the Qt sequence 382.5 seconds (6:22.5, 4.59%) sooner. Arm64 built FFmpeg 8.5 seconds faster, but FFmpeg took less than 30 seconds on either host. x86-64 remains the faster practical choice for the full build; both hosts now work and produce the same target architecture.

## SDK correction

The first Arm64 run accidentally used an older `webosbrew/native-toolchain` SDK (GCC 12.2), while x86-64 used the newer `openlgtv/buildroot-nc4` SDK. The old target sysroot's `<assert.h>` did not define the C11 `static_assert` macro, causing FFmpeg's generic `Compiler lacks support for C11 static assertions` configure failure.

The Arm64 matrix entry now uses the matching `arm-webos-linux-gnueabi_sdk-buildroot-aarch64.tar.gz` asset from the same `webos-a38c582` release as x86-64. The successful corrected run confirms that the original failure was an SDK mismatch, not an Arm64-runner limitation.
