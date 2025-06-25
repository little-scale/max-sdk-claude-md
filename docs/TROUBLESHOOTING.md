# Troubleshooting Guide

Build issues, runtime problems, and debugging techniques for Max SDK development.

## Build Issues

### Missing max-sdk-base
**Problem**: CMake can't find SDK headers or build scripts
**Solution**: 
```bash
git submodule update --init --recursive
# OR manual download from GitHub if not a git repository
```

### External Won't Load
**Problem**: Max shows "couldn't load" error for external
**Solution**: M1/M2 Macs require codesigning
```bash
codesign --force --deep -s - external.mxo
```

### CMake Errors
**Problem**: CMake configuration fails or generates wrong files
**Solution**: Use Unix Makefiles (default), not Xcode generator
```bash
# If using wrong generator, delete build directory and regenerate
rm -rf build && mkdir build && cd build
cmake -DCMAKE_OSX_ARCHITECTURES="x86_64;arm64" ..
```

### Architecture Issues
**Problem**: External only builds for one architecture
**Solution**: Ensure universal binary flag is specified
```bash
cmake -DCMAKE_OSX_ARCHITECTURES="x86_64;arm64" ..
```

### Missing Headers Error
**Problem**: `error: 'MaxAudioAPI.h' file not found`
**Solution**: Check max-sdk-base submodule exists and is populated
```bash
ls source/max-sdk-base/c74support/max-includes/
# Should contain headers like MaxAudioAPI.h, ext.h, etc.
```

### Wrong Architecture
**Problem**: External builds but won't load on M1/M2 Macs
**Solution**: Verify universal binary with file command
```bash
file externals/myext.mxo/Contents/MacOS/myext
# Should show: Mach-O universal binary with 2 architectures
```

### CMake Generator Mismatch
**Problem**: Build system becomes inconsistent
**Solution**: Delete build directory and regenerate with correct generator
```bash
rm -rf build && mkdir build && cd build
cmake -DCMAKE_OSX_ARCHITECTURES="x86_64;arm64" ..
cmake --build .
```

## Runtime Issues

### Parameters Not Working
**Problem**: Float messages received but external behavior never changes
**Solution**: MSP signal/float conflict - implement lores~ pattern
```c
// Track connection status in dsp64
x->param_has_signal = count[1];

// Choose signal vs float in perform64
double param = x->param_has_signal ? ins[1][i] : x->param_float;
```

### Division by Zero Crashes
**Problem**: External crashes when parameter reaches exactly 0.0
**Solution**: Clamp parameters above zero for divisor parameters
```c
// DANGEROUS: Allows 0.0 which crashes in division
x->character = CLAMP(character, 0.0, 1.0);

// SAFE: Minimum above zero for divisors
x->character = CLAMP(character, 0.01, 1.0);
```

### Audio Clicking/Gaps
**Problem**: Clicking artifacts or audio dropouts
**Solution**: Implement ring buffer system for frame-based processing
```c
typedef struct _ring_buffer {
    float *buffer;
    int write_pos, read_pos, size, available;
} t_ring_buffer;
```

### Attributes Not Saving
**Problem**: Object attributes don't persist in patches
**Solution**: Use message handlers instead of attributes, or ensure proper attribute setup
```c
// Alternative: Use message handlers instead of attributes
class_addmethod(c, (method)myobject_param, "param", A_FLOAT, 0);
```

### Too Many Inlets Showing
**Problem**: External shows more inlets than expected
**Solution**: Remove unnecessary `proxy_new()` calls
```c
// WRONG: Creates extra visible inlets
dsp_setup((t_pxobject*)x, 2);  // 2 signal inlets
inlet_new(x, "float");         // Extra inlet - avoid this

// RIGHT: Use only signal inlets with lores~ pattern
dsp_setup((t_pxobject*)x, 3);  // 3 signal inlets, handles floats too
```

### LFO Inlet Confusion
**Problem**: Users confused by inlet order
**Solution**: Put frequency on inlet 0, not trigger - matches user expectations
```c
// Preferred: frequency first (like cycle~)
// Inlet 0: frequency, Inlet 1: shape, Inlet 2: other params
```

### External Loads But No Audio
**Problem**: External appears to work but produces no sound
**Solution**: Verify `dsp_setup()` inlet count matches actual implementation
```c
// Ensure inlet count matches what perform64 expects
dsp_setup((t_pxobject*)x, 2);  // Must match ins[] array usage
```

### Segfaults on Load
**Problem**: Max crashes when loading external
**Solution**: Check object lifecycle (constructor/destructor), memory allocation/deallocation
```c
void* myobject_new(...) {
    t_myobject *x = (t_myobject*)object_alloc(myobject_class);
    if (x) {
        // Initialize ALL structure members
        x->parameter = 0.0;
        x->buffer = NULL;  // Important for cleanup
    }
    return x;
}

void myobject_free(t_myobject *x) {
    if (x->buffer) {
        free(x->buffer);  // Clean up allocated memory
    }
    dsp_free((t_pxobject*)x);
}
```

