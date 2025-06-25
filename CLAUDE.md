# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture Overview

This is the **Max Software Development Kit** for building external objects (plug-ins) for Cycling '74's Max/MSP/Jitter environment. The codebase is organized as:

### Core Architecture
- **Max/MSP/Jitter Integration**: Externals integrate with Max's real-time audio/visual environment through standardized object models
- **Multi-Language Support**: C/C++ with optional Python integration for neural networks  
- **Universal Binary Builds**: Cross-platform compilation for x86_64 and ARM64 architectures
- **CMake Build System**: Automated project generation and dependency management
- **Modular External Types**: Audio (MSP), Matrix (Jitter), UI, Basic data processing, and specialized categories

### Key Patterns & Design Principles
1. **Object-Oriented C**: Max externals use C structs as objects with function pointers for methods
2. **Real-Time Safety**: Audio processing must avoid memory allocation, file I/O, and blocking operations
3. **Signal/Float Duality**: The "lores~ pattern" enables seamless switching between signal and float inputs
4. **Universal Binary Target**: All builds produce fat binaries supporting both Intel and Apple Silicon Macs
5. **External Type Specialization**: Different base classes for audio (`t_pxobject`), basic (`t_object`), UI (`t_jbox`), and matrix processing

### Directory Structure Logic
```
source/
├── audio/          # MSP~ audio externals (real-time signal processing)
├── basics/         # Basic data processing, neural networks, utilities  
├── matrix/         # Jitter matrix/video processing externals
├── ui/             # Custom user interface elements
├── mc/             # Multi-channel audio processing
├── gl/             # OpenGL/3D graphics externals
├── advanced/       # Threading, collections, system integration
├── misc/           # General utilities and helper objects
├── dictionary/     # Dictionary/data structure manipulation
└── patcher/        # Patcher environment interaction
```

### Build System Architecture
- **Root CMakeLists.txt**: Auto-discovers and builds all external directories with CMakeLists.txt files
- **max-sdk-base/**: Core SDK providing headers, libraries, and build scripts
- **Universal Binary Default**: Automatic x86_64+arm64 compilation on supported systems
- **Output Centralization**: All built externals go to top-level `externals/` directory

---

## Documentation Structure

This documentation is split into focused files for better organization:

- **[Quick Reference](docs/QUICK_REFERENCE.md)** - Essential commands, templates, and decision matrices
- **[Development Patterns](docs/DEVELOPMENT_PATTERNS.md)** - Critical patterns, bug fixes, and advanced techniques
- **[Project History](docs/PROJECT_HISTORY.md)** - Completed projects and session summaries
- **[Troubleshooting Guide](docs/TROUBLESHOOTING.md)** - Build issues, runtime problems, and debugging

---

## Session Prompt
For the current session, please see the document named: SESSIONPROMPT.md

**Current Session**: ✅ **COMPLETED** `cycle.fold~` - Professional wave folding oscillator with progressive threshold folding and click-free parameter transitions for Max/MSP. Successfully implemented enhanced version of session prompt specifications with major improvements: lores~ pattern for perfect signal/float dual input on all 3 inlets, progressive threshold wave folding with reflection algorithm (fold=0→pure sine, fold=0.5→mid-level folding, fold=1→maximum folding), click-free parameter transitions through adaptive smoothing and continuous algorithm paths, always-on DC blocking for consistent processing, musical exponential phase warping curves (1-4 range), anti-aliasing protection with frequency-dependent processing, denormal protection, and phase synchronization with bang messages. **CRITICAL BREAKTHROUGH**: Click-free parameter transitions - eliminated all clicking artifacts when transitioning between fold amounts (especially to/from fold=0) through continuous algorithm paths, always-on DC blocking, and adaptive smoothing for both signal and float inputs. **NEW DISCOVERIES**: Progressive threshold mapping provides intuitive wave folding control, continuous algorithm execution prevents switching artifacts, adaptive smoothing rates optimize both responsiveness and click prevention, always-on processing eliminates conditional switching that causes audio discontinuities, and true reflection-based wave folding creates authentic analog-style folding characteristics.

Do not remove this section of this CLAUDE.md.

---

## Environment Notes

**This Installation**: `/Users/a1106632/Documents/Max 8/Packages/max-sdk-main/`
- **Platform**: M2 MacBook Air (ARM64 + x86_64 universal binaries required)
- **Repository Status**: Not a git repository - manual max-sdk-base submodule
- **Build Output**: `externals/[external].mxo` (centralized output directory)
- **Max Version**: 8.2.0+ compatibility (see package-info.json)

### Repository Navigation & File Patterns
```
max-sdk-main/
├── source/[category]/[external]/    # Source code + CMakeLists.txt
├── externals/                       # Built .mxo files (output)
├── help/                           # .maxhelp documentation files  
├── docs/                           # Documentation files (this split)
├── source/max-sdk-base/            # Core SDK headers & build scripts
├── projects/                       # Complex multi-file projects
├── build/                          # Build directory (created by user)
└── CMakeLists.txt                  # Root build configuration
```

**File Naming Conventions**:
- **Audio externals**: `myext~.c` → `myext~.mxo` → `myext~.maxhelp`
- **Basic externals**: `myext.c` → `myext.mxo` → `myext.maxhelp`  
- **Matrix externals**: `jit.myext.c` + `max.jit.myext.c` → `jit.myext.mxo`
- **CMake projects**: Each external directory contains `CMakeLists.txt`

### SDK Integration Notes  
- **Max Package Location**: Must be in `~/Documents/Max 8/Packages/` for Max to discover externals
- **Universal Binary Requirement**: Both Intel and Apple Silicon Macs require fat binaries
- **Codesigning Requirement**: M1/M2 Macs require ad-hoc codesigning for external loading
- **Auto-Discovery**: Root CMakeLists.txt automatically finds and builds all external directories
- **Hot Reload**: Max can reload externals during development (close/reopen patches)

---

*Max SDK reference reorganized for clarity and maintainability.*