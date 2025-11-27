# Fourier Transforms in EMAN2

A comprehensive guide to understanding and using Fourier transforms in EMAN2 for cryo-EM image processing.

## Table of Contents

1. [Introduction to Fourier Transforms](#introduction-to-fourier-transforms)
2. [Why Fourier Transforms in Cryo-EM?](#why-fourier-transforms-in-cryo-em)
3. [Fourier Transforms in EMAN2](#fourier-transforms-in-eman2)
4. [Practical Examples](#practical-examples)
5. [Common Operations](#common-operations)
6. [Advanced Topics](#advanced-topics)

## Introduction to Fourier Transforms

### What is a Fourier Transform?

A **Fourier Transform** converts an image from **real space** (spatial domain) to **Fourier space** (frequency domain).

**Key Concepts**:
- **Real Space**: Image as pixels with intensity values (what you see)
- **Fourier Space**: Image as frequencies and phases (how it's composed)
- **Frequency**: How rapidly intensity changes across the image
- **Phase**: The position/alignment of those changes

### Mathematical Foundation

For a 2D image f(x,y), the Fourier Transform F(u,v) is:

```
F(u,v) = ∫∫ f(x,y) * e^(-2πi(ux + vy)) dx dy
```

**In simple terms**:
- Decomposes image into sum of sine and cosine waves
- Each wave has a specific frequency and direction
- Low frequencies = large-scale features (overall shape)
- High frequencies = fine details (edges, noise)

### Real vs Complex Images

**Real Space Image**:
- Each pixel: single float value (intensity)
- Size: nx × ny pixels
- Memory: nx × ny × 4 bytes

**Fourier Space Image** (Complex):
- Each pixel: two float values (real + imaginary) or (amplitude + phase)
- Size: (nx/2+1) × ny for 2D (Hermitian symmetry)
- Memory: (nx/2+1) × ny × 8 bytes

## Why Fourier Transforms in Cryo-EM?

### 1. CTF Correction

The **Contrast Transfer Function (CTF)** affects images in Fourier space:

```
Image_observed(f) = Image_true(f) × CTF(f)
```

**Why Fourier space?**
- CTF is multiplicative in Fourier space (simple!)
- CTF is convolutional in real space (complex!)
- Easy to correct: divide by CTF in Fourier space

### 2. Filtering

**Low-pass filter** (remove high frequencies = noise):
```python
# In Fourier space: multiply by filter function
# In real space: would require complex convolution
```

**High-pass filter** (remove low frequencies = background):
```python
# Easy in Fourier space, hard in real space
```

### 3. 3D Reconstruction

**Central Slice Theorem**:
- A 2D projection of a 3D object
- Corresponds to a central slice through the 3D Fourier transform
- Makes reconstruction possible!

### 4. Alignment and Correlation

**Cross-correlation** for alignment:
- Multiplication in Fourier space
- Convolution in real space
- Much faster in Fourier space!

### 5. Resolution Assessment

**Fourier Ring Correlation (FRC)**:
- Measures resolution in Fourier space
- Shows which frequencies are reliable
- Standard metric in cryo-EM

## Fourier Transforms in EMAN2

### EMData FFT Operations

EMAN2 uses **FFTW3** (Fastest Fourier Transform in the West) library.

#### Forward FFT (Real → Fourier)

```python
from EMAN2 import *

# Load real-space image
img = EMData("particle.mrc")
print(img.is_real())      # True
print(img.is_complex())   # False

# Forward FFT (creates new image)
fft_img = img.do_fft()
print(fft_img.is_real())      # False
print(fft_img.is_complex())   # True

# Forward FFT (in-place, modifies original)
img.do_fft_inplace()
print(img.is_complex())   # True
```

#### Inverse FFT (Fourier → Real)

```python
# Inverse FFT (creates new image)
real_img = fft_img.do_ift()
print(real_img.is_real())  # True

# Inverse FFT (in-place)
fft_img.do_ift_inplace()
print(fft_img.is_real())   # True
```

### Memory Layout

**Real Space** (nx × ny):
```
[pixel(0,0), pixel(1,0), ..., pixel(nx-1,0),
 pixel(0,1), pixel(1,1), ..., pixel(nx-1,1),
 ...
 pixel(0,ny-1), ..., pixel(nx-1,ny-1)]
```

**Fourier Space** ((nx/2+1) × ny):
```
[complex(0,0), complex(1,0), ..., complex(nx/2,0),
 complex(0,1), complex(1,1), ..., complex(nx/2,1),
 ...
 complex(0,ny-1), ..., complex(nx/2,ny-1)]
```

Each complex value = 2 floats (real, imaginary)

### Fourier Space Coordinates

**Center vs Corner**:
- **Corner origin**: (0,0) is DC (zero frequency)
- **Center origin**: (nx/2, ny/2) is DC
- EMAN2 uses corner origin by default

```python
# Convert between origins
fft_img.process_inplace("xform.fourierorigin.tocenter")
fft_img.process_inplace("xform.fourierorigin.tocorner")
```

### Amplitude and Phase

```python
# Get amplitude (magnitude)
amplitude = fft_img.do_fft_amp()

# Get phase
phase = fft_img.do_fft_phase()

# Convert real/imaginary to amplitude/phase
fft_img.ri2ap()  # Real/Imag → Amplitude/Phase

# Convert amplitude/phase to real/imaginary
fft_img.ap2ri()  # Amplitude/Phase → Real/Imag

# Get intensity (amplitude squared)
intensity = fft_img.do_fft_intensity()
```

## Practical Examples

### Example 1: Basic FFT Round-Trip

```python
from EMAN2 import *

# Load image
img = EMData("particle.mrc")
original_mean = img.get_attr("mean")

# Forward FFT
fft = img.do_fft()
print(f"FFT size: {fft.get_xsize()} x {fft.get_ysize()}")
print(f"Is complex: {fft.is_complex()}")

# Inverse FFT
reconstructed = fft.do_ift()
reconstructed_mean = reconstructed.get_attr("mean")

# Should be nearly identical
print(f"Original mean: {original_mean}")
print(f"Reconstructed mean: {reconstructed_mean}")
print(f"Difference: {abs(original_mean - reconstructed_mean)}")
```

### Example 2: Low-Pass Filtering

```python
from EMAN2 import *

# Load image
img = EMData("noisy_particle.mrc")

# Method 1: Direct processor (handles FFT internally)
filtered = img.process("filter.lowpass.gauss", {
    "cutoff_freq": 0.3,  # Cutoff at 0.3 (Nyquist = 0.5)
    "apix": 1.2          # Angstroms per pixel
})

# Method 2: Manual FFT approach
fft = img.do_fft()

# Apply filter in Fourier space
fft.process_inplace("filter.lowpass.gauss", {"cutoff_freq": 0.3})

# Transform back
filtered = fft.do_ift()

filtered.write_image("filtered_particle.mrc")
```

### Example 3: CTF Correction (Phase Flip)

```python
from EMAN2 import *

# Load particle with CTF info
ptcl = EMData("particle.mrc")
ctf = ptcl["ctf"]  # EMAN2Ctf object

# Phase flip correction
# Multiplies by sign of CTF in Fourier space
ptcl.process_inplace("math.simulatectf", {
    "ctf": ctf,
    "return": "phaseflipped"
})

ptcl.write_image("phaseflipped.mrc")
```

### Example 4: Fourier Ring Correlation (FRC)

```python
from EMAN2 import *

# Load two half-maps
half1 = EMData("half_map_1.mrc")
half2 = EMData("half_map_2.mrc")

# Compute FRC
frc_curve = half1.calc_fourier_shell_correlation(half2)

# frc_curve is XYData object
# x = spatial frequency
# y = correlation coefficient

# Find resolution at FSC=0.143
resolution = None
for i in range(frc_curve.get_size()):
    freq, fsc = frc_curve.get_xypoint(i)
    if fsc < 0.143:
        resolution = 1.0 / freq  # Convert frequency to Angstroms
        break

print(f"Resolution at FSC=0.143: {resolution:.2f} Å")
```

### Example 5: Power Spectrum

```python
from EMAN2 import *

# Load micrograph
mic = EMData("micrograph.mrc")

# Compute power spectrum (amplitude squared)
fft = mic.do_fft()
power_spectrum = fft.do_fft_intensity()

# Often displayed as log scale
power_spectrum.process_inplace("math.log")

# Rotationally average for 1D plot
radial_profile = power_spectrum.calc_radial_dist(
    power_spectrum.get_xsize() // 2,  # Number of bins
    0, 0.5,  # From DC to Nyquist
    True     # Return as XYData
)

power_spectrum.write_image("power_spectrum.mrc")
```

## Common Operations

### Filtering Operations

```python
from EMAN2 import *

img = EMData("particle.mrc")

# Low-pass filters
img.process_inplace("filter.lowpass.gauss", {"cutoff_freq": 0.3})
img.process_inplace("filter.lowpass.butterworth", {
    "cutoff_freq": 0.3,
    "order": 4
})
img.process_inplace("filter.lowpass.tanh", {
    "cutoff_freq": 0.3,
    "fall_off": 0.1
})

# High-pass filters
img.process_inplace("filter.highpass.gauss", {"cutoff_freq": 0.05})

# Band-pass filter
img.process_inplace("filter.bandpass", {
    "low_cutoff": 0.05,
    "high_cutoff": 0.3
})

# Matched filter (template matching)
img.process_inplace("filter.matchto", {"to": template_img})
```

### CTF Operations

```python
from EMAN2 import *

# Create CTF
ctf = EMAN2Ctf()
ctf.defocus = 2.0      # microns
ctf.bfactor = 100.0    # B-factor
ctf.amplitude = 0.1    # Amplitude contrast
ctf.voltage = 300      # kV
ctf.cs = 2.7           # mm
ctf.apix = 1.2         # Angstroms/pixel

# Apply CTF to image
img = EMData("particle.mrc")

# Simulate CTF (multiply in Fourier space)
img.process_inplace("math.simulatectf", {"ctf": ctf})

# Phase flip
img.process_inplace("math.simulatectf", {
    "ctf": ctf,
    "return": "phaseflipped"
})

# Wiener filter
img.process_inplace("math.simulatectf", {
    "ctf": ctf,
    "return": "wiener",
    "snr": 0.1
})
```

### Masking in Fourier Space

```python
from EMAN2 import *

img = EMData("particle.mrc")
fft = img.do_fft()

# Wedge mask (for tomography)
fft.process_inplace("mask.wedgefill", {
    "thresh_sigma": 0.5
})

# Cone mask
fft.process_inplace("mask.cone", {
    "angle": 45,  # degrees
    "axis": "z"
})

# Back to real space
masked = fft.do_ift()
```

## Advanced Topics

### 1. Hermitian Symmetry

Real images have **Hermitian symmetric** Fourier transforms:
```
F(-u, -v) = F*(u, v)  (* = complex conjugate)
```

**Implications**:
- Only need to store half of Fourier space
- EMAN2 stores: (nx/2+1) × ny complex values
- Other half can be computed from symmetry
- Saves memory and computation time

### 2. Nyquist Frequency

**Nyquist frequency** = 0.5 cycles/pixel (maximum representable frequency)

```python
# Frequency in Angstroms
apix = 1.2  # Angstroms per pixel
nyquist_angstroms = 2 * apix  # 2.4 Å

# Cutoff as fraction of Nyquist
cutoff_angstroms = 10.0  # Want 10 Å resolution
cutoff_freq = (2 * apix) / cutoff_angstroms  # 0.24
```

### 3. Fourier Cropping/Padding

**Fourier cropping** = downsampling without aliasing:

```python
# Downsample by factor of 2
img.process_inplace("math.fft.resample", {"n": 2})

# Upsample by factor of 2  
img.process_inplace("math.fft.resample", {"n": 0.5})
```

**How it works**:
1. FFT to Fourier space
2. Crop/pad Fourier coefficients
3. Inverse FFT
4. Result: perfect band-limited resampling

### 4. Fourier Shell Correlation (FSC)

**3D version of FRC** for resolution assessment:

```python
from EMAN2 import *

# Two independent 3D reconstructions
vol1 = EMData("volume_half1.mrc")
vol2 = EMData("volume_half2.mrc")

# Compute FSC
fsc = vol1.calc_fourier_shell_correlation(vol2)

# Plot FSC curve
import matplotlib.pyplot as plt
freqs = [fsc.get_xatpoint(i) for i in range(fsc.get_size())]
corrs = [fsc.get_yatpoint(i) for i in range(fsc.get_size())]

plt.plot(freqs, corrs)
plt.axhline(y=0.143, color='r', linestyle='--', label='FSC=0.143')
plt.xlabel('Spatial Frequency (1/Å)')
plt.ylabel('FSC')
plt.legend()
plt.savefig('fsc_curve.png')
```

### 5. Spectral Signal-to-Noise Ratio (SSNR)

```python
# Estimate SSNR from two half-maps
ssnr = vol1.calc_ssnr(vol2)

# Use SSNR for Wiener filtering
img.process_inplace("filter.wiener", {"ssnr": ssnr})
```

### 6. Gridding and Interpolation

**Fourier gridding** for 3D reconstruction:

```python
from EMAN2 import *

# Create Fourier gridding reconstructor
recon = Reconstructors.get("fourier_gridding", {
    "size": 256,
    "npad": 2,      # Padding factor
    "kb_K": 6,      # Kaiser-Bessel kernel size
    "kb_alpha": 1.25  # Kaiser-Bessel alpha
})

# Insert 2D slices
for i in range(n_projections):
    proj = EMData(f"projection_{i}.mrc")
    euler = proj["xform.projection"]
    recon.insert_slice(proj, euler)

# Get 3D volume
volume = recon.finish()
```

**How it works**:
1. Each 2D projection → 2D FFT
2. Insert into 3D Fourier grid (with interpolation)
3. 3D inverse FFT → 3D volume

## Practical Tips and Best Practices

### 1. When to Use FFT

**Use FFT when**:
✅ Applying filters (low-pass, high-pass, band-pass)
✅ CTF correction
✅ Cross-correlation for alignment
✅ Resolution assessment (FRC/FSC)
✅ 3D reconstruction
✅ Spectral analysis

**Don't use FFT when**:
❌ Simple pixel operations (add, multiply, threshold)
❌ Geometric transforms (rotation, translation in real space)
❌ Masking in real space
❌ Normalization

### 2. Performance Considerations

```python
from EMAN2 import *

# GOOD: In-place operations (faster, less memory)
img.do_fft_inplace()
img.process_inplace("filter.lowpass.gauss", {"cutoff_freq": 0.3})
img.do_ift_inplace()

# LESS GOOD: Creates new images (slower, more memory)
fft = img.do_fft()
filtered = fft.process("filter.lowpass.gauss", {"cutoff_freq": 0.3})
result = filtered.do_ift()
```

**FFT Performance**:
- FFT is O(N log N) - very fast!
- FFTW3 is highly optimized
- Power-of-2 sizes are fastest (64, 128, 256, 512, etc.)
- In-place operations save memory

### 3. Avoiding Common Pitfalls

**Pitfall 1: Forgetting to transform back**
```python
# WRONG: Saving Fourier space image as real
img.do_fft_inplace()
img.write_image("output.mrc")  # Will look like noise!

# CORRECT: Transform back to real space
img.do_fft_inplace()
# ... do operations in Fourier space ...
img.do_ift_inplace()
img.write_image("output.mrc")
```

**Pitfall 2: Mixing real and complex operations**
```python
# WRONG: Can't add real and complex images
real_img = EMData("real.mrc")
fft_img = real_img.do_fft()
result = real_img + fft_img  # ERROR!

# CORRECT: Keep track of image type
if img.is_complex():
    img.do_ift_inplace()  # Convert to real first
```

**Pitfall 3: Incorrect frequency units**
```python
# Frequency can be specified in different ways:

# 1. Fraction of Nyquist (0 to 0.5)
cutoff_freq = 0.3  # 0.3 * Nyquist

# 2. Absolute frequency (1/Angstroms)
apix = 1.2
resolution_angstroms = 10.0
cutoff_abs = 1.0 / resolution_angstroms  # 0.1 Å^-1

# 3. Resolution (Angstroms)
# Some processors take resolution directly
img.process_inplace("filter.lowpass.gauss", {
    "cutoff_abs": cutoff_abs,
    "apix": apix
})
```

**Pitfall 4: Edge effects**
```python
# FFT assumes periodic boundaries
# Sharp edges cause ringing artifacts

# SOLUTION: Apply soft mask before FFT
img.process_inplace("mask.soft", {
    "outer_radius": 100,
    "width": 10  # Soft edge
})
img.do_fft_inplace()
```

### 4. Debugging Fourier Space Issues

```python
from EMAN2 import *

# Check if image is complex
print(f"Is complex: {img.is_complex()}")
print(f"Is real: {img.is_real()}")

# Check size
print(f"Real size: {img.get_xsize()} x {img.get_ysize()}")
fft = img.do_fft()
print(f"FFT size: {fft.get_xsize()} x {fft.get_ysize()}")
# FFT x-size should be (real_x/2 + 1)

# Visualize power spectrum
power = fft.do_fft_intensity()
power.process_inplace("math.log")  # Log scale for viewing
power.write_image("power_spectrum.mrc")

# Check for NaN or Inf
stats = img.get_attr_dict()
print(f"Min: {stats['minimum']}, Max: {stats['maximum']}")
print(f"Mean: {stats['mean']}, Sigma: {stats['sigma']}")
```

### 5. Optimizing Filter Parameters

```python
from EMAN2 import *

# Test different cutoff frequencies
img = EMData("particle.mrc")

cutoffs = [0.1, 0.2, 0.3, 0.4, 0.5]
for i, cutoff in enumerate(cutoffs):
    filtered = img.process("filter.lowpass.gauss", {
        "cutoff_freq": cutoff
    })
    filtered.write_image(f"filtered_{cutoff:.1f}.mrc")

# Visually inspect to find optimal cutoff
```

## Fourier Transform Cheat Sheet

### Quick Reference

| Operation | Code | Use Case |
|-----------|------|----------|
| **Forward FFT** | `fft = img.do_fft()` | Real → Fourier |
| **Inverse FFT** | `real = fft.do_ift()` | Fourier → Real |
| **In-place FFT** | `img.do_fft_inplace()` | Save memory |
| **In-place IFT** | `img.do_ift_inplace()` | Save memory |
| **Amplitude** | `amp = fft.do_fft_amp()` | Get magnitude |
| **Phase** | `phase = fft.do_fft_phase()` | Get phase |
| **Power** | `power = fft.do_fft_intensity()` | Amplitude² |
| **Low-pass** | `img.process_inplace("filter.lowpass.gauss", {"cutoff_freq": 0.3})` | Remove noise |
| **High-pass** | `img.process_inplace("filter.highpass.gauss", {"cutoff_freq": 0.05})` | Remove background |
| **CTF correct** | `img.process_inplace("math.simulatectf", {"ctf": ctf, "return": "phaseflipped"})` | Phase flip |
| **FRC** | `frc = img1.calc_fourier_shell_correlation(img2)` | Resolution |

### Frequency Conversions

```python
# Given
apix = 1.2              # Angstroms per pixel
resolution_A = 10.0     # Desired resolution in Angstroms
nx = 256                # Image size

# Conversions
nyquist_freq = 0.5                          # Cycles per pixel
nyquist_A = 2 * apix                        # 2.4 Å (Nyquist in Angstroms)

cutoff_freq = nyquist_freq * (nyquist_A / resolution_A)  # Fraction of Nyquist
# For 10 Å: cutoff_freq = 0.5 * (2.4 / 10.0) = 0.12

cutoff_abs = 1.0 / resolution_A             # 0.1 Å^-1 (absolute frequency)

# Pixel in Fourier space to frequency
pixel_freq = pixel_index / nx               # Cycles per pixel
pixel_A = 1.0 / (pixel_freq / apix)        # Angstroms
```

### Common Filter Cutoffs

| Resolution Goal | Cutoff (fraction of Nyquist) | Notes |
|----------------|------------------------------|-------|
| **Remove noise** | 0.3 - 0.4 | Typical for low-pass |
| **High resolution** | 0.4 - 0.5 | Preserve fine details |
| **Medium resolution** | 0.2 - 0.3 | Balance detail/noise |
| **Low resolution** | 0.1 - 0.2 | Smooth, remove details |
| **Remove background** | 0.02 - 0.05 | High-pass filter |

## Real-World Workflows

### Workflow 1: Particle Processing Pipeline

```python
from EMAN2 import *

def process_particle(input_file, output_file, ctf_params):
    """Complete particle processing with FFT operations"""

    # 1. Load particle
    ptcl = EMData(input_file)

    # 2. Normalize (real space)
    ptcl.process_inplace("normalize.edgemean")

    # 3. CTF correction (Fourier space)
    ctf = EMAN2Ctf()
    ctf.defocus = ctf_params['defocus']
    ctf.voltage = ctf_params['voltage']
    ctf.cs = ctf_params['cs']
    ctf.apix = ctf_params['apix']

    ptcl.process_inplace("math.simulatectf", {
        "ctf": ctf,
        "return": "phaseflipped"
    })

    # 4. Low-pass filter (Fourier space)
    ptcl.process_inplace("filter.lowpass.gauss", {
        "cutoff_freq": 0.3
    })

    # 5. Soft mask (real space)
    ptcl.process_inplace("mask.soft", {
        "outer_radius": 64,
        "width": 5
    })

    # 6. Save
    ptcl.write_image(output_file)

    return ptcl

# Use it
ctf_params = {
    'defocus': 2.0,
    'voltage': 300,
    'cs': 2.7,
    'apix': 1.2
}

processed = process_particle("raw_particle.mrc", "processed.mrc", ctf_params)
```

### Workflow 2: Resolution Assessment

```python
from EMAN2 import *

def assess_resolution(half1_file, half2_file, apix, fsc_threshold=0.143):
    """Compute resolution from two half-maps"""

    # Load half-maps
    half1 = EMData(half1_file)
    half2 = EMData(half2_file)

    # Compute FSC
    fsc = half1.calc_fourier_shell_correlation(half2)

    # Find resolution at threshold
    resolution = None
    for i in range(fsc.get_size()):
        freq = fsc.get_xatpoint(i)
        corr = fsc.get_yatpoint(i)

        if corr < fsc_threshold:
            # Convert frequency to Angstroms
            resolution = apix / freq if freq > 0 else float('inf')
            break

    # Save FSC curve
    fsc.write_file("fsc_curve.txt")

    print(f"Resolution at FSC={fsc_threshold}: {resolution:.2f} Å")
    return resolution, fsc

# Use it
resolution, fsc_curve = assess_resolution(
    "half_map_1.mrc",
    "half_map_2.mrc",
    apix=1.2,
    fsc_threshold=0.143
)
```

### Workflow 3: Power Spectrum Analysis

```python
from EMAN2 import *

def analyze_power_spectrum(micrograph_file, output_file):
    """Analyze micrograph power spectrum for CTF fitting"""

    # Load micrograph
    mic = EMData(micrograph_file)

    # Compute power spectrum
    fft = mic.do_fft()
    power = fft.do_fft_intensity()

    # Log scale for visualization
    power.process_inplace("math.log")

    # Rotational average
    radial = power.calc_radial_dist(
        power.get_xsize() // 2,
        0, 0.5,
        True
    )

    # Save
    power.write_image(output_file)
    radial.write_file(output_file.replace('.mrc', '_radial.txt'))

    return power, radial

# Use it
power, radial = analyze_power_spectrum(
    "micrograph.mrc",
    "power_spectrum.mrc"
)
```

## Summary

### Key Takeaways

1. **Fourier transforms** convert between real and frequency domains
2. **Many operations are easier** in Fourier space (filtering, CTF correction)
3. **EMAN2 uses FFTW3** for fast, efficient FFT computation
4. **Hermitian symmetry** saves memory for real images
5. **Always transform back** to real space for visualization/saving
6. **Use in-place operations** for better performance
7. **Understand frequency units** (fraction of Nyquist vs absolute)

### Further Reading

- **EMAN2 Wiki**: http://blake.bcm.edu/emanwiki
- **FFTW Documentation**: http://www.fftw.org/
- **Fourier Analysis**: "The Fourier Transform and Its Applications" by Bracewell
- **Cryo-EM**: "Electron Crystallography of Biological Macromolecules" by Glaeser et al.

### Related Tutorial Documents

- [CORE_CONCEPTS.md](CORE_CONCEPTS.md) - EMData class and transforms
- [API_REFERENCE.md](API_REFERENCE.md) - FFT method reference
- [DEVELOPER_GUIDE.md](DEVELOPER_GUIDE.md) - Adding custom filters

---

**Commit Message:**
docs: add comprehensive Fourier transform guide for EMAN2

Complete tutorial covering Fourier transforms in cryo-EM context:
- Mathematical foundations and intuition
- Why FFT is essential for cryo-EM
- EMAN2 FFT operations and API
- Practical examples (filtering, CTF, FRC)
- Advanced topics (gridding, SSNR, FSC)
- Best practices and common pitfalls
- Real-world workflows
- Quick reference cheat sheet


