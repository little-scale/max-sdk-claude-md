# Project History

Completed projects and session summaries from Max SDK development sessions.

## Session History

**Previous Session**: ✅ **COMPLETED** `decay~` - Exponential decay envelope generator with curve shaping and configurable click-free start smoothing for Max/MSP. Successfully implemented mathematically accurate exponential decay with user-customizable smoothing to eliminate click artifacts while preserving musical expressiveness. Key achievements: exponential coefficient calculation with sample-rate independence, power function curve shaping for exponential/linear/logarithmic envelope shapes, user-configurable start smoothing (0-100 samples) as fourth creation argument, smart smoothing activation only when significant level jumps detected, lores~ 3-inlet pattern for sample-accurate time/curve control, retrigger modes (from peak or current level), and comprehensive parameter range validation. **CRITICAL BREAKTHROUGH**: User-configurable smoothing system - default 0 samples preserves original sharp behavior while allowing 1-100 samples of linear interpolation smoothing for click-free starts, giving users complete control over attack characteristics. **NEW DISCOVERIES**: Smart smoothing activation (only when level jumps > 0.001) eliminates unnecessary processing overhead, fourth creation argument pattern for optional features, and mathematical precision in exponential decay implementation using proper time constants.

**Previous Session**: ✅ **COMPLETED** `modvca~` - Make Noise MODDEMIX-style VCA external with amplitude-dependent distortion for Max/MSP. Successfully implemented smooth linear saturation transition where low levels exhibit rich harmonic content and high levels remain clean, creating musical "tail character" during signal decay. Key achievements: linear VCA response curve for predictable control, linear drive interpolation algorithm for smooth saturation transition, single-stage tanh saturation with drive compensation, lores~ 2-inlet pattern for seamless signal/float CV control, unity gain passthrough at full levels, and elimination of discontinuities in saturation behavior. **CRITICAL BREAKTHROUGH**: Linear drive interpolation method - `drive = max_drive * (1 - amplitude) + min_drive` creates perfectly smooth saturation transition without the two-portion tail behavior seen in complex multi-stage or inverse scaling approaches. **NEW DISCOVERIES**: Simple linear relationships often outperform complex mathematical models for musical control, user experience benefits from predictable CV-to-amplitude mapping, and single-stage saturation with proper drive compensation provides more musical results than multi-stage processing. 

**Previous Session**: Completed `slewenv~` (formerly `maths-envelope~`) - a Make Noise Maths-style function generator/integrator external for Max/MSP. The project demonstrates advanced MSP inlet handling, real-time parameter control, and integrator-based envelope generation with curve shaping. **CRITICAL DISCOVERY**: MSP signal/float inlet conflicts where Max provides zero-signals to all `dsp_setup()` inlets, overriding float parameter values. Successfully renamed from `maths-envelope~` to `slewenv~` with full build and documentation update.

**Previous Session**: ✅ **COMPLETED** `tide~` - Advanced Tides-inspired LFO/envelope external with comprehensive feature set. Successfully implemented C++ wrapper pattern for integrating complex DSP algorithms into Max externals. Key achievements: asymmetric ramp generator with variable slope control, shape morphing (exponential/linear/logarithmic curves), dual-mode smoothness (2-pole LPF + triangle wavefolder), three ramp modes (AD/Loop/AR), proper signal/float dual-inlet support, bang message synchronization, sub-Hz frequency support with double precision, phase offset control, and frequency scaling attribute. **NEW DISCOVERIES**: C++ integration patterns via extern "C" wrappers, bang message routing to inlets, double precision phase accumulation for ultra-slow frequencies, simplified algorithm recreation approach, and attribute implementation patterns.

**Previous Session**: ✅ **COMPLETED** `easelfo~` - Signal-rate LFO with 12 easing functions from animation/motion design industry plus advanced phase control and mirror mode. Successfully implemented function pointer arrays, 3-inlet design, and sophisticated phase manipulation. Key achievements: Linear, Sine, Quadratic, Cubic, and Exponential easing variants (In/Out/InOut), proxy inlet for glitch-free function selection, bang message for phase synchronization, signal-rate frequency input with FM support, phase offset control (third inlet), mirror mode for smooth 0→1→0 waveform motion, bipolar output (-1 to +1), and mathematical precision with proper edge case handling. **NEW DISCOVERIES**: Function pointer array patterns for zero-overhead algorithm selection, animation easing function mathematics in audio context, **proxy inlet creation order affects Max inlet assignment** (must create in reverse order), mirror mode phase transformation algorithms, and efficient phase accumulation with bidirectional wrapping.

