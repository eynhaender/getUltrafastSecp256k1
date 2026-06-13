# Changes

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
