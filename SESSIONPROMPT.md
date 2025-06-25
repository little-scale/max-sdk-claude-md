# Claude Code Sonnet Prompt: cycle.fold~ Max External

## Task: Create a Max MSP audio-rate external called cycle.fold~

This external should be an oscillator with wave folding and warping capabilities. Please create a complete Max external in C following the Max SDK conventions.

## Specifications

### Object name: `cycle.fold~`

### Inlets: 3
1. **Inlet 1:** Frequency (float/int/signal) - oscillator frequency in Hz
2. **Inlet 2:** Fold amount (float/signal, 0-1) - wave folding intensity where 0 = pure sine, 1 = maximum folding
3. **Inlet 3:** Warp amount (float/signal, -1 to 1) - horizontal wave warping where -1 = squished left, 0 = symmetrical, 1 = squished right

### Outlets: 1
- Audio signal output

## Technical Implementation Details

### 1. Phase Accumulator
Use a phasor-style phase accumulator that wraps from 0 to 1:

```c
double phase;  // member variable
double phase_increment = frequency / samplerate;
phase += phase_increment;
if (phase >= 1.0) phase -= 1.0;
```

### 2. Wave Generation Pipeline
- Start with a sine wave: `sin(phase * TWOPI)`
- Apply horizontal warping FIRST (phase distortion)
- Apply vertical folding SECOND (amplitude folding)

### 3. Horizontal Warping (Phase Distortion)

```c
// Example warping function - transforms phase before sine calculation
double warp_phase(double phase, double warp_amount) {
    if (warp_amount > 0) {
        // Squish to right: use exponential curve
        return pow(phase, 1.0 / (1.0 + warp_amount * 2.0));
    } else if (warp_amount < 0) {
        // Squish to left: use inverse exponential
        double abs_warp = -warp_amount;
        return 1.0 - pow(1.0 - phase, 1.0 / (1.0 + abs_warp * 2.0));
    }
    return phase;  // No warping when warp_amount == 0
}
```

### 4. Wave Folding Implementation

```c
// Apply wave folding to sine wave output
double fold_wave(double input, double fold_amount) {
    if (fold_amount <= 0) return input;
    
    // Scale up the sine wave based on fold amount
    double scaled = input * (1.0 + fold_amount * 4.0);
    
    // Fold the wave when it exceeds [-1, 1]
    while (scaled > 1.0 || scaled < -1.0) {
        if (scaled > 1.0) {
            scaled = 2.0 - scaled;
        } else if (scaled < -1.0) {
            scaled = -2.0 - scaled;
        }
    }
    
    return scaled;
}
```

### 5. Structure Definition

```c
typedef struct _cyclefold {
    t_pxobject x_obj;        // Must be first - contains t_object + MSP data
    double phase;            // Current phase (0-1)
    double frequency;        // Current frequency
    double fold_amount;      // Current fold amount (0-1)
    double warp_amount;      // Current warp amount (-1 to 1)
    double sr;               // Sample rate
    long vs;                 // Vector size
} t_cyclefold;
```

### 6. Key Functions to Implement
- `cyclefold_new()` - Constructor, set up inlets/outlets
- `cyclefold_free()` - Destructor
- `cyclefold_dsp64()` - DSP method setup
- `cyclefold_perform64()` - Audio processing function
- `cyclefold_float()` - Handle float messages to inlets
- `cyclefold_int()` - Handle int messages to inlet 1
- `cyclefold_assist()` - Inlet/outlet assistance strings

### 7. DSP Processing Loop Example

```c
void cyclefold_perform64(t_cyclefold *x, t_object *dsp64, double **ins, long numins, 
                         double **outs, long numouts, long sampleframes, long flags, void *userparam) {
    double *in_freq = ins[0];
    double *in_fold = ins[1];
    double *in_warp = ins[2];
    double *out = outs[0];
    
    double phase = x->phase;
    double sr = x->sr;
    
    for (long i = 0; i < sampleframes; i++) {
        // Get current parameters
        double freq = in_freq[i];
        double fold = CLAMP(in_fold[i], 0.0, 1.0);
        double warp = CLAMP(in_warp[i], -1.0, 1.0);
        
        // Apply warping to phase
        double warped_phase = warp_phase(phase, warp);
        
        // Generate sine wave
        double sine = sin(warped_phase * TWOPI);
        
        // Apply folding
        double output = fold_wave(sine, fold);
        
        // Output
        out[i] = output;
        
        // Update phase
        phase += freq / sr;
        if (phase >= 1.0) phase -= 1.0;
    }
    
    x->phase = phase;
}
```

### 8. Class Registration

```c
void ext_main(void *r) {
    t_class *c = class_new("cycle.fold~", (method)cyclefold_new, (method)cyclefold_free,
                          sizeof(t_cyclefold), NULL, A_GIMME, 0);
    
    class_addmethod(c, (method)cyclefold_dsp64, "dsp64", A_CANT, 0);
    class_addmethod(c, (method)cyclefold_assist, "assist", A_CANT, 0);
    class_addmethod(c, (method)cyclefold_float, "float", A_FLOAT, 0);
    class_addmethod(c, (method)cyclefold_int, "int", A_LONG, 0);
    
    class_dspinit(c);
    class_register(CLASS_BOX, c);
    cyclefold_class = c;
}
```

## Important Considerations

### 1. Antialiasing
The wave folding process creates harmonics. Consider implementing oversampling or band-limiting for production use.

### 2. Parameter Smoothing
For float/int inputs, implement parameter smoothing to avoid clicks:

```c
// Smooth parameter changes over ~10ms
double smooth_param(double current, double target, double smooth_factor) {
    return current + (target - current) * smooth_factor;
}
```

### 3. Inlet Proxy Handling
Use proxy inlets for proper message routing:

```c
// In constructor
x->x_obj.z_misc = Z_NO_INPLACE;  // Force separate in/out buffers
dsp_setup((t_pxobject *)x, 3);   // 3 signal inlets
outlet_new(x, "signal");          // 1 signal outlet
```

### 4. Error Handling
Validate all inputs and handle edge cases (e.g., negative frequencies, NaN values).

### 5. Optimization
Consider using lookup tables for sine calculation if CPU usage is critical.

## Final Instructions

Please create the complete external with:
- All necessary includes (#include statements)
- Proper Max SDK conventions
- Clear comments explaining the implementation
- Complete implementation of all functions
- Proper memory management
- Thread-safe code where applicable

The external should compile cleanly and work reliably in Max MSP for audio-rate processing with the specified wave folding and warping behavior.