**Previous Session**: ✅ **COMPLETED** `ramplfo~` - Advanced asymmetric ramping LFO with 6-inlet audio-rate processing, extended curve shaping families, and phase control. Successfully implemented the **lores~ pattern** for perfect multi-inlet signal/float dual processing. Key achievements: asymmetric rise/fall timing control, 5 mathematical curve families (7th power + exponential/logarithmic/S-curves) in unified -3.0 to 3.0 parameter range, organic jitter system with smoothed randomness, phase offset control, full audio-rate modulation on all 6 inlets, unipolar 0-1 output, and comprehensive musical applications. **MAJOR DISCOVERY**: **lores~ Multi-Inlet Pattern** - the definitive solution for multi-inlet signal/float handling: use ONLY `dsp_setup(x, N)` signal inlets (no proxy inlets), `proxy_getinlet()` works perfectly on signal inlets for float routing, `count` array tracks connections, perfect signal/float dual input on ALL inlets. This solves the long-standing MSP multi-inlet problem completely.

**Previous Session**: ✅ **COMPLETED** Multi-External lores~ Pattern Implementation - Successfully applied the definitive lores~ pattern to all existing externals with multi-inlet signal/float issues. **EXTERNALS FIXED**: `easelfo~` (converted from proxy pattern to 3-inlet lores~ pattern), `harmosc~` (added frequency inlet + lores~ pattern), `tide~` (already had correct pattern), `slewenv~` (converted from workaround pattern to proper lores~ pattern). All externals now support seamless signal/float dual input on ALL inlets with proper connection detection. **ACHIEVEMENT**: Standardized multi-inlet pattern across entire SDK ensures consistent behavior and eliminates long-standing MSP inlet conflicts. All externals built as universal binaries and ready for production use.

**Previous Session**: ✅ **COMPLETED** `physicslfo~` - Advanced physics-based LFO with 6 realistic simulation types and enhanced envelope mode. Successfully implemented natural physics evolution with proper settling behavior. Key achievements: 6 physics types (bounce, damped decay with ring-out, enhanced multi-frequency spin, overshoot settling, multi-bounce with complete stop, wobble settling), lores~ 4-inlet pattern for seamless signal/float handling, looping vs envelope modes with natural physics evolution, console parameter information system, phase message control, and comprehensive documentation. **CRITICAL BREAKTHROUGH**: Envelope mode natural physics evolution - removed artificial cutoffs to let physics simulations play out naturally until energy dissipates, creating realistic settling and stopping behavior. **NEW DISCOVERIES**: Physics simulation algorithms for audio (damped oscillation, multi-frequency spin, exponential energy decay), natural stop conditions for avoiding perpetual motion, and comprehensive real-time physics parameter feedback system.

**Previous Session**: ✅ **COMPLETED** `cycle-2d~` - 2D morphing wavetable oscillator with bilinear interpolation between four corner waveforms and custom buffer integration. Successfully implemented real-time 2D spatial control over timbre. Key achievements: 4096-sample wavetables for smooth interpolation, corner waveform mapping (sine, triangle, sawtooth, square at normalized coordinates), bilinear interpolation algorithm for seamless morphing, lores~ 4-inlet pattern for audio-rate position control, custom buffer loading with distance-weighted blending (up to 16 simultaneous tables), phase accumulation with bang reset, and comprehensive spatial audio documentation. **BREAKTHROUGH INNOVATION**: 2D spatial timbre control - users can intuitively navigate through timbral space using X/Y coordinates, creating new forms of musical expression through spatial metaphors. **NEW DISCOVERIES**: Bilinear interpolation for audio synthesis, distance-weighted buffer blending algorithms, spatial audio concepts applied to wavetable morphing, and 2D parameter space design for musical interfaces.

