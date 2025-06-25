# Development Patterns

Critical patterns, bug fixes, and advanced techniques for Max SDK development.

## Critical Bug Fixes & Patterns

### MSP Signal/Float Inlet Conflicts (CRITICAL Discovery)

**Problem**: `dsp_setup((t_pxobject*)x, N)` creates N signal inlets where Max provides zero-signals to ALL inlets, overriding float parameter values.

**Symptoms**: Float messages appear received but external behavior never changes.

**Complete Solution Pattern (from tide~)**:
```c
// 1. Structure with connection flags and parameter storage
typedef struct _myobject {
    t_pxobject ob;
    double param1_float;        // Stored float values
    double param2_float;
    short param1_has_signal;    // Connection status flags
    short param2_has_signal;
} t_myobject;

// 2. Setup multiple signal inlets
void* myobject_new(...) {
    dsp_setup((t_pxobject*)x, N);  // N signal inlets
    // Initialize connection status to 0
    x->param1_has_signal = 0;
    x->param2_has_signal = 0;
}

// 3. Track connection status in dsp64
void myobject_dsp64(..., short* count, ...) {
    x->param1_has_signal = count[1];  // Inlet 1 connected?
    x->param2_has_signal = count[2];  // Inlet 2 connected?
}

// 4. Float handler with proxy_getinlet routing
void myobject_float(t_myobject* x, double f) {
    long inlet = proxy_getinlet((t_object*)x);
    switch (inlet) {
        case 0: x->frequency_float = CLAMP(f, 0.001, 1000.0); break;  // Inlet 0 works
        case 1: x->param1_float = CLAMP(f, 0.0, 1.0); break;         // Inlets 1+ work
        case 2: x->param2_float = CLAMP(f, 0.0, 1.0); break;
    }
}

// 5. Choose signal vs float in perform routine
void myobject_perform64(...) {
    double param1 = x->param1_has_signal ? ins[1][i] : x->param1_float;
    double param2 = x->param2_has_signal ? ins[2][i] : x->param2_float;
    // Process audio with chosen parameter values...
}
```

**Key Insights**:
- `proxy_getinlet()` correctly routes float messages to all inlets (0-N)
- `count` array in `dsp64()` indicates which inlets have signal connections
- Must check connection status per-sample to choose signal vs float input
- This pattern enables true dual signal/float inlets on all parameters
- **IMPORTANT**: Do NOT mix proxy inlets with signal inlets - use one approach or the other

### 🎯 PROVEN SOLUTION: lores~ Pattern (Multi-Inlet Signal/Float)

```c
// THE SOLUTION: Use ONLY signal inlets (like lores~) - NO proxy inlets!
dsp_setup((t_pxobject*)x, N);  // Creates N signal inlets

// Track connection status in dsp64 (like lores~)
void myobject_dsp64(..., short* count, ...) {
    x->param1_has_signal = count[1];  // Is signal connected to inlet 1?
    x->param2_has_signal = count[2];  // Is signal connected to inlet 2?
    // ... etc for all inlets
}

// Float handler uses proxy_getinlet() - works on signal inlets too!
void myobject_float(t_myobject* x, double f) {
    long inlet = proxy_getinlet((t_object*)x);  // Works perfectly!
    switch (inlet) {
        case 0: x->freq_float = f; break;      // All inlets work
        case 1: x->param1_float = f; break;    // Max handles conversion
        case 2: x->param2_float = f; break;    // No proxy inlets needed
    }
}

// Signal vs float selection in perform64 (exactly like lores~)
void myobject_perform64(...) {
    double param1 = x->param1_has_signal ? *ins[1]++ : x->param1_float;
    double param2 = x->param2_has_signal ? *ins[2]++ : x->param2_float;
    // Perfect signal/float dual input on ALL inlets
}
```

**ramplfo~ Implementation (6-Inlet Audio-Rate Pattern)**:
```c
// All 6 inlets accept both signals and floats automatically (lores~ pattern)
dsp_setup((t_pxobject*)x, 6);  // Creates 6 signal inlets, NO proxy inlets

// Store connection status in dsp64
x->freq_has_signal = count[0];
x->shape_has_signal = count[1];
x->rise_has_signal = count[2];
x->fall_has_signal = count[3];
x->jitter_has_signal = count[4];
x->phase_has_signal = count[5];

// Choose signal vs float per parameter in perform64
double freq = x->freq_has_signal ? *freq_in++ : x->freq_float;
double shape = x->shape_has_signal ? *shape_in++ : x->shape_float;
double rise_curve = x->rise_has_signal ? *rise_in++ : x->rise_curve_float;
double fall_curve = x->fall_has_signal ? *fall_in++ : x->fall_curve_float;
double jitter_amount = x->jitter_has_signal ? *jitter_in++ : x->jitter_float;
double phase_offset = x->phase_has_signal ? *phase_in++ : x->phase_offset_float;
```

## Parameter Safety Patterns

### Division by Zero Prevention
```c
// DANGEROUS: Allows 0.0 which crashes in division
x->character = CLAMP(character, 0.0, 1.0);

// SAFE: Minimum above zero for divisor parameters  
x->character = CLAMP(character, 0.01, 1.0);
```

