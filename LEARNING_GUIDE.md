# EMAN2 Learning Guide

Welcome to the EMAN2 codebase! This guide will help you navigate and understand this comprehensive cryo-electron microscopy image processing suite.

## 📚 Documentation Structure

This learning documentation is organized into several interconnected guides:

1. **LEARNING_GUIDE.md** (this file) - Start here for overview and navigation
2. **ARCHITECTURE.md** - System architecture and component relationships
3. **CORE_CONCEPTS.md** - Key abstractions, design patterns, and data structures
4. **DEVELOPER_GUIDE.md** - Building, testing, and extending EMAN2
5. **API_REFERENCE.md** - Quick reference for main classes and functions

## 🎯 What is EMAN2?

EMAN2 is an open-source software suite for **single particle analysis** and **electron micrograph analysis** in cryo-electron microscopy (CryoEM) and cryo-electron tomography (CryoET). It's used by researchers worldwide to determine high-resolution 3D structures of biological macromolecules.

### Key Capabilities

- **Image Processing**: 2D/3D image manipulation, filtering, FFT operations
- **Particle Picking**: Automated and manual particle selection from micrographs
- **CTF Correction**: Contrast transfer function estimation and correction
- **3D Reconstruction**: Multiple algorithms for reconstructing 3D volumes
- **Classification**: Clustering and classification of particle images
- **Refinement**: Iterative refinement of 3D structures

## 🏗️ High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    User Applications                         │
│  (programs/*.py - e2*.py command-line tools & GUIs)         │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                   Python Layer (libpyEM)                     │
│  • EMAN2.py - Main Python API                               │
│  • EMAN3.py - Modern API with TensorFlow/JAX support        │
│  • Boost.Python wrappers (libpy*.cpp)                       │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                   C++ Core Library (libEM)                   │
│  • EMData - Core image class                                 │
│  • Processors, Aligners, Averagers, Cmps                    │
│  • Projectors, Reconstructors, Analyzers                    │
│  • Image I/O (MRC, HDF5, TIFF, etc.)                        │
│  • FFT, transforms, geometry                                 │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│              External Dependencies                           │
│  HDF5, FFTW3, GSL, Boost, NumPy, Qt (optional CUDA)        │
└─────────────────────────────────────────────────────────────┘
```

## 📂 Directory Structure

```
eman2/
├── libEM/              # C++ core library
│   ├── emdata.h/cpp    # Main image class
│   ├── processor.h/cpp # Image processing algorithms
│   ├── aligner.h/cpp   # Image alignment algorithms
│   ├── io/             # File format handlers
│   ├── sparx/          # SPARX-specific extensions
│   └── sphire/         # SPHIRE-specific extensions
│
├── libpyEM/            # Python bindings
│   ├── EMAN2.py        # Main Python API
│   ├── EMAN3.py        # Modern API
│   ├── libpy*.cpp      # Boost.Python wrappers
│   └── qtgui/          # Qt GUI components
│
├── programs/           # Command-line tools & applications
│   ├── e2*.py          # EMAN2 programs
│   └── ...
│
├── sparx/              # SPARX framework
│   ├── bin/            # SPARX executables
│   └── libpy/          # SPARX Python libraries
│
├── sphire/             # SPHIRE package
│   ├── bin_py3/        # SPHIRE executables
│   └── libpy_py3/      # SPHIRE Python libraries
│
├── examples/           # Example scripts
├── tests/              # Test suite
├── doc/                # Documentation
└── cmake/              # Build configuration
```

## 🚀 Quick Start Learning Path

### For Users
1. Read the [official wiki](http://blake.bcm.edu/emanwiki)
2. Explore `programs/` directory to see available tools
3. Try example scripts in `examples/`

### For Developers
1. **Start Here**: Read `ARCHITECTURE.md` to understand the system design
2. **Core Concepts**: Study `CORE_CONCEPTS.md` for key abstractions
3. **Build & Test**: Follow `DEVELOPER_GUIDE.md` to set up your environment
4. **API Reference**: Use `API_REFERENCE.md` as you code
5. **Extend**: Learn to add new Processors, Aligners, etc.

## 🔑 Key Concepts to Understand

1. **EMData** - The central image/volume data structure
2. **Factory Pattern** - How algorithms are registered and instantiated
3. **Modular Classes** - Processors, Aligners, Averagers, Cmps, Projectors, Reconstructors
4. **Image I/O** - Plugin architecture for file formats
5. **Python Bindings** - How C++ is exposed to Python via Boost.Python
6. **SPARX vs SPHIRE** - Two frameworks built on EMAN2

## 📖 Recommended Reading Order

1. **Day 1**: This guide + ARCHITECTURE.md
2. **Day 2**: CORE_CONCEPTS.md (EMData, Factory pattern)
3. **Day 3**: DEVELOPER_GUIDE.md (build system, testing)
4. **Day 4**: Study one modular class (e.g., Processor)
5. **Day 5**: Explore Image I/O system
6. **Day 6+**: Deep dive into specific areas of interest

## 🔍 Finding Your Way Around

### To understand how images are processed:
- Start with `libEM/emdata.h` and `libEM/emdata_core.h`
- Look at `libEM/processor.h` for processing algorithms
- Check `programs/e2proc2d.py` for practical usage

### To understand file I/O:
- Start with `libEM/io/imageio.h`
- Look at `libEM/io/mrcio.cpp` for MRC format example
- Check `libEM/emutil.cpp::get_imageio()` for format detection

### To understand Python bindings:
- Start with `libpyEM/EMAN2_cppwrap.py`
- Look at `libpyEM/libpyEMData2.cpp` for Boost.Python example
- Check `libpyEM/EMAN2.py` for the complete Python API

### To understand the build system:
- Start with root `CMakeLists.txt`
- Look at `cmake/find_all.cmake` for dependencies
- Check `libEM/CMakeLists.txt` for library structure

## 💡 Tips for Learning

1. **Use the examples**: The `examples/` directory has many small, focused scripts
2. **Read the tests**: Tests in `rt/` and `tests/` show how to use the API
3. **Follow the data flow**: Trace how an EMData object flows through processing
4. **Use Doxygen**: Build documentation with `ENABLE_AUTODOC=ON`
5. **Ask questions**: The EMAN2 community is helpful

## 🛠️ Next Steps

Choose your path:

- **Want to understand the architecture?** → Read `ARCHITECTURE.md`
- **Want to understand core concepts?** → Read `CORE_CONCEPTS.md`
- **Want to build and extend?** → Read `DEVELOPER_GUIDE.md`
- **Need API reference?** → Read `API_REFERENCE.md`

---

**Version**: EMAN 2.99.70  
**License**: GPL2, BSD 3-Clause  
**Homepage**: http://blake.bcm.edu/emanwiki

