# webOS cross-compile benchmarks

This repository compares GitHub-hosted x86-64 and Arm64 runners as host machines for building Qt6.11. Both jobs cross-compile for the same `arm-webos-linux-gnueabi` target (32-bit Arm, Cortex-A53/Thumb); the matrix changes the build host, not the target architecture.

## Workflow

[`webos-static-lto-bench.yml`](.github/workflows/webos-static-lto-bench.yml):

- builds the native Qt host tools and one static target Qt installation with LTO;
- builds a static FFmpeg 8.1 installation with LTO;
- records timings, caches dependency prefixes, and uploads logs and packaged prefixes.

## Observed results

GitHub Actions run [29030109815](https://github.com/sachk/actions-ffmpeg-qt-crosscompile-benchmarks/actions/runs/29030109815), 2026-07-09:

| Host runner              | Qt `all` time | FFmpeg time | Outcome                               |
| ------------------------ | ------------: | ----------: | ------------------------------------- |
| `ubuntu-24.04` x86-64    |     2:16:09.4 |      29.0 s | Success                               |
| `ubuntu-24.04-arm` Arm64 |     2:20:15.7 |           — | Qt succeeded; FFmpeg configure failed |

The Qt timer wraps one `build-qt6-611.sh all` invocation. That invocation runs two sequential phases: a native host Qt/tooling build, then **one** static target Qt build. It is not two static target builds. Across the two matrix jobs, those independent host-plus-target sequences ran concurrently.

In this single run, x86-64 finished the Qt sequence 246.2 seconds (4:06.2, 2.93%) sooner.

## Arm64 failure

The Arm64 host used the `webosbrew/native-toolchain` Arm64 SDK (GCC 12.2). Its target sysroot's `<assert.h>` is an old C99-era header and does not define the C11 `static_assert` macro. FFmpeg 8.1 probes both `static_assert` and `_Static_assert`, then reports the generic `Compiler lacks support for C11 static assertions` error when that probe fails.

The x86-64 host used the newer `openlgtv/buildroot-nc4` SDK (GCC 14.2), and the same FFmpeg configuration succeeded. Use the x86-64 job for the practical build: it was slightly faster, produces the same webOS target architecture, and currently has the compatible SDK. An Arm64-host comparison would require a newer/matching Arm64 SDK sysroot.
