# EMAN2 Developer Guide

This guide covers building EMAN2 from source, understanding the build system, adding new features, and testing.

## Table of Contents

1. [Building from Source](#building-from-source)
2. [Build System Overview](#build-system-overview)
3. [Adding New Features](#adding-new-features)
4. [Testing](#testing)
5. [Debugging Tips](#debugging-tips)
6. [Contributing](#contributing)

## Building from Source

### Prerequisites

EMAN2 is built within **Anaconda/Miniconda**. Building without Anaconda is not supported.

**Required**:
- Anaconda or Miniconda
- CMake >= 3.14
- C++ compiler with C++17 support (GCC >= 5, Clang, MSVC)
- Python 3.x

**Dependencies** (installed via conda):
- HDF5
- FFTW3
- GSL (GNU Scientific Library)
- Boost (with Boost.Python)
- NumPy
- Qt5 (for GUI)
- Optional: CUDA toolkit (for GPU acceleration)

### Build Steps

```bash
# 1. Create conda environment
conda create -n eman2-dev python=3.9
conda activate eman2-dev

# 2. Install dependencies
conda install cmake hdf5 fftw gsl boost numpy pyqt

# 3. Clone repository
git clone https://github.com/cryoem/eman2.git
cd eman2

# 4. Create build directory
mkdir build
cd build

# 5. Configure with CMake
cmake .. \
    -DCMAKE_BUILD_TYPE=Release \
    -DENABLE_FFTW3=ON \
    -DENABLE_OPENGL=ON \
    -DENABLE_AUTODOC=OFF

# 6. Build
make -j$(nproc)

# 7. Install
make install
```

### Build Options

Key CMake options (set with `-DOPTION=ON/OFF`):

| Option | Default | Description |
|--------|---------|-------------|
| `ENABLE_FFTW3` | ON | Use FFTW3 for FFT (recommended) |
| `ENABLE_OPENGL` | ON | Enable OpenGL visualization |
| `ENABLE_AUTODOC` | OFF | Generate Doxygen documentation |
| `ENABLE_EMAN_CUDA` | OFF | Enable CUDA support for EMAN2 |
| `ENABLE_SPARX_CUDA` | OFF | Enable CUDA support for SPARX |
| `ENABLE_TIFF` | ON | TIFF format support |
| `ENABLE_JPEG` | ON | JPEG format support |
| `ENABLE_PNG` | ON | PNG format support |
| `ENABLE_RT` | OFF | Enable regression tests |

### Platform-Specific Notes

**Linux**:
- Most straightforward platform
- Use system package manager for some dependencies if needed

**macOS**:
- May need to install Xcode Command Line Tools
- OpenGL support may require additional configuration

**Windows**:
- Use Visual Studio 2019 or later
- Build within Anaconda Prompt
- Some features may be limited

## Build System Overview

### CMake Structure

```
eman2/
├── CMakeLists.txt              # Root build configuration
├── cmake/
│   ├── find_all.cmake          # Find all dependencies
│   ├── functions.cmake         # Helper functions
│   ├── FindFFTW3.cmake         # Find FFTW3
│   ├── HDF5.cmake              # Find HDF5
│   └── ...                     # Other find modules
├── libEM/CMakeLists.txt        # Build libEM2 library
├── libpyEM/CMakeLists.txt      # Build Python bindings
├── programs/CMakeLists.txt     # Install programs
├── sparx/CMakeLists.txt        # Build SPARX
└── sphire/CMakeLists.txt       # Build SPHIRE
```

### Build Targets

```bash
# Build everything
make

# Build specific targets
make EM2                # C++ core library
make libpyEMData2       # Python bindings for EMData
make libpyProcessor2    # Python bindings for Processor

# Install
make install            # Install to conda environment
make install-sphire     # Install SPHIRE separately

# Documentation
make mkdoc              # Generate Doxygen docs (if ENABLE_AUTODOC=ON)

# Testing
make test               # Run tests (if ENABLE_RT=ON)
```

### Key Build Outputs

- `libEM2.so` (or `.dylib`/`.dll`) - Core C++ library
- `libpy*.so` - Python extension modules
- Python files installed to `$CONDA_PREFIX/lib/pythonX.Y/site-packages/`
- Programs installed to `$CONDA_PREFIX/bin/`

## Adding New Features

### Adding a New Processor

Processors are the most common extension point. Here's how to add one:

**1. Define the class** in `libEM/processor.h`:

```cpp
class MyNewProcessor : public Processor {
public:
    void process_inplace(EMData* image);
    
    string get_name() const { return NAME; }
    static Processor* NEW() { return new MyNewProcessor(); }
    
    string get_desc() const {
        return "Description of what this processor does";
    }
    
    TypeDict get_param_types() const {
        TypeDict d;
        d.put("threshold", EMObject::FLOAT, "Threshold value");
        d.put("mode", EMObject::INT, "Processing mode");
        return d;
    }
    
    static const string NAME;
};
```

**2. Implement the class** in `libEM/processor.cpp`:

```cpp
const string MyNewProcessor::NAME = "mynew";

void MyNewProcessor::process_inplace(EMData* image) {
    // Get parameters
    float threshold = params.set_default("threshold", 0.5f);
    int mode = params.set_default("mode", 0);
    
    // Get image data
    float* data = image->get_data();
    int nx = image->get_xsize();
    int ny = image->get_ysize();
    int nz = image->get_zsize();
    size_t size = (size_t)nx * ny * nz;
    
    // Process the image
    for (size_t i = 0; i < size; ++i) {
        if (data[i] > threshold) {
            data[i] *= 2.0f;  // Example operation
        }
    }
    
    // Update image statistics
    image->update();
}
```

**3. Register in factory** in `libEM/processor.cpp`:

```cpp
template <> Factory<Processor>::Factory() {
    // ... existing processors ...
    force_add<MyNewProcessor>();
}
```

**4. Rebuild**:

```bash
cd build
make -j$(nproc)
make install
```

**5. Use from Python**:

```python
from EMAN2 import *

img = EMData("input.mrc")
img.process_inplace("mynew", {"threshold": 0.7, "mode": 1})
img.write_image("output.mrc")
```

### Adding a New File Format

**1. Create header** `libEM/io/myformatio.h`:

```cpp
class MyFormatIO : public ImageIO {
public:
    explicit MyFormatIO(const string& fname, IOMode rw);
    ~MyFormatIO();
    
    DEFINE_IMAGEIO_FUNC;
    static bool is_valid(const void* first_block);
    
private:
    // Format-specific members
    FILE* file;
    // ...
};
```

**2. Implement** `libEM/io/myformatio.cpp`:

```cpp
bool MyFormatIO::is_valid(const void* first_block) {
    // Check magic number or header signature
    const char* data = static_cast<const char*>(first_block);
    return (strncmp(data, "MYFORMAT", 8) == 0);
}

int MyFormatIO::read_header(Dict& dict, int image_index, 
                            const Region* area, bool is_3d) {
    // Read header, populate dict with metadata
    // ...
}

int MyFormatIO::read_data(float* data, int image_index,
                          const Region* area, bool is_3d) {
    // Read pixel data into float array
    // ...
}

// Implement write_header, write_data similarly
```

**3. Register format** in `libEM/emutil.cpp`:

```cpp
ImageType EMUtil::get_image_type(const string& filename) {
    // ... existing checks ...
    else if (MyFormatIO::is_valid(first_block)) {
        image_type = IMAGE_MYFORMAT;
    }
    // ...
}

ImageIO* EMUtil::get_imageio(const string& filename, int rw,
                             ImageType image_type) {
    // ... existing cases ...
    case IMAGE_MYFORMAT:
        imageio = new MyFormatIO(filename, rw_mode);
        break;
    // ...
}
```

**4. Add to** `libEM/io/all_imageio.h`:

```cpp
#include "myformatio.h"
```

### Adding Python-Only Extensions

You can extend EMAN2 in pure Python without touching C++:

```python
# myextensions.py
from EMAN2 import *

def my_custom_workflow(input_stack, output_file):
    """Custom processing workflow"""
    avg = Averagers.get("mean")
    
    n = EMUtil.get_image_count(input_stack)
    for i in range(n):
        img = EMData(input_stack, i)
        img.process_inplace("normalize")
        img.process_inplace("filter.lowpass", {"cutoff_freq": 0.3})
        avg.add_image(img)
    
    result = avg.finish()
    result.write_image(output_file)
    return result

# Use it
if __name__ == "__main__":
    my_custom_workflow("particles.hdf", "average.hdf")
```

## Testing

### Running Tests

```bash
# Build with testing enabled
cmake .. -DENABLE_RT=ON
make

# Run all tests
make test

# Or use ctest directly
ctest -V  # Verbose output
ctest -R processor  # Run only processor tests
```

### Test Structure

```
rt/                    # Regression tests
├── pyem/              # Python API tests
├── imageio/           # Image I/O tests
└── demo/              # Demo tests

tests/                 # Additional tests
├── test_imports.py    # Test Python imports
└── run_prog_tests.py  # Test programs
```

### Writing Unit Tests

For C++ (if using Catch2):

```cpp
#include <catch2/catch.hpp>
#include "emdata.h"

TEST_CASE("EMData basic operations", "[emdata]") {
    EMData img;
    img.set_size(64, 64, 1);
    
    REQUIRE(img.get_xsize() == 64);
    REQUIRE(img.get_ysize() == 64);
    REQUIRE(img.get_zsize() == 1);
}
```

For Python:

```python
import unittest
from EMAN2 import *

class TestMyProcessor(unittest.TestCase):
    def test_basic_processing(self):
        img = EMData()
        img.set_size(64, 64, 1)
        img.to_one()
        
        img.process_inplace("mynew", {"threshold": 0.5})
        
        # Verify results
        self.assertGreater(img.get_attr("mean"), 0)

if __name__ == '__main__':
    unittest.main()
```

## Debugging Tips

### Enable Debug Build

```bash
cmake .. -DCMAKE_BUILD_TYPE=Debug
make
```

### Using GDB

```bash
gdb python
(gdb) run my_script.py
(gdb) bt  # Backtrace on crash
```

### Print Debugging in C++

```cpp
#include "log.h"

void MyProcessor::process_inplace(EMData* image) {
    LOGDEBUG("Processing image: %dx%d", image->get_xsize(), image->get_ysize());
    // ...
}
```

### Common Issues

**Issue**: `ImportError: cannot import name 'EMData'`  
**Solution**: Rebuild and reinstall Python bindings

**Issue**: Segmentation fault in C++ code  
**Solution**: Check for null pointers, array bounds, memory management

**Issue**: CMake can't find dependencies  
**Solution**: Ensure conda environment is activated, check `CMAKE_PREFIX_PATH`

## Contributing

### Code Style

- **C++**: Follow existing style (mostly K&R with tabs)
- **Python**: PEP 8 style
- **Comments**: Document public APIs, complex algorithms

### Commit Guidelines

- Write clear commit messages
- One logical change per commit
- Test before committing

### Pull Request Process

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Ensure all tests pass
6. Submit pull request with description

---

**Next**: Read [API_REFERENCE.md](API_REFERENCE.md) for quick API lookup.

