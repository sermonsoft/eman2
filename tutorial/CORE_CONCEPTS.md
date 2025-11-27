# EMAN2 Core Concepts

This document explains the fundamental concepts, data structures, and design patterns you need to understand to work with EMAN2 effectively.

## Table of Contents

1. [EMData - The Core Image Class](#emdata---the-core-image-class)
2. [The Factory Pattern](#the-factory-pattern)
3. [Modular Classes](#modular-classes)
4. [Image I/O Architecture](#image-io-architecture)
5. [Transforms and Geometry](#transforms-and-geometry)
6. [Python-C++ Integration](#python-c-integration)

## EMData - The Core Image Class

`EMData` is the fundamental data structure in EMAN2. Everything revolves around it.

### What is EMData?

An `EMData` object represents:
- A 1D, 2D, or 3D image/volume
- Can be in **real space** or **Fourier space** (complex)
- Stores pixel data as `float*` array
- Contains metadata as a `Dict` (dictionary)

### Memory Layout

```
Data ordering: x increases fastest, then y, then z
For a 3D image (nx, ny, nz):
  index = x + y*nx + z*nx*ny
```

### Key Operations

```cpp
// C++ Example
EMData* img = new EMData();
img->read_image("input.mrc");           // Load from file
img->set_size(256, 256, 1);             // Set dimensions
float* data = img->get_data();          // Access raw data
img->process_inplace("filter.lowpass"); // Apply filter
img->do_fft_inplace();                  // Transform to Fourier
img->write_image("output.hdf");         // Save to file
```

```python
# Python Example
from EMAN2 import *

img = EMData("input.mrc")               # Load from file
img.process_inplace("filter.lowpass", {"cutoff_freq": 0.3})
img.do_fft_inplace()                    # Transform to Fourier
img.write_image("output.hdf")           # Save to file

# Access metadata
img["apix_x"] = 1.2                     # Set pixel size
ctf = img["ctf"]                        # Get CTF parameters
```

### EMData Structure

The class is split across multiple header files for organization:

```cpp
class EMData {
    // Core data
    float* rdata;           // Real space data
    int nx, ny, nz;         // Dimensions
    Dict attr_dict;         // Metadata
    
    // Functionality split into headers:
    #include "emdata_io.h"         // read_image(), write_image()
    #include "emdata_metadata.h"   // get_attr(), set_attr()
    #include "emdata_modular.h"    // process(), align(), etc.
    #include "emdata_transform.h"  // do_fft(), rotate(), etc.
    #include "emdata_core.h"       // get_data(), operators
};
```

### Important Methods

**File I/O**:
- `read_image(filename, img_index=0, header_only=false)`
- `write_image(filename, img_index=-1, img_type=EMUtil::IMAGE_UNKNOWN)`

**Processing**:
- `process_inplace(processor_name, params_dict)` - Modify in place
- `process(processor_name, params_dict)` - Return new image

**Transforms**:
- `do_fft()` / `do_fft_inplace()` - Forward FFT
- `do_ift()` / `do_ift_inplace()` - Inverse FFT
- `rotate(az, alt, phi)` - 3D rotation
- `translate(dx, dy, dz)` - Translation

**Data Access**:
- `get_data()` - Get raw float pointer
- `get_value_at(x, y, z)` - Get single pixel
- `set_value_at(x, y, z, value)` - Set single pixel
- `get_clip(region)` - Extract sub-region

**Metadata**:
- `set_attr(key, value)` - Set attribute
- `get_attr(key)` - Get attribute
- `has_attr(key)` - Check if attribute exists

## The Factory Pattern

The Factory pattern is central to EMAN2's plugin architecture.

### Why Factory Pattern?

1. **Extensibility**: Add new algorithms without modifying core code
2. **Runtime Selection**: Choose algorithm by name at runtime
3. **Discoverability**: List all available algorithms
4. **Consistency**: Uniform interface for all algorithms

### How It Works

```cpp
// 1. Define the Factory template (emobject.h)
template <class T>
class Factory {
    static T* get(const string& name);
    static T* get(const string& name, const Dict& params);
    static vector<string> get_list();
};

// 2. Each algorithm class defines:
class MyProcessor : public Processor {
    static const string NAME = "myprocessor";
    static Processor* NEW() { return new MyProcessor(); }
    // ... implementation
};

// 3. Register in factory constructor (processor.cpp)
template <> Factory<Processor>::Factory() {
    force_add<MyProcessor>();
    force_add<AnotherProcessor>();
    // ...
}

// 4. Use from code
Processor* p = Factory<Processor>::get("myprocessor");
p->process_inplace(image);
```

### Factory-Based Classes

All these use the Factory pattern:

| Class | Purpose | Example Names |
|-------|---------|---------------|
| **Processor** | Image processing | `filter.lowpass`, `normalize`, `mask.soft` |
| **Aligner** | Image alignment | `rotate_translate`, `refine` |
| **Averager** | Image averaging | `mean`, `ctf.auto` |
| **Cmp** | Image comparison | `ccc`, `frc`, `sqeuclidean` |
| **Projector** | 3D→2D projection | `standard`, `gauss_fft` |
| **Reconstructor** | 2D→3D reconstruction | `fourier`, `nn4`, `wiener_fourier` |
| **Analyzer** | Data analysis | `pca`, `kmeans` |

### Using Factories from Python

```python
from EMAN2 import *

# List all available processors
processors = Processors.get_list()
print(processors)  # ['filter.lowpass', 'normalize', ...]

# Get a specific processor
proc = Processors.get("filter.lowpass", {"cutoff_freq": 0.3})

# Or use directly on EMData
img.process_inplace("filter.lowpass", {"cutoff_freq": 0.3})
```

## Modular Classes

### Processor - Image Processing

**Purpose**: Apply transformations to images

**Base Class**: `libEM/processor.h`

**Common Processors**:
- `normalize` - Normalize mean/std
- `filter.lowpass` - Low-pass filter
- `filter.highpass` - High-pass filter
- `mask.soft` - Soft circular mask
- `math.fft.resample` - Resample in Fourier space

**Interface**:
```cpp
class Processor {
    virtual void process_inplace(EMData* image) = 0;
    virtual EMData* process(const EMData* image);
    virtual string get_name() const = 0;
    virtual Dict get_params() const;
    virtual void set_params(const Dict& params);
};
```

**Example**:
```python
# Apply Gaussian low-pass filter
img.process_inplace("filter.lowpass.gauss", {"cutoff_freq": 0.25})

# Chain multiple processors
img.process_inplace("normalize")
img.process_inplace("filter.lowpass", {"cutoff_freq": 0.3})
img.process_inplace("mask.soft", {"outer_radius": 64})
```

### Aligner - Image Alignment

**Purpose**: Align one image to another

**Base Class**: `libEM/aligner.h`

**Common Aligners**:
- `rotate_translate` - 2D rotation + translation
- `rotate_translate_flip` - Include mirror symmetry
- `refine` - Refinement around initial alignment

**Interface**:
```cpp
class Aligner {
    virtual EMData* align(EMData* to_img, EMData* from_img) const = 0;
    virtual EMData* align(EMData* to_img, EMData* from_img,
                          const string& cmp_name, const Dict& cmp_params) const = 0;
};
```

**Example**:
```python
# Align img2 to img1
aligned = img2.align("rotate_translate", img1, "ccc", {})

# Get alignment transform
xform = aligned["xform.align2d"]
```

### Averager - Image Averaging

**Purpose**: Average multiple images

**Base Class**: `libEM/averager.h`

**Common Averagers**:
- `mean` - Simple mean
- `ctf.auto` - CTF-corrected averaging
- `median` - Median averaging

**Interface**:
```cpp
class Averager {
    virtual void add_image(EMData* image) = 0;
    virtual EMData* finish() = 0;
};
```

**Example**:
```python
# Average a stack of images
avg = Averagers.get("mean")
for i in range(n):
    img = EMData("stack.hdf", i)
    avg.add_image(img)
result = avg.finish()
```

### Cmp - Image Comparison

**Purpose**: Compute similarity between images

**Base Class**: `libEM/cmp.h`

**Common Cmps**:
- `ccc` - Cross-correlation coefficient
- `frc` - Fourier ring correlation
- `sqeuclidean` - Squared Euclidean distance

**Interface**:
```cpp
class Cmp {
    virtual float cmp(EMData* img1, EMData* img2) const = 0;
};
```

**Example**:
```python
# Compare two images
cmp = Cmps.get("ccc")
similarity = cmp.cmp(img1, img2)
```

## Image I/O Architecture

### ImageIO Base Class

All file format handlers inherit from `ImageIO`:

```cpp
class ImageIO {
    virtual int read_header(Dict& dict, int image_index, 
                           const Region* area, bool is_3d) = 0;
    virtual int read_data(float* data, int image_index,
                         const Region* area, bool is_3d) = 0;
    virtual int write_header(const Dict& dict, int image_index,
                            const Region* area, ...) = 0;
    virtual int write_data(float* data, int image_index,
                          const Region* area, ...) = 0;
    virtual bool is_valid(const void* first_block) = 0;
};
```

### Format Detection

```cpp
// Automatic format detection
ImageType type = EMUtil::get_image_type(filename);

// Get appropriate I/O handler
ImageIO* io = EMUtil::get_imageio(filename, ImageIO::READ_ONLY, type);
```

### Supported Formats

- **MRC/MRCS**: Most common in cryo-EM (`libEM/io/mrcio.cpp`)
- **HDF5**: EMAN2's preferred format (`libEM/io/hdfio2.cpp`)
- **TIFF**: Standard image format (`libEM/io/tifio.cpp`)
- **DM3/DM4**: Gatan Digital Micrograph
- **SPIDER**: Common in single particle analysis
- **IMAGIC**: Another common format
- Plus 15+ more formats

## Transforms and Geometry

### Transform Class

Represents rotation, translation, and scale:

```cpp
Transform t;
t.set_rotation(Dict("type", "eman", "az", 45.0, "alt", 30.0, "phi", 0.0));
t.set_trans(10.0, 20.0, 0.0);
t.set_scale(1.5);

EMData* rotated = img->process("xform", Dict("transform", t));
```

### Geometry Classes

- **Vec3f**: 3D float vector
- **Vec3i**: 3D integer vector
- **Region**: Rectangular 2D/3D region
- **Pixel**: 3D coordinates + value

## Python-C++ Integration

### Boost.Python Wrappers

Each C++ module has a corresponding Python wrapper:

```cpp
// libpyEM/libpyEMData2.cpp
BOOST_PYTHON_MODULE(libpyEMData2) {
    class_<EMData>("EMData", init<>())
        .def("read_image", &EMData::read_image)
        .def("write_image", &EMData::write_image)
        .def("process_inplace", &EMData::process_inplace)
        // ... many more methods
    ;
}
```

### NumPy Integration

EMData can share memory with NumPy arrays:

```python
from EMAN2 import *
import numpy as np

# EMData to NumPy (shares memory!)
img = EMData("file.mrc")
arr = EMNumPy.em2numpy(img)
arr[0,0] = 100  # Modifies img too!

# NumPy to EMData (copies data)
arr = np.random.rand(128, 128).astype(np.float32)
img = EMNumPy.numpy2em(arr)
```

---

**Next**: Read [DEVELOPER_GUIDE.md](DEVELOPER_GUIDE.md) to learn how to build and extend EMAN2.

