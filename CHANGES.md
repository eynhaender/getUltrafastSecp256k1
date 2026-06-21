# Changes

## 4.4.0.1 (2026-06-21)

- vc145 rebuilt from **dev 4.4.0** (tip `d300e8cb`; engine + bridge updates incl.
  the libbitcoin batch cancellation token). CPU `static`/`ltcg`/`dynamic` + the
  self-contained `cuda` linkage (no `-rdc`, `cudart_static` → consumer links with
  plain MSVC, runs with only an NVIDIA GPU + driver). Vectors green on both
  variants; `/GL` gate correct.

## 4.3.0.3 (2026-06-17)

- CUDA lib now built **self-contained** (`CUDA_SEPARABLE_COMPILATION OFF`, no `-rdc`):
  a consumer links `secp256k1_cuda` + `cudart` with a plain MSVC link — no device-link
  step / `cudadevrt` / CUDA-aware project required. (4.3.0.2's RDC lib needed the
  consumer to device-link.) Single-TU lib, so whole-program device linking is fine.

## 4.3.0.2 (2026-06-17)

- Optional **CUDA** support for the vc145 package: `build.ps1 -Cuda [-CudaArchitectures 89]`
  builds a Release `secp256k1_cuda` lib in its own decoupled build dir and adds a
  `cuda` linkage (CPU static libs + `secp256k1_cuda` + `cudart`). Requires the CUDA
  Toolkit; nvcc runs against the v145 host via `-allow-unsupported-compiler`
  (`NVCC_PREPEND_FLAGS`) with the toolkit dir passed as `-T v145,cuda=<dir>` and the
  VS CUDA MSBuild integration copied into VS 2026's BuildCustomizations.
- CPU `static` / `ltcg` / `dynamic` variants unchanged; CUDA is decoupled so the
  CPU libs stay clean and the Debug-CUDA upstream issues are avoided.
- Local patch for an upstream Windows bug (`small` identifier vs the Windows
  `#define small char` macro) — see `UPSTREAM_ISSUE_cuda_windows.md`.
- Built CPU-only by default (`build.ps1 -Pack`); CUDA is opt-in.

## 4.3.0.1 (2026-06-17)

- vc145 retargeted to **main 4.3.0** (the libbitcoin shim + bridge fixes from the
  4.1.1.x dev line are now merged into main; default source branch back to `main`)
- Same package shape: static + ltcg variants, x64, Release/Debug; CPU only
  (CUDA not included — needs the CUDA Toolkit + a vc143/VS2022 host; nvcc does
  not support vc145/VS2026)
- Also available as the multi-toolset pipeline: `-Toolset vc143` builds the
  VS 2022 equivalent

## 4.1.1.7 (2026-06-13)

- Rebuilt from dev tip `2d58f70`, incl. shim fix `684141e` (key ECDSA cache by full pubkey)

## 4.1.1.6 (2026-06-13)

- Default source branch switched to `dev` (carries the libbitcoin shim + bridge fixes; `main` lacks them)
- Bridge batching fixes from dev: row + column high-S ECDSA normalize (`acbf…`, `91ae9ba`)
- libbitcoin `sign` / `encode_signature` vectors verified on both variants

## 4.1.1.4 (2026-06-12)

- Prefixed bridge include `"ufsecp/ufsecp_error.h"`; natural `include/ufsecp/` subdir layout (single `include\` root)

## 4.1.1.3 (2026-06-12)

- Switch to dev shim: `secp256k1_*` C ABI compiled into fastsecp256k1 (`BUILD_SHIM=ON`); opaque ECDSA signature layout fix (little-endian, matches upstream)
- Two static variants: `static` (`/O2`, no `/GL`, clean consumer link) + `ltcg` (`/O2 /GL`, link `/LTCG`); `.static.lib` / `.ltcg.lib` names; libs under `build/native/bin/`

## 4.1.1.2 (2026-06-11)

- Compile the libsecp256k1 shim into fastsecp256k1 so `secp256k1_*` symbols link (was: headers only)
- Fix bridge static define: `UFSECP_STATIC_LIB` (not `UFSECP_STATIC`) — drops `__imp_ufsecp_*`

## 4.1.1.1 (2026-06-05)

- Switch source branch from dev to main (official 4.1.1 release)
- 4-part versioning: {source_major}.{source_minor}.{source_patch}.{packaging_revision}
- Lib names embed all 4 digits: fastsecp256k1-x64-vc145-mt-s-4_1_1_0.lib

## 4.1.0 (2026-06-03)

- Initial release based on UltrafastSecp256k1 v4.1.0 (shrec/dev)
- Toolset: Visual Studio 2026 / MSVC vc145
- Platforms: x64, x86
- Configurations: Release, Debug
- Link types: static (default), shared
