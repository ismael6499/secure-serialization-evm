# 🧶 EVM Serialization Primitives: ABI Encoding Patterns

![Solidity](https://img.shields.io/badge/Solidity-0.8.24-363636?style=flat-square&logo=solidity)
![EVM](https://img.shields.io/badge/Topic-Low_Level_Bytes-red?style=flat-square)
![Gas](https://img.shields.io/badge/Gas-Packed_vs_Standard-green?style=flat-square)

A technical reference implementation exploring the nuances of data serialization on the Ethereum Virtual Machine.

This project isolates the mechanisms of **ABI Encoding** (`abi.encode`) versus **Packed Encoding** (`abi.encodePacked`), providing a benchmark for developers to understand the trade-offs between cryptographic integrity (collision resistance) and gas efficiency when preparing data for hashing (`keccak256`) or external calls.

## 🏗 Architecture & Design Decisions

### 1. Serialization Strategy (Standard vs. Packed)
- **Standard Encoding (`abi.encode`):**
  - Utilized for inter-contract calls to strictly adhere to the ABI specification, ensuring 32-byte padding for static types.
  - **Use Case:** Validated its necessity for preventing **Hash Collisions** when dealing with dynamic types (e.g., `(string, string)`), where packed encoding fails to distinguish boundaries.
- **Packed Encoding (`abi.encodePacked`):**
  - Implemented for specific gas-optimization scenarios where data layout compactness outweighs ABI compliance.
  - **Optimization:** Demonstrates significant gas reduction by removing padding zeroes, critical for generating cheap commitments or signatures within the contract logic.

### 2. Cryptographic Integrity
- **Collision Analysis:**
  - The repository includes test cases specifically designed to demonstrate how `abi.encodePacked("a", "bc")` produces the exact same hash as `abi.encodePacked("ab", "c")`.
  - **Security Pattern:** This serves as a negative test case to justify the architectural decision of using `abi.encode` for hashing distinct dynamic parameters, a common vulnerability in signature verification schemas.

### 3. String & Bytes Manipulation
- **Type Casting:**
  - Demonstrates the low-level conversion of `string` to `bytes` for raw manipulation. This is foundational for understanding how high-level Solidity types translate to EVM memory allocation.

## 🧪 Testing Strategy (Foundry)

The test suite validates the binary output of the encoding functions:

- **Byte-Level Assertions:**
  - Unlike standard logic tests, this suite asserts the exact hexadecimal output of the encoding functions to verify padding behavior.
- **Collision Fuzzing:**
  - Validates that `encodePacked` produces identical byte streams for ambiguous inputs, reinforcing the security warnings documented in the codebase.

## 🛠 Tech Stack

* **Core:** Solidity `^0.8.24`
* **Focus:** `abi.encode`, `abi.encodePacked`, `keccak256`
* **Tooling:** Foundry (Cast/Forge)

## 📝 Serialization Interface

The contract exposes primitives to inspect EVM memory layout:

```solidity
// Returns 32-byte padded data (Safe for hashing dynamic types)
function encodeStandard(uint256 number, address addr) external pure returns (bytes memory) {
    return abi.encode(number, addr);
}

// Returns minimal byte stream (Gas efficient, collision prone)
function encodePacked(uint256 number, address addr) external pure returns (bytes memory) {
    return abi.encodePacked(number, addr);
}
```
