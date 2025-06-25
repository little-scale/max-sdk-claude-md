# Quick Reference

Essential commands, templates, and decision matrices for Max SDK development.

## External Type Decision Matrix

| **Use Case** | **Type** | **Directory** | **Base** | **Headers** | **Reference** |
|--------------|----------|---------------|----------|-------------|---------------|
| Audio effects/generators | Audio | `source/audio/` | `t_pxobject ob` | `ext.h`, `z_dsp.h` | `pinknoise~/` |
| Data processing/AI | Basic | `source/basics/` | `t_object ob` | `ext.h`, `ext_obex.h` | `latenttable/` |
| Video/matrix processing | Matrix | `source/matrix/` | Dual structure | `jit.common.h` | `jit.simple/` |
| Custom GUI elements | UI | `source/ui/` | `t_jbox` | `jgraphics.h` | `uisimp/` |
| Multi-channel audio | MC | `source/mc/` | `t_pxobject ob` | `ext.h`, `z_dsp.h` | `mc.pack~/` |
| Threading/collections | Advanced | `source/advanced/` | `t_object ob` | `ext.h`, `ext_systhread.h` | `threadpool/` |
| General utilities | Misc | `source/misc/` | `t_object ob` | `ext.h` | `buddy/` |
| OpenGL/3D graphics | GL | `source/gl/` | Dual structure | `jit.gl.h` | `jit.gl.cube/` |

## Essential Commands

### Setup Check
```bash
ls source/max-sdk-base/script/  # Verify SDK base exists
```

### Build Universal Binary (Recommended)
```bash
cd source/audio/myexternal~
mkdir build && cd build
cmake -DCMAKE_OSX_ARCHITECTURES="x86_64;arm64" ..
cmake --build .
```

### Build All Externals (Bulk Development)
```bash
# Build everything from root
mkdir build && cd build
cmake -DCMAKE_OSX_ARCHITECTURES="x86_64;arm64" ..
cmake --build . -j4

# Verify all built externals
ls ../externals/*.mxo
```

### Verify Build
```bash
ls ../../../externals/myexternal.mxo
file ../../../externals/myexternal.mxo/Contents/MacOS/myexternal
# Should show: Mach-O universal binary with 2 architectures

# Codesign for M1/M2 Macs
codesign --force --deep -s - ../../../externals/myexternal.mxo
```

## Templates

### CMakeLists.txt Template
```cmake
cmake_minimum_required(VERSION 3.19)
include(${CMAKE_CURRENT_SOURCE_DIR}/../../max-sdk-base/script/max-pretarget.cmake)
include_directories("${MAX_SDK_INCLUDES}" "${MAX_SDK_MSP_INCLUDES}")
file(GLOB PROJECT_SRC "*.h" "*.c" "*.cpp")
add_library(${PROJECT_NAME} MODULE ${PROJECT_SRC})
include(${CMAKE_CURRENT_SOURCE_DIR}/../../max-sdk-base/script/max-posttarget.cmake)
```

### Audio External Template
```c
typedef struct _myobject {
    t_pxobject ob;          // REQUIRED: Audio object base
    double parameter;       // Your parameters
} t_myobject;

// REQUIRED functions:
void *myobject_new(t_symbol *s, long argc, t_atom *argv);
void myobject_free(t_myobject *x);
void myobject_dsp64(t_myobject *x, t_object *dsp64, short *count, 
                    double samplerate, long maxvectorsize, long flags);
void myobject_perform64(t_myobject *x, t_object *dsp64, double **ins, 
                        long numins, double **outs, long numouts, 
                        long sampleframes, long flags, void *userparam);
```

## Development Workflow

### Standard Development Process
1. **Choose External Type**: Use the decision matrix above to determine category (audio/basic/matrix/ui/etc.)
2. **Create Directory**: `mkdir source/[category]/myexternal/` 
3. **Setup Build**: Copy CMakeLists.txt template and modify PROJECT_NAME
4. **Implement External**: Start with minimal structure, add features incrementally
5. **Build & Test**: Use universal binary build commands, verify with `file` command
6. **Code Signing**: Apply ad-hoc codesigning for M1/M2 Mac compatibility
7. **Help File**: Create corresponding `.maxhelp` file in `help/` directory

### Multi-External Development Commands
```bash
# Build everything at once (recommended for SDK development)
mkdir build && cd build
cmake -DCMAKE_OSX_ARCHITECTURES="x86_64;arm64" ..
cmake --build . -j4

# Verify all externals built successfully  
ls ../externals/*.mxo | wc -l  # Count built externals

# Bulk codesign all externals (M1/M2 Macs)
find ../externals -name "*.mxo" -exec codesign --force --deep -s - {} \;
```

### Testing & Validation Workflow
```bash
# Test specific external
file externals/myext.mxo/Contents/MacOS/myext  # Check universal binary
otool -L externals/myext.mxo/Contents/MacOS/myext  # Check dependencies

# Integration testing in Max
# 1. Load external in Max patch
# 2. Check for errors in Max console  
# 3. Test all inlets/outlets and parameters
# 4. Verify help file loads and examples work
```

### Project Organization Best Practices
- **Single Responsibility**: Each external should have one clear purpose
- **Consistent Naming**: Follow Max conventions (`external~` for audio, `external` for basic)
- **Universal Builds**: Always build for both x86_64 and arm64 architectures  
- **Help Documentation**: Every external needs a corresponding `.maxhelp` file
- **Incremental Development**: Start simple, add complexity gradually
- **Real-Time Safety**: Audio externals must never allocate memory in perform routines

## Quick Command Reference

### Most Common Development Commands
```bash
# Setup new external
mkdir source/audio/myext~
cd source/audio/myext~
# Copy CMakeLists.txt template, edit PROJECT_NAME

# Build single external  
mkdir build && cd build
cmake -DCMAKE_OSX_ARCHITECTURES="x86_64;arm64" ..
cmake --build .

# Build all externals (bulk development)
cd /Users/a1106632/Documents/Max\ 8/Packages/max-sdk-main
mkdir build && cd build  
cmake -DCMAKE_OSX_ARCHITECTURES="x86_64;arm64" ..
cmake --build . -j4

# Verify and codesign
file ../externals/myext.mxo/Contents/MacOS/myext
codesign --force --deep -s - ../externals/myext.mxo

# Clean rebuild when needed
rm -rf build && mkdir build && cd build
```

### Essential File Locations
- **Source Code**: `source/[category]/[external]/[external].c`
- **Build Config**: `source/[category]/[external]/CMakeLists.txt`  
- **Output**: `externals/[external].mxo`
- **Help Files**: `help/[external].maxhelp`
- **Core SDK**: `source/max-sdk-base/`

### Learning Path & Examples
- **Start Here**: `source/audio/pinknoise~/` (minimal audio external)
- **Neural Networks**: `source/basics/latenttable/` (PyTorch → C++ integration)
- **Advanced Audio**: `source/audio/opuscodec~/` (real-time codec with ring buffers)
- **User Interface**: `source/ui/uisimp/` (custom GUI elements)
- **Multi-Inlet Patterns**: `source/audio/ramplfo~/` (6-inlet lores~ pattern)
- **C++ Integration**: `source/audio/tide~/` (C++ wrapper pattern)

---

*Quick reference for Max SDK development - essential commands and templates.*