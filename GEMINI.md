# llvm-custom

A specialized build system and collection of patches for building LLVM on diverse platforms (Bionic, musl, BSD, and Windows). It leverages GitHub Actions for automated building and uses Zig as a cross-compiler to produce static binaries.

## Project Overview

- **Purpose:** Provide custom-patched LLVM toolchains, particularly for environments like Termux or other musl-based systems.
- **Key Technologies:**
  - **LLVM:** The core compiler infrastructure.
  - **Zig:** Used as the C/C++ compiler toolchain (via `zig-as-llvm`) for cross-compilation and static linking.
  - **GitHub Actions:** Orchestrates the build, patching, and release process.
  - **Android NDK:** Source of the base LLVM variant and initial set of patches.

## Repository Structure

- `patches/global/llvm/<version>/`: Patches applied to LLVM regardless of the target libc.
- `patches/musl/llvm/<version>/`: musl-specific LLVM patches.
- `patches/musl/zig/`: Modifications to Zig's internal libc (musl) source, often used to adjust standard paths for restricted environments.
- `.github/workflows/`: CI workflows for different target platforms (e.g., `build_llvm_musl.yml`).

## Building and Running

The primary way to use this project is through GitHub Actions.

### Triggering a Build

1. Go to the "Actions" tab in the GitHub repository.
2. Select one of the "Build LLVM" workflows (e.g., `Build LLVM (musl libc)`).
3. Click "Run workflow" and provide the following inputs:
   - **NDK Release version:** (e.g., `30`)
   - **NDK Release revision:** (e.g., `-beta1`)
   - **Projects to build:** Semicolon-separated list of LLVM subprojects (e.g., `bolt;clang;clang-tools-extra;lld`).

### Build Process (Internal)

If you wish to replicate the build locally, the general steps are:
1. **Setup Toolchain:** Install `cmake`, `ninja`, `xz`, and a recent version of `zig`.
2. **Download LLVM:** Use the logic in the workflows to download the Android LLVM project source matching the specified NDK version.
3. **Apply Patches:**
   - Apply standard Android patches from the `llvm_android` repository.
   - Apply global patches from `patches/global/llvm/`.
   - Apply libc-specific patches from `patches/<libc>/llvm/`.
4. **Prepare Zig:** If building for musl, apply the contents of `patches/musl/zig/` to your Zig installation's `lib/libc/musl` directory.
5. **Build Dependencies:** Build `zlib` and `zstd` statically using Zig.
6. **Build LLVM:** Use `cmake` with the `Ninja` generator, pointing to the Zig wrapper as the C/C++ compiler and setting `LLVM_BUILD_STATIC=ON`.

## Development Conventions

- **Patch Organization:** Always organize new patches by LLVM version and target libc in the `patches/` directory.
- **Naming:** Follow the naming convention `Project-Short-Description.patch` (e.g., `BOLT-32-bits-fix.patch`).
- **Zig Toolchain:** The build system relies heavily on `zig-as-llvm` for consistent cross-compilation results.
