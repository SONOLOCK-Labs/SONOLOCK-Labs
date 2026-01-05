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
* **Dual-Domain Security:** A unified algorithm supporting both cloud-side Intel SGX TEE verification and edge-side ARM Cortex-M real-time generation.
* **Ultra-Lightweight:** Core binary footprint is approximately **7.5 KB**, ensuring minimal Flash consumption.
* **Heap-Less Design:** Full-stack computation without dynamic memory allocation (`malloc`/`free`), completely eliminating memory fragmentation and OOM risks.
* **Industrial Stability:** Verified through Rust strong-typing safety and Hard-float ABI physical alignment, eliminating "Exit Code 101" linking errors.

---

## 🏗️ The Master Tree: Architectural Overview

A standard, auditable Rust Workspace featuring built-in C-ABI auto-generation and cross-platform build matrices.

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
⚖️ The Three Laws: Core Design PrinciplesV5.2 establishes three non-negotiable constraints at the engineering level:1. Protocol as Physical Layout (The Law)sonolock-protocol defines immutable, cross-language consistent byte structures.Packed Strategy: Deliberately uses packed structures to trade space efficiency for transmission determinism.Binary Consistency: Ensures identical byte sequences across any platform (ARM/x86).Constraint: Strictly 73 Bytes. Direct field access on unaligned memory is prohibited.2. FFI Boundary as Security Boundary (The Bridge)sonolock-verifier-api provides the sole legal C-ABI entry point.Engineering Iron Rule: Passing structure pointers is prohibited; only byte buffers are allowed.Byte Mover Pattern: Core export functions use memcpy semantics for cross-boundary transfers, fundamentally preventing Hard Faults on ARM Cortex-M and RISC-V due to unaligned access.3. Code as Header (No Drift)Every cargo build automatically triggers cbindgen to generate include/sonolock_core.h. This ensures the Rust implementation and C API remain perfectly synchronized.🛠️ Implementation Detail: Engineering Specifications1. Industrial Build ConfigurationEnforces "Panic Abort" and "Fat LTO" to ensure minimal artifacts without Unwind risks.Ini, TOML[profile.release]
panic = "abort"        # Prevents Unwind across FFI boundaries to avoid UB
lto = "fat"            # Enables maximum Link-Time Optimization
codegen-units = 1      # Sacrifice compile time for extreme runtime performance
strip = true           # Removes symbol tables to prevent reverse engineering
overflow-checks = true # Safety first: checks for overflows even in Release mode
2. Core Protocol DefinitionImmutable data contract, strictly locked at 73 bytes.Rust#[repr(C, packed)]
pub struct SoundCommitment {
    pub version: u8,             // Protocol Version (0x52)
    pub timestamp: u64,          // Unix Timestamp
    pub salt: [u8; 16],          // Entropy Source
    pub fingerprint: [u8; 32],   // Audio Fingerprint
    pub context_extension: [u8; 16], // Extension Fields
}

// Compile-time assertion: ensure size is strictly 73 bytes
const _: () = assert!(core::mem::size_of::<SoundCommitment>() == 73);
📦 The Artifacts: Delivery & Integration Specs1. Physical Componentslibsonolock_verifier_api.a: Static library optimized for ARM Cortex-M4F.sonolock_core.h: Standard C interface definition, compatible with C99/C++.2. Integration SpecsParameterSpecificationTarget DeviceARM Cortex-M4FArchitecturethumbv7em-none-eabihfFloat ABIHard-float (-mfloat-abi=hard)FPU TypeFPv4-SP-D16Panic StrategyAbort (Silent in-place)🚀 The Code: Quick Start & ABI ContractC Integration ExampleC#include "include/sonolock_core.h"

int main() {
    float pcm_buffer[100]; // Fill with your audio samples
    uint8_t commitment[73];
    
    // Invoke the Rust Core Algorithm
    int8_t status = snlk_generate_commitment(pcm_buffer, 100, commitment);
    
    if (status == 0) {
        // [SECURITY SAFE] Must move to aligned struct via memcpy before access
        snlk_SoundCommitment c;
        memcpy(&c, commitment, sizeof(c));
        // printf("Timestamp: %llu\n", c.timestamp);
    }
    return 0;
}
⚠️ The Iron Standard: Embedded Integration Rules[!IMPORTANT]DSP & Embedded Format CompatibilityIn automotive SoCs, source audio is often int16_t. The kernel exclusively uses Float32. The caller is responsible for normalizing int16 data to float (range -1.0 to 1.0) before invocation.✅ Correct Flow: Allocate uint8_t buffer[73] -> Call Interface -> memcpy to local aligned struct.❌ Forbidden Action: Directly casting pointers ((snlk_SoundCommitment*)raw) will cause Hard Faults on ARM.🛡️ Toxic Data Audit: Principles of DestructionSonolock treats user audio as High-Risk Transient Data (Toxic Data).Lifecycle: Strictly < 20ms.Mechanism: Immediate Zeroization following hash computation.Storage Strategy: No disk persistence, no caching, zero recovery potential.MetricResultConstraintStatusP50 Latency4.12 ms< 10 msPASSEDP99 Latency7.42 ms< 20 msPASSEDMemory Scrub100% Cleared0% ResiduePASSED🏁 The Manifesto: Final PositioningSonolock V5.2 “Iron Genesis” is not an SDK, nor a demo, nor research code.It is the blast furnace for digital sound assets.
