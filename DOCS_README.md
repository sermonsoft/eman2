# EMAN2 Learning Documentation

Welcome! This directory contains comprehensive learning documentation for the EMAN2 codebase.

## 🚀 Quick Start

**New to EMAN2?** Start here:

1. **Read**: [LEARNING_GUIDE.md](LEARNING_GUIDE.md) - Overview and navigation
2. **Understand**: [ARCHITECTURE.md](ARCHITECTURE.md) - How it all fits together
3. **Learn**: [CORE_CONCEPTS.md](CORE_CONCEPTS.md) - Key abstractions
4. **Build**: [DEVELOPER_GUIDE.md](DEVELOPER_GUIDE.md) - Hands-on development
5. **Reference**: [API_REFERENCE.md](API_REFERENCE.md) - Quick API lookup

## 📚 Documentation Files

| File | Purpose | When to Read |
|------|---------|--------------|
| **[LEARNING_GUIDE.md](LEARNING_GUIDE.md)** | Entry point, overview, navigation | **Start here** |
| **[ARCHITECTURE.md](ARCHITECTURE.md)** | System design, components, data flow | Day 1-2 |
| **[CORE_CONCEPTS.md](CORE_CONCEPTS.md)** | EMData, Factory pattern, key classes | Day 2-3 |
| **[DEVELOPER_GUIDE.md](DEVELOPER_GUIDE.md)** | Building, testing, extending | When coding |
| **[API_REFERENCE.md](API_REFERENCE.md)** | Quick API reference | While coding |
| **[DOCUMENTATION_INDEX.md](DOCUMENTATION_INDEX.md)** | Complete index and learning paths | For navigation |

## 🎯 Choose Your Path

### I want to USE EMAN2
→ Read: LEARNING_GUIDE.md + API_REFERENCE.md  
→ Explore: `examples/` directory  
→ Time: 1-2 days

### I want to ADD ALGORITHMS
→ Read: All docs, focus on CORE_CONCEPTS.md (Factory pattern)  
→ Follow: DEVELOPER_GUIDE.md → "Adding New Features"  
→ Time: 3-5 days

### I want to CONTRIBUTE to CORE
→ Read: All documentation in order  
→ Study: Build system and Python bindings  
→ Time: 1-2 weeks

## 📖 What is EMAN2?

EMAN2 is an open-source software suite for **cryo-electron microscopy** (CryoEM) image processing:

- **Single particle analysis** - Determine 3D structures from 2D images
- **Electron tomography** - 3D reconstruction from tilt series
- **Image processing** - Filtering, alignment, classification
- **3D reconstruction** - Multiple algorithms for volume reconstruction

**Used by**: Structural biology researchers worldwide  
**License**: GPL2, BSD 3-Clause  
**Language**: C++ core with Python interface

## 🏗️ Architecture Overview

```
Applications (e2*.py programs)
        ↓
Python API (EMAN2.py, EMAN3.py)
        ↓
Boost.Python Bindings (libpy*.cpp)
        ↓
C++ Core Library (libEM)
        ↓
External Dependencies (HDF5, FFTW3, GSL, etc.)
```

**Key Components**:
- **EMData**: Core image/volume class
- **Factory Pattern**: Plugin architecture for algorithms
- **Modular Classes**: Processors, Aligners, Averagers, etc.
- **Image I/O**: 20+ file format handlers
- **Python Bindings**: Seamless C++/Python integration

## 🔑 Key Concepts

### EMData
The fundamental data structure - represents 1D/2D/3D images in real or Fourier space.

```python
from EMAN2 import *
img = EMData("file.mrc")
img.process_inplace("filter.lowpass", {"cutoff_freq": 0.3})
img.write_image("output.hdf")
```

### Factory Pattern
Algorithms are registered and accessed by name:

```python
# List all processors
processors = Processors.get_list()

# Use a processor
img.process_inplace("normalize")
```

### Modular Classes
- **Processor**: Image processing (100+ algorithms)
- **Aligner**: Image alignment (20+ methods)
- **Averager**: Image averaging (10+ methods)
- **Cmp**: Image comparison (15+ metrics)
- **Projector**: 3D→2D projection (5+ methods)
- **Reconstructor**: 2D→3D reconstruction (10+ algorithms)

## 📂 Directory Structure

```
eman2/
├── libEM/              # C++ core library
│   ├── emdata.*        # EMData class
│   ├── processor.*     # Image processors
│   ├── io/             # File format handlers
│   └── ...
├── libpyEM/            # Python bindings
│   ├── EMAN2.py        # Main Python API
│   ├── libpy*.cpp      # Boost.Python wrappers
│   └── ...
├── programs/           # Command-line tools (e2*.py)
├── sparx/              # SPARX framework
├── sphire/             # SPHIRE package
├── examples/           # Example scripts
└── tests/              # Test suite
```

## 🛠️ Quick Examples

### Load and Process Image
```python
from EMAN2 import *

img = EMData("input.mrc")
img.process_inplace("normalize")
img.process_inplace("filter.lowpass", {"cutoff_freq": 0.3})
img.write_image("output.hdf")
```

### Average Particles
```python
avg = Averagers.get("mean")
for i in range(n):
    ptcl = EMData("particles.hdf", i)
    avg.add_image(ptcl)
result = avg.finish()
```

### Align Images
```python
aligned = img2.align("rotate_translate", ref_img, "ccc", {})
xform = aligned["xform.align2d"]
```

## 🔗 Additional Resources

- **Official Wiki**: http://blake.bcm.edu/emanwiki
- **Homepage**: http://eman2.org
- **GitHub**: https://github.com/cryoem/eman2
- **Doxygen**: Build with `ENABLE_AUTODOC=ON`

## 💡 Learning Tips

1. **Start with LEARNING_GUIDE.md** - Don't skip this!
2. **Follow the data flow** - Trace how EMData moves through the system
3. **Read examples** - The `examples/` directory is invaluable
4. **Build it yourself** - Understanding comes from doing
5. **Use the tests** - They show correct API usage

## ✅ Success Checklist

You understand EMAN2 when you can:

- [ ] Explain what EMData is
- [ ] Describe the Factory pattern
- [ ] Load, process, and save images
- [ ] Navigate the codebase
- [ ] Build from source
- [ ] Add a simple Processor
- [ ] Write a Python script using EMAN2

## 📝 Documentation Status

**Created**: 2025  
**Version**: EMAN 2.99.70  
**Status**: Complete and current

**Files**:
- ✅ LEARNING_GUIDE.md (Overview & navigation)
- ✅ ARCHITECTURE.md (System design)
- ✅ CORE_CONCEPTS.md (Key abstractions)
- ✅ DEVELOPER_GUIDE.md (Building & extending)
- ✅ API_REFERENCE.md (API quick reference)
- ✅ DOCUMENTATION_INDEX.md (Complete index)

---

**Ready to learn?** → Start with [LEARNING_GUIDE.md](LEARNING_GUIDE.md)

**Commit Message:**
docs: add comprehensive learning documentation for EMAN2 codebase

This commit adds complete learning documentation including:
- LEARNING_GUIDE.md - Entry point and navigation
- ARCHITECTURE.md - System architecture and design
- CORE_CONCEPTS.md - Key abstractions (EMData, Factory, etc.)
- DEVELOPER_GUIDE.md - Building, testing, extending
- API_REFERENCE.md - Quick API reference
- DOCUMENTATION_INDEX.md - Complete index
- DOCS_README.md - Quick start guide

The documentation provides structured learning paths for users,
algorithm developers, and core contributors.
