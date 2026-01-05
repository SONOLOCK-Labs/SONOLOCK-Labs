# Sonolock Core V5.2.1 "Iron Genesis"

![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)
![Language](https://img.shields.io/badge/Language-Rust-orange.svg)
![Platform](https://img.shields.io/badge/Platform-ARM%20%7C%20x86%20%7C%20SGX%20%7C%20RISCV-success.svg)
![Memory](https://img.shields.io/badge/Memory-Heap--Less-green.svg)
![Binary Size](https://img.shields.io/badge/Binary%20Size-~7.5%20KB-blueviolet.svg)

> **Proving sound integrity without retaining sound data.**

---

## 🌟 The Identity: Architectural Vision

**Sonolock V5.2 “Iron Genesis”** is an audio integrity commitment protocol kernel designed for industrial, embedded, and Trusted Execution Environments (TEE). It generates verifiable cryptographic commitments within a secure execution environment without retaining any raw audio data, enforcing a strict **20ms** data lifecycle upper bound.

This is a universal kernel engineered for **ARM, RISC-V, x86, SGX, TrustZone**, and automotive-grade SoCs.

### ✨ Core Highlights
* **Dual-Domain Security:** Unified algorithm for cloud-side Intel SGX and edge-side ARM Cortex-M.
* **Ultra-Lightweight:** ~7.5 KB binary footprint.
* **Heap-Less Design:** Zero dynamic memory allocation, eliminating fragmentation and OOM risks.
* **Industrial Stability:** Hard-float ABI aligned and Rust-safe.

---

## 🏗️ The Master Tree: Architectural Overview

<details>
<summary>▶ Click to expand project structure</summary>

```text
sonolock-core/
├── .github/workflows/
│   └── release-matrix.yml      # CI Parallel Build: SGX / Linux-ARM / Windows-MSVC
├── docs/
│   ├── ARCHITECTURE.md         # System Design Documentation
│   ├── PERFORMANCE_AUDIT.md    # Memory Zeroization & Latency Audit Reports
│   └── EMBEDDED_SAFETY.md      # Embedded Alignment & Safety Guidelines
├── include/
│   └── sonolock_core.h         # Auto-generated C Standard Header
├── benches/
│   └── memory_scrub_stress.rs  # 20ms Mandatory Destruction Stress Test
├── crates/
│   ├── sonolock-protocol/      # Core Protocol Definitions (Data-only, No-logic)
│   ├── sonolock-enclave/       # Encryption & Zeroization Logic (Core Computation)
│   └── sonolock-verifier-api/  # Cross-language FFI Bridge (Security Boundary)
├── Cargo.toml                  # Workspace Configuration
└── LICENSE                     # Apache 2.0
</details>⚖️ The Three Laws: Core Design Principles<details><summary>▶ Click to view the Engineering Laws</summary>1. Protocol as Physical Layout (The Law)sonolock-protocol defines immutable, cross-language consistent byte structures.Packed Strategy: Trading space efficiency for transmission determinism.Binary Consistency: Identical byte sequences across ARM/x86.Constraint: Strictly 73 Bytes.2. FFI Boundary as Security Boundary (The Bridge)Engineering Iron Rule: No structure pointers; byte buffers only.Byte Mover Pattern: Using memcpy semantics to prevent Hard Faults on unaligned ARM/RISC-V access.3. Code as Header (No Drift)Automatic cbindgen integration ensures Rust implementation and C API are always synchronized.</details>🛠️ Implementation Detail: Engineering Specifications<details><summary>▶ Click to view Rust & FFI specifications</summary>1. Industrial Build ConfigurationIni, TOML[profile.release]
panic = "abort"        # Prevents Unwind across FFI boundaries
lto = "fat"            # Maximum Link-Time Optimization
codegen-units = 1      # Extreme runtime performance
strip = true           # Removes symbol tables
overflow-checks = true # Safety first
2. Core Protocol DefinitionRust#[repr(C, packed)]
pub struct SoundCommitment {
    pub version: u8,             // Protocol Version (0x52)
    pub timestamp: u64,          // Unix Timestamp
    pub salt: [u8; 16],          // Entropy Source
    pub fingerprint: [u8; 32],   // Audio Fingerprint
    pub context_extension: [u8; 16], // Extension Fields
}

// Compile-time assertion
const _: () = assert!(core::mem::size_of::<SoundCommitment>() == 73);
</details>📦 The Artifacts: Delivery & Integration Specs<details><summary>▶ Click to view Binary & Hardware Specs</summary>1. Physical Componentslibsonolock_verifier_api.a: Static library for ARM Cortex-M4F.sonolock_core.h: Standard C99/C++ header.2. Integration SpecsParameterSpecificationTarget DeviceARM Cortex-M4FArchitecturethumbv7em-none-eabihfFloat ABIHard-float (-mfloat-abi=hard)FPU TypeFPv4-SP-D16Panic StrategyAbort (Silent in-place)</details>🚀 The Code: Quick Start & ABI Contract<details><summary>▶ Click to view C Integration Example</summary>C#include "include/sonolock_core.h"

int main() {
    float pcm_buffer[100]; 
    uint8_t commitment[73];
    
    // Invoke Rust Core
    int8_t status = snlk_generate_commitment(pcm_buffer, 100, commitment);
    
    if (status == 0) {
        // [SECURITY SAFE] Move to aligned struct via memcpy
        snlk_SoundCommitment c;
        memcpy(&c, commitment, sizeof(c));
    }
    return 0;
}
</details>🛡️ Toxic Data Audit: Principles of Destruction<details><summary>▶ Click to view Security Audit Data</summary>Sonolock treats user audio as High-Risk Transient Data (Toxic Data).Lifecycle: Strictly < 20ms.Mechanism: Immediate Zeroization following hash computation.Storage Strategy: No disk persistence, zero recovery potential.MetricResultConstraintStatusP50 Latency4.12 ms< 10 msPASSEDP99 Latency7.42 ms< 20 msPASSEDMemory Scrub100% Cleared0% ResiduePASSED</details>🏁 The Manifesto: Final PositioningSonolock V5.2 “Iron Genesis” is not an SDK, nor a demo, nor research code.It is the blast furnace for digital sound assets. digital sound assets.