**Previous Session**: ✅ **COMPLETED** `ssm2044~` - Authentic SSM2044 analog filter emulation with zero-delay feedback topology and comprehensive analog modeling. Successfully implemented classic vintage synthesizer filter characteristics. Key achievements: zero-delay feedback (ZDF) 4-pole low-pass filter for stable self-oscillation, analog saturation modeling with tanh-based nonlinearities, authentic SSM2044 frequency response and resonance behavior, lores~ 4-inlet pattern for sample-accurate parameter modulation, multi-stage saturation throughout signal path, denormal protection and stability safeguards, and comprehensive vintage synthesizer documentation. **MAJOR BREAKTHROUGH**: Zero-delay feedback implementation - eliminates traditional recursive filter delays while maintaining perfect stability and authentic self-oscillation characteristics of classic analog hardware. **NEW DISCOVERIES**: ZDF filter topology for Max externals, analog circuit modeling techniques, progressive saturation systems, vintage synthesizer frequency response emulation, and stable self-oscillation algorithms.

## Completed Projects

| Project | Type | Status | Key Features | Why Study |
|---------|------|--------|--------------|-----------|
| **`pinknoise~`** | Audio | ✅ Complete | Basic DSP, single param | **Start here** - minimal complexity |
| **`latenttable`** | Basic | ✅ Complete | PyTorch port, C++ classes, file I/O | C++ integration patterns |
| **`vactrol~`** | Audio | ✅ Complete | Multi-inlet MSP, analog modeling, CV/envelope | **Intermediate MSP** - dual control sources |
| **`opuscodec~`** | Audio | ✅ Complete | Ring buffers, external libs, real-time codec | **Advanced** - frame-based processing |
| **`slewenv~`** | Audio | ✅ Complete + lores~ | Integrator-based envelopes, curve shaping | **MSP inlet handling** discovery |
| **`harmosc~`** | Audio | ✅ Complete + lores~ | Harmonic oscillator, wavetable synthesis, detuning, custom amplitude lists | **Multi-argument parsing** and **A_GIMME** patterns |
| **`tide~`** | Audio | ✅ Complete + lores~ | Tides-inspired LFO, asymmetric ramps, shape morphing, bang sync, sub-Hz frequencies | **C++ integration** patterns |
| **`easelfo~`** | Audio | ✅ Complete + lores~ | Animation easing functions, mirror mode, phase control, function pointer arrays | **Easing mathematics** and **proxy inlet patterns** |
| **`ramplfo~`** | Audio | ✅ Complete + lores~ | Asymmetric ramping LFO, curve families, jitter, 6-inlet design | **lores~ pattern discovery** |
| **`physicslfo~`** | Audio | ✅ Complete + lores~ | Physics-based LFO, 6 simulation types, natural settling/stopping, envelope mode evolution | **Physics simulation** and **natural evolution** |
| **`cycle-2d~`** | Audio | ✅ Complete + lores~ | 2D morphing wavetable oscillator, bilinear interpolation, custom buffer integration | **2D spatial audio** and **wavetable morphing** |
| **`ssm2044~`** | Audio | ✅ Complete + lores~ | SSM2044 analog filter emulation, zero-delay feedback, analog saturation modeling | **ZDF topology** and **analog modeling** |
| **`modvca~`** | Audio | ✅ Complete + lores~ | MODDEMIX-style VCA with amplitude-dependent distortion, linear drive interpolation, smooth saturation transition | **Linear algorithm design** and **amplitude-dependent processing** |
| **`decay~`** | Audio | ✅ Complete + lores~ | Exponential decay envelope generator with curve shaping, configurable start smoothing, retrigger modes | **Mathematical precision** and **user-configurable features** |
| **`cycle.fold~`** | Audio | ✅ Complete + lores~ | Wave folding oscillator with progressive threshold folding, click-free transitions, phase warping | **Professional audio processing** and **click prevention** |

## Development Challenges Solved