### "Brighter Wins" Logic (from vactrol~)
```c
// Use fmin() for resistance - lower resistance = brighter
x->resistance = fmin(envelope_resistance, cv_resistance);
```

### Parameter Responsiveness Enhancement
```c
// Mathematically enhance subtle effects
if (x->pole_count == 2) {
    effective_cutoff = cutoff * 0.8;  // 20% reduction for audible difference
}
```

### Click-Free Parameter Transitions
```c
// Adaptive smoothing: heavy for float inputs, light for signal inputs
if (x->fold_has_signal) {
    fold_amount = smooth_param(x->fold_smooth, fold_target, smooth_factor * 0.1);
} else {
    fold_amount = smooth_param(x->fold_smooth, fold_target, smooth_factor);
}

// Always apply processing for consistency (prevents on/off clicks)
double output = dc_block(x, folded_wave);

// Continuous algorithm paths eliminate switching artifacts
```

## Advanced Patterns

### Ring Buffer Pattern (Frame-based Processing)
```c
typedef struct _ring_buffer {
    float *buffer;
    int write_pos, read_pos, size, available;
} t_ring_buffer;
// Eliminates clicking artifacts in codec externals
```

### Neural Network Integration (PyTorch → Max)
1. Train in PyTorch → 2. Extract weights → 3. Pure C++ inference → 4. Max external
```cpp
typedef struct _yournet {
    t_object ob;                    // Basic object (NOT t_pxobject)  
    YourNetwork *network;           // C++ neural network class
} t_yournet;
```

### Threading Safety
```c
// NEVER in perform routine: malloc, outlets, post(), file I/O, mutex locks
// Safe: Pure computation, cached parameters, atomic reads
void myobject_perform64(...) {
    double param = x->parameter;  // Cache once per vector
    for (int i = 0; i < sampleframes; i++) {
        outs[0][i] = process_sample(ins[0][i], param);  // Pure computation
    }
}
```

### Multi-Inlet CV/Envelope Control
```c
// Dual control sources: envelope (bang-triggered) + CV signal
double envelope_resistance = calculate_envelope_resistance(x);
double cv_resistance = calculate_cv_resistance(cv_input);
x->resistance = fmin(envelope_resistance, cv_resistance);  // "Brighter wins"
```

### Advanced Waveshaping Patterns (from tide~)
```c
// Multi-table lookup with shape morphing
double shape_index = shape_param * (NUM_SHAPES - 1);
int shape_a = (int)shape_index;
int shape_b = shape_a + 1;
double shape_mix = shape_index - shape_a;

// Interpolate between lookup tables
double val_a = x->shape_lut[shape_a][index] * (1.0 - fract) + 
               x->shape_lut[shape_a][index + 1] * fract;
double val_b = x->shape_lut[shape_b][index] * (1.0 - fract) + 
               x->shape_lut[shape_b][index + 1] * fract;
double result = val_a * (1.0 - shape_mix) + val_b * shape_mix;
```

### Progressive Threshold Wave Folding
```c
// Progressive threshold: fold=0→threshold=1.0, fold=1→threshold=0.01
double threshold = 1.0 - safe_fold_amount * 0.99;

// Reflection-based folding when signal exceeds thresholds
while (output > threshold || output < -threshold) {
    if (output > threshold) {
        output = 2.0 * threshold - output;  // Reflect down
    } else if (output < -threshold) {
        output = -2.0 * threshold - output;  // Reflect up
    }
}
```

### C++ Integration Pattern (from tide~)
```c
// Main external structure with opaque pointer
typedef struct _tide {
    t_pxobject ob;
    void* poly_slope_generator;    // Opaque pointer to C++ object
    // Max-specific parameters and state...
} t_tide;

// Forward declarations for C++ wrapper functions
extern "C" {
    void* tides_create(void);
    void tides_destroy(void* obj);
    void tides_render(void* obj, /* parameters */);
}

// Object lifecycle
void* tide_new(...) {
    x->poly_slope_generator = tides_create();  // Create C++ object
    if (!x->poly_slope_generator) return NULL; // Handle failure
}

void tide_free(t_tide* x) {
    if (x->poly_slope_generator) {
        tides_destroy(x->poly_slope_generator);  // Clean C++ object
    }
}
```

### Bang Message Routing
```c
// Bang handler with inlet detection
void myobject_bang(t_myobject* x) {
    long inlet = proxy_getinlet((t_object*)x);
    switch (inlet) {
        case 0: 
            x->reset_phase = 1;  // Frequency inlet: reset phase
            post("myobject~: phase reset");
            break;
        case 1:
            x->trigger_envelope = 1;  // Shape inlet: trigger envelope
            break;
        // Handle other inlets as needed
    }
}

// In perform routine: check and clear flags
void myobject_perform64(...) {
    for (long i = 0; i < sampleframes; i++) {
        if (x->reset_phase) {
            reset_dsp_state(x);
            x->reset_phase = 0;  // Clear flag
        }
        // Continue processing...
    }
}
```

