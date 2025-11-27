# EMAN2 Architecture

This document explains the overall architecture of EMAN2, how components relate to each other, and the design principles that guide the system.

## Table of Contents

1. [Layered Architecture](#layered-architecture)
2. [Core Components](#core-components)
3. [Data Flow](#data-flow)
4. [Design Patterns](#design-patterns)
5. [Module Relationships](#module-relationships)

## Layered Architecture

EMAN2 follows a classic layered architecture with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────────────┐
│  Layer 4: Applications & Tools                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ Command-line │  │  GUI Apps    │  │   Workflows  │          │
│  │ Tools (e2*)  │  │  (Qt-based)  │  │   (SPARX)    │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│  Layer 3: Python API & Bindings                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  EMAN2.py    │  │  EMAN3.py    │  │ EMAN2db.py   │          │
│  │  (Classic)   │  │  (Modern)    │  │ (Database)   │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                   │
│  Boost.Python Wrappers (libpy*.cpp)                             │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│  Layer 2: C++ Core Library (libEM)                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Data Model  │  │  Algorithms  │  │   I/O        │          │
│  │  (EMData)    │  │  (Factories) │  │  (ImageIO)   │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Transforms  │  │  Geometry    │  │   Utilities  │          │
│  │  (FFT, etc)  │  │  (Vec3, etc) │  │  (EMUtil)    │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│  Layer 1: External Dependencies                                  │
│  HDF5 | FFTW3 | GSL | Boost | NumPy | Qt | CUDA (optional)     │
└─────────────────────────────────────────────────────────────────┘
```

### Layer Responsibilities

**Layer 1: External Dependencies**
- Provides foundational functionality (FFT, linear algebra, file I/O)
- EMAN2 wraps these to provide consistent interfaces

**Layer 2: C++ Core Library (libEM)**
- Core data structures (EMData, Transform, Region, etc.)
- Algorithm implementations via Factory pattern
- File format handlers
- Performance-critical operations

**Layer 3: Python API & Bindings**
- Exposes C++ functionality to Python
- Adds Python-specific conveniences
- Provides high-level workflows
- Database and metadata management

**Layer 4: Applications & Tools**
- End-user programs (e2*.py)
- GUI applications
- Workflow frameworks (SPARX, SPHIRE)

## Core Components

### 1. EMData - The Central Data Structure

`EMData` is the heart of EMAN2. It represents a 1D, 2D, or 3D image in either real or Fourier space.

**Location**: `libEM/emdata.h`, `libEM/emdata_*.cpp`

**Key Features**:
- Stores pixel data as float array
- Metadata dictionary (Dict) for attributes
- Supports real and complex (Fourier) data
- Memory-efficient region operations
- Integrated with all processing algorithms

**Modular Design**:
```cpp
class EMData {
    #include "emdata_io.h"         // File I/O operations
    #include "emdata_metadata.h"   // Attribute get/set
    #include "emdata_modular.h"    // Process, align, etc.
    #include "emdata_transform.h"  // FFT, transforms
    #include "emdata_core.h"       // Basic operations
    #include "sparx/emdata_sparx.h"   // SPARX extensions
    #include "sphire/emdata_sphire.h" // SPHIRE extensions
};
```

### 2. Factory Pattern - Algorithm Registration

EMAN2 uses the Factory pattern extensively for algorithm plugins.

**Location**: `libEM/emobject.h` (Factory template)

**Modular Classes** (all use Factory pattern):
- **Processor**: Image processing operations
- **Aligner**: Image alignment algorithms
- **Averager**: Image averaging methods
- **Cmp**: Image comparison/similarity metrics
- **Projector**: 3D→2D projection methods
- **Reconstructor**: 2D→3D reconstruction algorithms
- **Analyzer**: Data analysis tools

**How it works**:
```cpp
// 1. Define a new processor
class MyProcessor : public Processor {
    static const string NAME = "myprocessor";
    static Processor *NEW() { return new MyProcessor(); }
    void process_inplace(EMData *image) { /* ... */ }
};

// 2. Register in factory (processor.cpp)
template <> Factory<Processor>::Factory() {
    force_add<MyProcessor>();
    // ... other processors
}

// 3. Use from Python
img.process_inplace("myprocessor", {"param": value})
```

### 3. Image I/O System

Pluggable architecture for reading/writing various file formats.

**Location**: `libEM/io/`

**Base Class**: `ImageIO` (`libEM/io/imageio.h`)

**Supported Formats**:
- MRC/MRCS (most common in cryo-EM)
- HDF5 (EMAN2's preferred format)
- TIFF, JPEG, PNG
- DM3/DM4 (Gatan)
- SPIDER, IMAGIC
- Many others...

**Architecture**:
```
ImageIO (abstract base)
  ├── MrcIO
  ├── HdfIO / HdfIO2
  ├── TiffIO
  ├── ImagicIO
  └── ... (20+ formats)
```

**Format Detection**: `EMUtil::get_image_type()` auto-detects format from file header

### 4. Python Bindings

Boost.Python wraps C++ classes for Python access.

**Location**: `libpyEM/`

**Wrapper Modules**:
- `libpyEMData2.cpp` → EMData class
- `libpyProcessor2.cpp` → Processor factory
- `libpyAligner2.cpp` → Aligner factory
- `libpyAverager2.cpp` → Averager factory
- `libpyCmp2.cpp` → Cmp factory
- `libpyProjector2.cpp` → Projector factory
- `libpyReconstructor2.cpp` → Reconstructor factory
- `libpyUtils2.cpp` → Utility functions
- `libpyTypeConverter2.cpp` → NumPy integration

**Integration**:
```python
# All wrappers imported in EMAN2_cppwrap.py
from libpyEMData2 import *
from libpyProcessor2 import *
# ... etc

# Then exposed via EMAN2.py
from EMAN2_cppwrap import *
```

## Data Flow

### Typical Image Processing Workflow

```
1. Load Image
   ┌─────────────────────────────────────┐
   │ EMData img = EMData("file.mrc")     │
   │   ↓                                  │
   │ EMUtil::get_imageio("file.mrc")     │
   │   ↓                                  │
   │ MrcIO::read_data()                  │
   └─────────────────────────────────────┘
                ↓
2. Process Image
   ┌─────────────────────────────────────┐
   │ img.process_inplace("filter.lowpass",│
   │                     {"cutoff": 0.5}) │
   │   ↓                                  │
   │ Factory<Processor>::get("filter...")│
   │   ↓                                  │
   │ LowPassProcessor::process_inplace() │
   └─────────────────────────────────────┘
                ↓
3. Transform
   ┌─────────────────────────────────────┐
   │ img.do_fft_inplace()                │
   │   ↓                                  │
   │ EMFFT::real_to_complex_1d()         │
   │   ↓                                  │
   │ FFTW3 library                       │
   └─────────────────────────────────────┘
                ↓
4. Save Result
   ┌─────────────────────────────────────┐
   │ img.write_image("output.hdf")       │
   │   ↓                                  │
   │ EMUtil::get_imageio("output.hdf")   │
   │   ↓                                  │
   │ HdfIO2::write_data()                │
   └─────────────────────────────────────┘
```

## Design Patterns

### 1. Factory Pattern
**Purpose**: Plugin architecture for algorithms  
**Used in**: All modular classes (Processor, Aligner, etc.)  
**Benefit**: Easy to add new algorithms without modifying core code

### 2. Strategy Pattern
**Purpose**: Interchangeable algorithms  
**Used in**: Comparison (Cmp), Alignment (Aligner)  
**Benefit**: Runtime algorithm selection

### 3. Template Method Pattern
**Purpose**: Define algorithm skeleton, let subclasses fill in details  
**Used in**: ImageIO base class  
**Benefit**: Consistent interface across file formats

### 4. Singleton Pattern
**Purpose**: Single instance of Factory  
**Used in**: Factory<T> template  
**Benefit**: Global algorithm registry

## Module Relationships

### libEM Internal Structure

```
libEM/
├── Core Data Structures
│   ├── emdata.* (EMData class)
│   ├── emobject.* (EMObject, Dict, Factory)
│   ├── transform.* (Transform class)
│   └── geometry.* (Region, Vec3, etc.)
│
├── Modular Algorithms
│   ├── processor.* (100+ image processors)
│   ├── aligner.* (20+ alignment algorithms)
│   ├── averager.* (10+ averaging methods)
│   ├── cmp.* (15+ comparison metrics)
│   ├── projector.* (5+ projection methods)
│   ├── reconstructor.* (10+ reconstruction algorithms)
│   └── analyzer.* (analysis tools)
│
├── I/O System
│   └── io/ (20+ file format handlers)
│
├── Math & Transforms
│   ├── emfft.* (FFT operations)
│   ├── interp.* (interpolation)
│   └── symmetry.* (symmetry operations)
│
└── Extensions
    ├── sparx/ (SPARX-specific code)
    └── sphire/ (SPHIRE-specific code)
```

### Dependency Graph

```
programs/*.py
    ↓ imports
libpyEM/EMAN2.py
    ↓ imports
libpyEM/EMAN2_cppwrap.py
    ↓ imports
libpyEM/libpy*.so (Boost.Python modules)
    ↓ links to
libEM/libEM2.so (C++ library)
    ↓ links to
External libs (HDF5, FFTW3, GSL, etc.)
```

## SPARX and SPHIRE Integration

### SPARX
- **Purpose**: Single Particle Analysis framework
- **Location**: `sparx/` and `libEM/sparx/`
- **Integration**: Extends EMData with SPARX-specific methods
- **Language**: Python + C++ extensions

### SPHIRE
- **Purpose**: Advanced cryo-EM processing package
- **Location**: `sphire/` and `libEM/sphire/`
- **Integration**: Built on EMAN2 environment
- **Language**: Python + C++ extensions
- **Requirement**: Needs EMAN2 >= 2.9

Both are **separate packages** that extend EMAN2's core functionality.

---

**Next**: Read [CORE_CONCEPTS.md](CORE_CONCEPTS.md) to understand key abstractions in detail.

