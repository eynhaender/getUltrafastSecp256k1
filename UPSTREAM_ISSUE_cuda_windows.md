# Upstream issue report — CUDA/GPU path is not Windows-clean (vc145 / VS 2026)

**Repo:** shrec/UltrafastSecp256k1  **Branch:** main (4.3.0, tip `0e20ccb`)
**Reporter:** eynhaender (getUltrafastSecp256k1 NuGet packaging)
**Context:** building the NuGet package with `-DSECP256K1_BUILD_CUDA=ON` (CUDA 13.3)
on Windows. Host toolchain VS 2022 (v143) and VS 2026 (v145). These are
**Windows-cleanliness** defects in the GPU/CUDA code path — they reproduce on any
MSVC, not just v145; the pure-CPU build never includes the offending headers so
they have gone unnoticed.

---

## Bug 1 — identifier `small` collides with the Windows SDK macro `#define small char`

**File:** `src/cuda/include/secp256k1.cuh`
**Function:** `field_mul_small` (around line 1234/1241)

```cpp
__device__ inline void field_mul_small(const FieldElement* a, uint32_t small, FieldElement* r) {
    ...
    tmp[i] = muladd64(a->limbs[i], static_cast<uint64_t>(small), 0, carry);
}
```

The Windows SDK header `rpcndr.h` (pulled in transitively on the GPU host
compile, e.g. via `<windows.h>`/CUDA runtime headers through nvcc's host side)
contains `#define small char` (and `#define hyper __int64`). The token `small`
is therefore macro-expanded to the type keyword `char`, so the cast becomes
`static_cast<uint64_t>(char)`:

```
secp256k1.cuh(1234): error : invalid combination of type specifiers
secp256k1.cuh(1241): error : type name is not allowed
            tmp[i] = muladd64(a->limbs[i], static_cast<uint64_t>(char), 0, carry);
```

Only hit on the GPU/CUDA path because the C-ABI lib `ufsecp_static` compiles
`src/gpu/src/gpu_backend_cuda.cu`, whose host preprocessing includes the Windows
SDK headers. The CPU-only build never includes that chain.

**Suggested fix (any of):**
- rename the parameter `small` → e.g. `factor` (done locally in our packaging
  patch), or
- `#undef small` / `#undef hyper` in a GPU compat header (e.g. `gpu_compat.h`)
  after the system includes, to make the whole GPU TU robust to these macros.

`hyper` is not currently used as an identifier, but the same hazard applies — a
defensive `#undef` of both is the most future-proof.

---

## Bug 2 — `src/cuda` bench/test executables fail to device-link on Windows

When `SECP256K1_BUILD_CUDA=ON`, `src/cuda/CMakeLists.txt` adds many **executables**
(`secp256k1_cuda_bench`, `bench_snark_witness`, `gpu_audit_runner`,
`bench_compare`, `gpu_bench_unified`, `test_ct_smoke`, `bench_zk`, …)
unconditionally — they are **not** gated by `SECP256K1_BUILD_TESTS` /
`SECP256K1_BUILD_BENCH` / `BUILD_TESTING`. Several fail at device-link with
unresolved kernel symbols, e.g.:

```
bench_cuda.obj : error LNK2019: unresolved external symbol
  "void __cdecl secp256k1::cuda::ecdsa_sign_batch_kernel(...)" ...
secp256k1_cuda_bench.exe : fatal error LNK1120: 7 unresolved externals
```

The **library** target `secp256k1_cuda_lib` builds fine; only these executables
fail. A redistributable/library build cannot opt out of them.

**Suggested fix:** gate the `src/cuda` executables behind
`SECP256K1_BUILD_TESTS` / `SECP256K1_BUILD_BENCH` (as the CPU and bridge trees
do), so `-DSECP256K1_BUILD_TESTS=OFF -DSECP256K1_BUILD_BENCH=OFF` yields a
library-only CUDA build.

(Workaround on our side: build only the library targets
`fastsecp256k1 ufsecp_static secp256k1_cuda_lib`.)

---

## Note — `secp256k1_cuda_lib` separable compilation hurts external consumers

`secp256k1_cuda_lib` is forced `CUDA_SEPARABLE_COMPILATION ON` (`-rdc=true`), both
via the global default (`CMakeLists.txt` ~line 28) and the target property
(~line 54-55). RDC is appropriate for the project's own multi-target GPU builds,
but it makes the **static lib** hard to consume from a non-CMake / non-CUDA
project: the relocatable device code requires the consumer to run a device-link
step (`nvcc -dlink`) and link `cudadevrt`, else `__cudaRegisterLinkedBinary_*`
unresolved. `secp256k1_cuda_lib` is a single translation unit, so it links its
device code whole-program just fine with RDC **off**.

**Suggested:** expose an option (e.g. `SECP256K1_CUDA_LIB_SELF_CONTAINED`) to build
`secp256k1_cuda_lib` with `CUDA_SEPARABLE_COMPILATION OFF`, so a plain consumer
can link `secp256k1_cuda` + `cudart` directly. (Our packaging patches the target
property OFF.)

---

## Note — vc145 / VS 2026 host with CUDA 13.3

nvcc rejects MSVC 14.50 (VS 2026) as an unsupported host. Passing
`-allow-unsupported-compiler` (we set it via `NVCC_PREPEND_FLAGS`) lets nvcc
compile the engine `.cu` cleanly with the v145 host. Not an upstream bug —
recorded for completeness. CUDA 13.x also drops Maxwell (`sm_52`); minimum is
`sm_75`.