### Double Precision Frequency Handling
```c
// Structure with double precision for ultra-slow frequencies
typedef struct _myobject {
    t_pxobject ob;
    double frequency_float;      // High precision frequency storage
    double freq_scale;          // Scaling factor for sub-Hz rates
} t_myobject;

// In perform routine: double precision phase accumulation
void myobject_perform64(...) {
    double current_freq = x->freq_has_signal ? freq_in[i] : x->frequency_float;
    current_freq *= x->freq_scale;  // Apply scaling
    
    // Normalize to phase increment per sample (keep double precision)
    double norm_frequency = current_freq / x->sample_rate;
    
    // C++ object handles double precision internally
    cpp_set_frequency(x->cpp_object, norm_frequency);
}
```

### Attribute Implementation Pattern
```c
// In ext_main() after class_new()
CLASS_ATTR_DOUBLE(c, "freqscale", 0, t_myobject, freq_scale);
CLASS_ATTR_FILTER_MIN(c, "freqscale", 0.0001);
CLASS_ATTR_FILTER_MAX(c, "freqscale", 1.0);
CLASS_ATTR_DEFAULT(c, "freqscale", 0, "1.0");
CLASS_ATTR_LABEL(c, "freqscale", 0, "Frequency Scale");
CLASS_ATTR_SAVE(c, "freqscale", 0);

// In object constructor
void* myobject_new(...) {
    x->freq_scale = 1.0;  // Initialize before attr_args_process
    attr_args_process(x, argc, argv);  // Process attributes from arguments
}
```

### A_GIMME List Message Pattern (from harmosc~)
```c
// Function declaration for variable-length list messages
void myobject_amps(t_myobject *x, t_symbol *s, long argc, t_atom *argv);

// Register in ext_main()
class_addmethod(c, (method)myobject_amps, "amps", A_GIMME, 0);

// Implementation handles flexible list input
void myobject_amps(t_myobject *x, t_symbol *s, long argc, t_atom *argv) {
    if (argc == 0) {
        object_error((t_object *)x, "amps: requires at least one value");
        return;
    }
    
    // Process each value in the list
    int num_values = MIN(argc, x->max_values);
    for (int i = 0; i < num_values; i++) {
        double value = 0.0;
        
        // Handle both float and int atoms
        if (atom_gettype(argv + i) == A_FLOAT) {
            value = atom_getfloat(argv + i);
        } else if (atom_gettype(argv + i) == A_LONG) {
            value = (double)atom_getlong(argv + i);
        } else {
            object_error((t_object *)x, "amps: argument %d is not a number", i + 1);
            continue;
        }
        
        // Clamp and store value
        value = CLAMP(value, 0.0, 1.0);
        x->values[i] = value;
    }
    
    // Handle remaining values if fewer provided
    for (int i = num_values; i < x->max_values; i++) {
        x->values[i] = 0.0;  // Default remaining to 0
    }
    
    post("myobject~: %d values set", num_values);
}

// Usage: [amps 1.0 0.5 0.3 0.8] sets first 4 values
```

## Advanced Integration Patterns

### Python Neural Networks (from `latenttable`, `euclid-space-mlp`)

**Workflow**: PyTorch → Weight Export → C++ Inference → Max External

```cpp
// Pattern from latenttable and rhythm externals
typedef struct _mynet {
    t_object ob;                    // Basic object base (NOT t_pxobject)
    MyNetwork *network;             // C++ inference class
    float *weights;                // Pre-loaded binary weights
} t_mynet;

// Implementation steps:
// 1. Train in PyTorch
// 2. Export weights with convert_pytorch_weights.py  
// 3. Load binary weights in C++
// 4. Implement pure C++ inference class
// 5. Wrap in Max external
```

### Multi-Language Projects

**C++ Classes with Max Wrapper** (from `tide~`):
```c
// C interface for C++ DSP classes
extern "C" {
    void* tides_create(void);
    void tides_destroy(void* obj);
    void tides_render(void* obj, /* parameters */);
}

// Max external structure
typedef struct _tide {
    t_pxobject ob;
    void* cpp_dsp_object;    // Opaque pointer to C++ object
} t_tide;

// Constructor
void *tide_new(...) {
    x->cpp_dsp_object = tides_create();
}

// Destructor  
void tide_free(t_tide *x) {
    if (x->cpp_dsp_object) {
        tides_destroy(x->cpp_dsp_object);
    }
}
```

**C++ Wrapper Implementation**:
```cpp
// tides_wrapper.cpp - separate C++ file
extern "C" {
    void* tides_create(void) {
        return new TidesClass();
    }
    
    void tides_destroy(void* obj) {
        delete static_cast<TidesClass*>(obj);
    }
}
```

### Testing & Validation

**Build Verification**:
```bash
# Check universal binary
file externals/myext.mxo/Contents/MacOS/myext
# Should show: Mach-O universal binary with 2 architectures

# Test external loading
# Load in Max, check for errors in console
```

**Help File Validation**:
- Help files should be in `help/[external].maxhelp`
- Test all inlets and parameters
- Include audio examples for audio externals
- Verify assist strings match actual inlets

---

*Development patterns for Max SDK - critical discoveries and advanced techniques.*