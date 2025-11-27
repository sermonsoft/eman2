# EMAN2 API Reference

Quick reference for the most commonly used EMAN2 classes and functions.

## Table of Contents

1. [EMData Class](#emdata-class)
2. [Factory Classes](#factory-classes)
3. [Geometry Classes](#geometry-classes)
4. [Utility Functions](#utility-functions)
5. [Common Processors](#common-processors)
6. [Common Aligners](#common-aligners)
7. [File I/O](#file-io)

## EMData Class

The core image/volume class.

### Construction

```python
from EMAN2 import *

# Create empty image
img = EMData()

# Load from file
img = EMData("file.mrc")
img = EMData("stack.hdf", 5)  # Load image #5 from stack

# Create with specific size
img = EMData(256, 256, 1)  # 256x256 2D image
```

### File I/O

```python
# Read
img.read_image("input.mrc")
img.read_image("stack.hdf", 10)  # Read image #10
img.read_image("file.mrc", 0, True)  # Header only

# Write
img.write_image("output.mrc")
img.write_image("stack.hdf", -1)  # Append to stack
img.write_image("stack.hdf", 5)   # Overwrite image #5

# Get image count in file
n = EMUtil.get_image_count("stack.hdf")
```

### Basic Properties

```python
# Dimensions
nx = img.get_xsize()
ny = img.get_ysize()
nz = img.get_zsize()
ndim = img.get_ndim()  # 1, 2, or 3

# Data type
is_complex = img.is_complex()
is_real = img.is_real()

# Statistics (auto-calculated)
mean = img.get_attr("mean")
sigma = img.get_attr("sigma")
minimum = img.get_attr("minimum")
maximum = img.get_attr("maximum")
```

### Data Access

```python
# Get raw data pointer (use with caution!)
data = img.get_data()

# Get/set single pixel
value = img.get_value_at(x, y, z)
img.set_value_at(x, y, z, value)

# Get/set via indexing (2D only)
value = img[x, y]
img[x, y] = value

# Copy
img2 = img.copy()
img2 = img.copy_head()  # Copy header only, no data
```

### Metadata (Attributes)

```python
# Set attributes
img["apix_x"] = 1.2
img["apix_y"] = 1.2
img["apix_z"] = 1.2
img["ctf"] = ctf_object

# Get attributes
apix = img["apix_x"]
ctf = img["ctf"]

# Check existence
if img.has_attr("ctf"):
    ctf = img["ctf"]

# Get all attributes
attr_dict = img.get_attr_dict()
```

### Processing

```python
# Process in-place (modifies image)
img.process_inplace("normalize")
img.process_inplace("filter.lowpass", {"cutoff_freq": 0.3})

# Process (returns new image)
img2 = img.process("filter.lowpass", {"cutoff_freq": 0.3})

# Common operations
img.to_zero()           # Set all pixels to 0
img.to_one()            # Set all pixels to 1
img.mult(2.0)           # Multiply all pixels by 2
img.add(10.0)           # Add 10 to all pixels
img.update()            # Recalculate statistics
```

### Transforms

```python
# FFT
fft_img = img.do_fft()          # Forward FFT (returns new)
img.do_fft_inplace()            # Forward FFT (in-place)
real_img = fft_img.do_ift()     # Inverse FFT (returns new)
fft_img.do_ift_inplace()        # Inverse FFT (in-place)

# Rotation (3D)
rotated = img.process("xform", {"transform": transform_obj})

# Translation
img.translate(dx, dy, dz)

# Scaling
scaled = img.get_clip(Region(0, 0, 0, new_nx, new_ny, new_nz))
```

### Region Operations

```python
# Extract region
region = Region(x0, y0, z0, width, height, depth)
clip = img.get_clip(region)

# Insert region
img.insert_clip(clip, Vec3i(x0, y0, z0))

# Clip to size
clipped = img.get_clip(Region((nx-64)//2, (ny-64)//2, 0, 64, 64, 1))
```

### Alignment

```python
# Align to reference
aligned = img.align("rotate_translate", ref_img, "ccc", {})

# Get alignment transform
xform = aligned["xform.align2d"]

# Apply transform
transformed = img.process("xform", {"transform": xform})
```

## Factory Classes

### Processors

```python
# List all processors
procs = Processors.get_list()

# Get processor instance
proc = Processors.get("filter.lowpass")
proc = Processors.get("filter.lowpass", {"cutoff_freq": 0.3})

# Use processor
proc.process_inplace(img)
img2 = proc.process(img)

# Or use directly on EMData
img.process_inplace("filter.lowpass", {"cutoff_freq": 0.3})
```

### Aligners

```python
# List all aligners
aligners = Aligners.get_list()

# Get aligner
aligner = Aligners.get("rotate_translate")

# Align images
aligned = aligner.align(to_img, from_img, "ccc", {})
```

### Averagers

```python
# Create averager
avg = Averagers.get("mean")

# Add images
for i in range(n):
    img = EMData("stack.hdf", i)
    avg.add_image(img)

# Get result
result = avg.finish()
```

### Cmps (Comparators)

```python
# Create comparator
cmp = Cmps.get("ccc")

# Compare images
similarity = cmp.cmp(img1, img2)
```

### Projectors

```python
# Create projector
proj = Projectors.get("standard")

# Project 3D volume to 2D
projection = proj.project3d(volume)
```

### Reconstructors

```python
# Create reconstructor
recon = Reconstructors.get("fourier")

# Setup
recon.setup()

# Insert slices
for i in range(n):
    slice_img = EMData("slices.hdf", i)
    euler = slice_img["xform.projection"]
    recon.insert_slice(slice_img, euler)

# Get 3D reconstruction
volume = recon.finish()
```

## Geometry Classes

### Vec3f (3D Float Vector)

```python
v = Vec3f(1.0, 2.0, 3.0)
x = v[0]  # or v.x
y = v[1]  # or v.y
z = v[2]  # or v.z

# Operations
v2 = v1 + v2
v2 = v1 - v2
v2 = v1 * 2.0
length = v.length()
normalized = v.normalize()
```

### Transform

```python
t = Transform()

# Set rotation (Euler angles)
t.set_rotation({"type": "eman", "az": 45, "alt": 30, "phi": 0})

# Set translation
t.set_trans(10, 20, 0)

# Set scale
t.set_scale(1.5)

# Get components
rot_dict = t.get_rotation("eman")
trans = t.get_trans()

# Apply to image
transformed = img.process("xform", {"transform": t})
```

### Region

```python
# 2D region
r = Region(x, y, width, height)

# 3D region
r = Region(x, y, z, width, height, depth)

# Properties
origin = r.get_origin()  # Vec3f
size = r.get_size()      # FloatSize
```

## Utility Functions

### EMUtil

```python
# Image info
n_images = EMUtil.get_image_count("stack.hdf")
img_type = EMUtil.get_image_type("file.mrc")

# Image statistics
stats = EMUtil.get_image_stats("file.mrc", 0)

# All image info
info = EMUtil.get_all_attributes("file.mrc", 0)
```

### TestUtil

```python
# Create test images
img = TestUtil.make_sphere(64, 64, 64, 20)  # Sphere
img = TestUtil.make_gaussian(64, 64, 64, 10)  # Gaussian
```

## Common Processors

### Filters

```python
# Low-pass filter
img.process_inplace("filter.lowpass.gauss", {"cutoff_freq": 0.3})
img.process_inplace("filter.lowpass.butterworth", {"cutoff_freq": 0.3, "order": 4})

# High-pass filter
img.process_inplace("filter.highpass.gauss", {"cutoff_freq": 0.05})

# Band-pass filter
img.process_inplace("filter.bandpass", {"low_cutoff": 0.05, "high_cutoff": 0.3})
```

### Normalization

```python
# Normalize to mean=0, std=1
img.process_inplace("normalize")

# Normalize to specific mean/std
img.process_inplace("normalize", {"mean": 0, "std": 1})

# Edge mean normalization
img.process_inplace("normalize.edgemean")
```

### Masking

```python
# Soft circular mask
img.process_inplace("mask.soft", {"outer_radius": 64, "width": 5})

# Sharp circular mask
img.process_inplace("mask.sharp", {"outer_radius": 64})

# Gaussian mask
img.process_inplace("mask.gaussian", {"outer_radius": 64})
```

### Math Operations

```python
# Absolute value
img.process_inplace("math.absvalue")

# Square
img.process_inplace("math.squared")

# Square root
img.process_inplace("math.sqrt")

# Log
img.process_inplace("math.log")
```

## Common Aligners

```python
# 2D rotation + translation
aligned = img.align("rotate_translate", ref)

# 2D rotation + translation + flip
aligned = img.align("rotate_translate_flip", ref)

# Refinement
aligned = img.align("refine", ref, "ccc", {"xform.align2d": initial_xform})
```

## File I/O

### Supported Formats

- **MRC/MRCS**: `.mrc`, `.mrcs`, `.st`, `.ali`
- **HDF5**: `.hdf`, `.h5`, `.hdf5`
- **TIFF**: `.tif`, `.tiff`
- **SPIDER**: `.spi`
- **IMAGIC**: `.hed`, `.img`
- **DM3/DM4**: `.dm3`, `.dm4`
- **PNG/JPEG**: `.png`, `.jpg`

### Reading

```python
# Single image
img = EMData("file.mrc")

# From stack
img = EMData("stack.hdf", 5)

# Header only
img = EMData()
img.read_image("file.mrc", 0, True)

# Region
region = Region(0, 0, 0, 128, 128, 1)
img = EMData("file.mrc", 0, False, region)
```

### Writing

```python
# Single image
img.write_image("output.mrc")

# To stack (append)
img.write_image("stack.hdf", -1)

# To stack (specific index)
img.write_image("stack.hdf", 5)

# Specify format
img.write_image("output.img", 0, EMUtil.IMAGE_IMAGIC)
```

## CTF (Contrast Transfer Function)

### EMAN2Ctf

```python
from EMAN2 import *

# Create CTF
ctf = EMAN2Ctf()

# Set parameters
ctf.defocus = 2.0        # Defocus in microns
ctf.bfactor = 100.0      # B-factor
ctf.amplitude = 0.1      # Amplitude contrast (0-1)
ctf.voltage = 300        # kV
ctf.cs = 2.7             # Spherical aberration (mm)
ctf.apix = 1.2           # Angstroms per pixel

# Compute CTF
ctf.compute_2d_complex(img, Ctf.CtfType.CTF_AMP)

# Apply CTF correction
img.process_inplace("math.simulatectf", {"ctf": ctf})
```

### CTF from Image

```python
# Get CTF from image metadata
if img.has_attr("ctf"):
    ctf = img["ctf"]
    defocus = ctf.defocus
```

## Database Operations

### EMAN2DB

```python
from EMAN2db import *

# Open database
db = EMAN2DB.open_db("project.eman2db")

# Store data
db.set_data("key", value)

# Retrieve data
value = db.get_data("key")

# List keys
keys = db.get_all_keys()
```

## Advanced Topics

### Symmetry

```python
# Apply symmetry
sym = Symmetry("c5")  # C5 symmetry
img.process_inplace("xform.applysym", {"sym": "c5"})

# Get symmetry transforms
xforms = sym.get_syms()
```

### Fourier Operations

```python
# Phase flip
img.process_inplace("math.phaseflip")

# Amplitude to 1
img.process_inplace("math.amplitude.to.one")

# Real to complex
img.ri2inten()  # Real/Imaginary to Intensity

# Apply function in Fourier space
img.process_inplace("math.fft.resample", {"n": 2})
```

### Parallel Processing

```python
from EMAN2PAR import *

# Initialize MPI
EMData.set_mpi_mode(True)

# Parallel image processing
# (requires MPI-enabled build)
```

## Python-Specific Features

### NumPy Integration

```python
import numpy as np
from EMAN2 import *

# EMData to NumPy (shares memory!)
img = EMData("file.mrc")
arr = EMNumPy.em2numpy(img)

# Modify NumPy array (affects EMData)
arr *= 2.0

# NumPy to EMData (copies data)
arr = np.random.rand(128, 128).astype(np.float32)
img = EMNumPy.numpy2em(arr)
```

### List Operations

```python
# Process list of images
images = [EMData("stack.hdf", i) for i in range(10)]

# Apply processor to list
proc = Processors.get("normalize")
proc.process_list_inplace(images)
```

### Context Managers

```python
# Safe file handling
with EMData("input.mrc") as img:
    img.process_inplace("normalize")
    img.write_image("output.mrc")
```

## Common Workflows

### Basic Image Processing Pipeline

```python
from EMAN2 import *

# Load image
img = EMData("micrograph.mrc")

# Normalize
img.process_inplace("normalize.edgemean")

# Low-pass filter
img.process_inplace("filter.lowpass.gauss", {"cutoff_freq": 0.3})

# Apply soft mask
img.process_inplace("mask.soft", {"outer_radius": 100, "width": 10})

# Save result
img.write_image("processed.mrc")
```

### Particle Averaging

```python
from EMAN2 import *

# Create averager
avg = Averagers.get("mean")

# Load and add particles
n = EMUtil.get_image_count("particles.hdf")
for i in range(n):
    ptcl = EMData("particles.hdf", i)
    ptcl.process_inplace("normalize")
    avg.add_image(ptcl)

# Get average
average = avg.finish()
average.write_image("class_average.hdf")
```

### 3D Reconstruction

```python
from EMAN2 import *

# Create reconstructor
recon = Reconstructors.get("fourier")
recon.setup()

# Add projections
n = EMUtil.get_image_count("projections.hdf")
for i in range(n):
    proj = EMData("projections.hdf", i)

    # Get orientation
    xform = proj["xform.projection"]

    # Insert into reconstruction
    recon.insert_slice(proj, xform)

# Finish reconstruction
volume = recon.finish()
volume.write_image("reconstruction.hdf")
```

### CTF Correction

```python
from EMAN2 import *

# Load particle
ptcl = EMData("particle.mrc")

# Get CTF parameters
ctf = ptcl["ctf"]

# Phase flip
ptcl.process_inplace("math.simulatectf", {
    "ctf": ctf,
    "return": "phaseflipped"
})

# Or Wiener filter
ptcl.process_inplace("math.simulatectf", {
    "ctf": ctf,
    "return": "wiener",
    "snr": 0.1
})
```

## Performance Tips

### Memory Management

```python
# Explicitly delete large images
del img
import gc
gc.collect()

# Use in-place operations when possible
img.process_inplace("normalize")  # Better than img = img.process("normalize")

# Read header only when you don't need data
img = EMData()
img.read_image("file.mrc", 0, True)  # Header only
```

### Batch Processing

```python
# Process multiple images efficiently
for i in range(n):
    img = EMData("stack.hdf", i)
    img.process_inplace("normalize")
    img.process_inplace("filter.lowpass", {"cutoff_freq": 0.3})
    img.write_image("processed.hdf", -1)  # Append
```

### Region Processing

```python
# Process only a region (saves memory)
region = Region(x, y, z, width, height, depth)
img = EMData("large_file.mrc", 0, False, region)
img.process_inplace("normalize")
```

## Error Handling

```python
from EMAN2 import *

try:
    img = EMData("file.mrc")
except RuntimeError as e:
    print(f"Error loading image: {e}")

try:
    img.process_inplace("nonexistent_processor")
except Exception as e:
    print(f"Processing error: {e}")

# Check if file exists
import os
if os.path.exists("file.mrc"):
    img = EMData("file.mrc")
```

## Logging

```python
from EMAN2 import *

# Set log level
Log.logger().set_level(Log.DEBUG_LOG)

# Log messages
Log.logger().debug("Debug message")
Log.logger().info("Info message")
Log.logger().warning("Warning message")
Log.logger().error("Error message")
```

---

**See Also**:
- [LEARNING_GUIDE.md](LEARNING_GUIDE.md) - Start here
- [ARCHITECTURE.md](ARCHITECTURE.md) - System architecture
- [CORE_CONCEPTS.md](CORE_CONCEPTS.md) - Key concepts
- [DEVELOPER_GUIDE.md](DEVELOPER_GUIDE.md) - Building and extending
- [Official EMAN2 Wiki](http://blake.bcm.edu/emanwiki)

**Commit Message:**
docs: create comprehensive API reference for EMAN2