| Challenge | Solution | Key Learning |
|-----------|----------|--------------|
| **MSP Signal/Float Conflicts** | Force float values or check `count` array | Max provides zero-signals to all `dsp_setup()` inlets |
| **Division by Zero** | Clamp parameters above zero for divisors | `character` parameter crashed when exactly 0.0 |
| **Frame Boundary Clicking** | Ring buffer system from `mp3codec~` | Continuous output eliminates audio gaps |
| **Attribute System Failures** | **ABANDON** - use message handlers instead | Attributes unreliable, messages proven |
| **Parameter Responsiveness** | Mathematical enhancement (20% cutoff reduction) | Subtle effects need amplification for audibility |
| **Multi-Inlet Assist Strings** | Match inlet order carefully in `assist()` | Easy to swap inlet descriptions |
| **Inlet Configuration Design** | Frequency on inlet 0 for LFO objects | More intuitive than trigger - matches user expectations |
| **Proxy Inlet Pitfalls** | **AVOID** unnecessary `proxy_new()` calls | Creates extra visible inlets beyond `dsp_setup()` count |
| **Proxy Inlet Creation Order** | Create proxy inlets in **reverse order** for proper routing | `proxy_new()` call order affects Max inlet assignment |
| **Signal vs Proxy Inlet Choice** | Use `dsp_setup(x, N)` for true signal inlets, avoid mixing with proxies | Signal inlets accept both signals and floats automatically |
| **C++ Integration Complexity** | Use extern "C" wrapper pattern with opaque pointers | Clean language boundary, easy to maintain |
| **Bang Message Routing** | Use `proxy_getinlet()` to route bangs to specific inlets | Enables phase sync and trigger functionality |
| **Sub-Hz Frequency Support** | Double precision phase accumulation | Required for ultra-slow automation (geological time scales) |
| **Attribute System Implementation** | Use `CLASS_ATTR_*` macros with proper defaults | Better than message handlers for persistent settings |
| **Variable-Length List Messages** | Use `A_GIMME` with `argc`/`argv` parsing | Enables flexible list input like amplitude arrays |
| **Complex Algorithm Simplification** | Linear relationships often outperform complex models | Simple math provides more predictable musical control |
| **Smooth Saturation Transitions** | Linear drive interpolation instead of inverse scaling | Eliminates discontinuities and two-portion behavior |
| **Unity Gain Passthrough** | Proper drive compensation and minimal saturation at full levels | Critical for authentic VCA behavior |
| **User-Configurable Features** | Optional parameters via creation arguments with sensible defaults | Preserves simplicity while enabling advanced control |
| **Smart Processing Activation** | Conditional expensive operations only when needed | Eliminates overhead for unused features |
| **Mathematical Precision** | Accurate exponential decay with proper time constants | Sample-rate independent and mathematically correct |
| **Click-Free Parameter Transitions** | Continuous algorithm paths, adaptive smoothing, always-on processing | Eliminates switching artifacts and audio discontinuities |
| **Progressive Threshold Wave Folding** | Reflection-based folding with intuitive parameter mapping | True analog-style wave folding characteristics |

## Key Discoveries & Breakthroughs

### lores~ Multi-Inlet Pattern (MAJOR DISCOVERY)
The definitive solution for multi-inlet signal/float handling discovered in `ramplfo~` development:
- Use ONLY `dsp_setup(x, N)` signal inlets (no proxy inlets)
- `proxy_getinlet()` works perfectly on signal inlets for float routing
- `count` array tracks connections in `dsp64()`
- Perfect signal/float dual input on ALL inlets
- Solves the long-standing MSP multi-inlet problem completely

### Click-Free Parameter Transitions (CRITICAL BREAKTHROUGH)
Comprehensive solution for eliminating audio artifacts discovered in `cycle.fold~` development:
- Continuous algorithm paths eliminate switching artifacts
- Always-on processing prevents conditional switching that causes clicks
- Adaptive smoothing (heavy for float inputs, light for signal inputs)
- Mathematical continuity ensures smooth parameter transitions

### C++ Integration Patterns
Proven pattern for integrating complex C++ DSP algorithms:
- Extern "C" wrapper functions with opaque pointers
- Clean language boundary separation
- Easy maintenance and debugging
- Demonstrated in `tide~` and other advanced externals

### Professional Audio Processing Standards
Essential features for production-ready externals:
- Anti-aliasing protection for nonlinear processing
- DC offset removal and denormal protection
- Parameter smoothing for click-free operation
- Universal binary compilation for cross-platform compatibility

### Mathematical Algorithm Enhancement
Key principles for musical algorithm design:
- Linear relationships often outperform complex models
- Progressive parameter mapping provides intuitive control
- Mathematical precision with sample-rate independence
- Stability analysis and edge case handling

---

*Project history and key discoveries from Max SDK development sessions.*