### Max Console Errors
**Problem**: Errors appear in Max console when using external
**Solution**: Check for typos in method registration and function signatures
```c
// Ensure method names and signatures match exactly
class_addmethod(c, (method)myobject_float, "float", A_FLOAT, 0);
//                         ^^^^^^^^^^^^^^  must match actual function name

void myobject_float(t_myobject *x, double f);  // Signature must match A_FLOAT
```

### Click Artifacts on Parameter Changes
**Problem**: Audible clicks when changing parameters
**Solution**: Implement parameter smoothing and continuous algorithm paths
```c
// Adaptive smoothing for click prevention
if (x->param_has_signal) {
    param = smooth_param(x->param_smooth, param_target, smooth_factor * 0.1);
} else {
    param = smooth_param(x->param_smooth, param_target, smooth_factor);
}

// Always-on processing prevents switching artifacts
double output = always_process(x, input);  // Don't use conditional processing
```

## Debugging Commands

### Detailed Build Output
```bash
# Get verbose build information for troubleshooting
cmake --build . --verbose
```

### Check External Dependencies and Architecture
```bash
# Check what libraries external links against
otool -L externals/myext.mxo/Contents/MacOS/myext

# Verify universal binary architecture
file externals/myext.mxo/Contents/MacOS/myext
```

### Verify Codesigning Status
```bash
# Check codesigning information
codesign -dv externals/myext.mxo
```

### Clean Rebuild When Build Cache is Corrupted
```bash
# Nuclear option: completely clean rebuild
rm -rf build && mkdir build && cd build
cmake -DCMAKE_OSX_ARCHITECTURES="x86_64;arm64" ..
cmake --build .
```

### Find All Built Externals and Their Architectures
```bash
# Check all externals in one command
find externals -name "*.mxo" -exec sh -c 'echo "{}:" && file "$1/Contents/MacOS/"*' _ {} \;
```

### Memory Debugging
```bash
# Run Max with memory debugging (if available)
export MallocStackLogging=1
/Applications/Max.app/Contents/MacOS/Max

# Use instruments or other profiling tools
instruments -t "Allocations" -D ~/Desktop/max_memory.trace /Applications/Max.app/Contents/MacOS/Max
```

## Max Console Debugging

### Enable Max Console Logging
- **Window → Max Console**, set to verbose
- Watch for errors when loading external
- Check for method registration issues

### Check Method Registration
```c
// Ensure all class_addmethod() calls match function signatures exactly
class_addmethod(c, (method)myobject_dsp64, "dsp64", A_CANT, 0);
class_addmethod(c, (method)myobject_assist, "assist", A_CANT, 0);
class_addmethod(c, (method)myobject_float, "float", A_FLOAT, 0);
```

### Verify Assist Strings
```c
void myobject_assist(t_myobject *x, void *b, long m, long a, char *s) {
    if (m == ASSIST_INLET) {
        switch (a) {
            case 0: sprintf(s, "Frequency (Hz)"); break;
            case 1: sprintf(s, "Parameter 1"); break;
            // Ensure case count matches actual inlet count
        }
    }
}
```

### Test Inlet Functionality
```c
// Add debug posts to verify inlet routing
void myobject_float(t_myobject *x, double f) {
    long inlet = proxy_getinlet((t_object*)x);
    post("Float %.3f received on inlet %ld", f, inlet);  // Debug output
    // Remove debug posts in production
}
```

### Monitor Real-Time Audio
- Use Max's DSP status window to check for glitches/dropouts
- Monitor CPU usage in activity monitor
- Check for audio buffer underruns

## Performance Debugging

### CPU Usage Analysis
```bash
# Monitor CPU usage during Max operation
top -pid `pgrep Max`

# Use Instruments for detailed profiling
instruments -t "Time Profiler" /Applications/Max.app/Contents/MacOS/Max
```

### Memory Usage Analysis
```bash
# Check memory usage
vm_stat
# Monitor memory during Max operation
memory_pressure
```

### Audio Performance
```c
// Add timing code to perform routines (debug only)
#ifdef DEBUG
    uint64_t start_time = mach_absolute_time();
#endif

    // Your audio processing code here

#ifdef DEBUG
    uint64_t end_time = mach_absolute_time();
    double elapsed_ms = (end_time - start_time) * 1e-6;  // Convert to milliseconds
    if (elapsed_ms > 1.0) {  // Warn if processing takes > 1ms
        post("Warning: perform64 took %.2f ms", elapsed_ms);
    }
#endif
```

## Common Error Messages

### "couldn't load"
- Missing codesigning (M1/M2 Macs)
- Wrong architecture
- Missing dependencies

### "dsp method not found"
- Missing `dsp64` method registration
- Incorrect function signature

### "bad inlet"
- Mismatch between `dsp_setup()` count and actual inlet usage
- Proxy inlet configuration issues

### "out of memory"
- Memory leak in constructor/destructor
- Buffer allocation without proper cleanup

### "perform method error"
- Array bounds violations in perform routine
- Division by zero or other math errors
- Accessing NULL pointers

---

*Troubleshooting guide for Max SDK development - solutions for common issues.*