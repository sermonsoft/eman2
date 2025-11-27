# EMAN2 Documentation Index

Complete learning documentation for the EMAN2 codebase.

## 📚 Documentation Files

### 1. [LEARNING_GUIDE.md](LEARNING_GUIDE.md) - **START HERE**
Your entry point to understanding EMAN2. Provides:
- Overview of what EMAN2 is and what it does
- High-level architecture diagram
- Directory structure explanation
- Recommended learning path
- Navigation guide to other documentation

**Read this first** to get oriented.

### 2. [ARCHITECTURE.md](ARCHITECTURE.md)
Deep dive into the system architecture:
- Layered architecture (Applications → Python → C++ → Dependencies)
- Core components (EMData, Factory pattern, I/O system)
- Data flow through the system
- Design patterns used throughout
- Module relationships and dependencies
- SPARX and SPHIRE integration

**Read this second** to understand how everything fits together.

### 3. [CORE_CONCEPTS.md](CORE_CONCEPTS.md)
Fundamental concepts and abstractions:
- **EMData** - The central image/volume class
- **Factory Pattern** - How algorithms are registered and used
- **Modular Classes** - Processors, Aligners, Averagers, Cmps, etc.
- **Image I/O** - File format handling
- **Transforms** - Rotations, translations, FFT
- **Python-C++ Integration** - Boost.Python bindings

**Read this third** to understand the key abstractions.

### 4. [DEVELOPER_GUIDE.md](DEVELOPER_GUIDE.md)
Practical guide for developers:
- Building from source (with Anaconda)
- CMake build system overview
- Adding new Processors, file formats, etc.
- Testing framework
- Debugging tips
- Contributing guidelines

**Read this** when you're ready to build and extend EMAN2.

### 5. [API_REFERENCE.md](API_REFERENCE.md)
Quick reference for common APIs:
- EMData class methods
- Factory classes (Processors, Aligners, etc.)
- Geometry classes (Vec3f, Transform, Region)
- Utility functions
- Common processors and their parameters
- File I/O operations
- CTF handling
- NumPy integration
- Common workflows and examples

**Use this** as a quick lookup while coding.

## 🎯 Learning Paths

### Path 1: User/Scripter
*Goal: Use EMAN2 for image processing tasks*

1. Read: LEARNING_GUIDE.md (overview)
2. Read: API_REFERENCE.md (focus on EMData and common processors)
3. Explore: `examples/` directory
4. Try: Write simple Python scripts using EMAN2
5. Reference: API_REFERENCE.md as needed

**Time**: 1-2 days

### Path 2: Algorithm Developer
*Goal: Add new processing algorithms*

1. Read: LEARNING_GUIDE.md
2. Read: ARCHITECTURE.md (understand the system)
3. Read: CORE_CONCEPTS.md (focus on Factory pattern)
4. Read: DEVELOPER_GUIDE.md (focus on "Adding New Features")
5. Practice: Add a simple Processor
6. Reference: Existing processors in `libEM/processor.cpp`

**Time**: 3-5 days

### Path 3: Core Developer
*Goal: Contribute to EMAN2 core*

1. Read: All documentation in order
2. Study: Build system (CMakeLists.txt files)
3. Study: Python bindings (libpyEM/*.cpp)
4. Study: Core classes (libEM/emdata.*, libEM/processor.*, etc.)
5. Practice: Build from source, run tests
6. Contribute: Fix bugs, add features

**Time**: 1-2 weeks

### Path 4: Format Support Developer
*Goal: Add support for new file formats*

1. Read: LEARNING_GUIDE.md
2. Read: ARCHITECTURE.md (focus on I/O system)
3. Read: CORE_CONCEPTS.md (focus on Image I/O)
4. Read: DEVELOPER_GUIDE.md (focus on "Adding a New File Format")
5. Study: Existing format handlers in `libEM/io/`
6. Practice: Implement a simple format handler

**Time**: 2-4 days

## 🔍 Quick Reference

### Finding Information

| I want to... | Look at... |
|--------------|------------|
| Understand what EMAN2 does | LEARNING_GUIDE.md |
| See the big picture | ARCHITECTURE.md |
| Understand EMData | CORE_CONCEPTS.md → EMData section |
| Add a new Processor | DEVELOPER_GUIDE.md → Adding New Features |
| Use a specific API | API_REFERENCE.md |
| Build from source | DEVELOPER_GUIDE.md → Building from Source |
| Understand Factory pattern | CORE_CONCEPTS.md → Factory Pattern |
| Add file format support | DEVELOPER_GUIDE.md → Adding a New File Format |
| Find example code | `examples/` directory + API_REFERENCE.md |
| Run tests | DEVELOPER_GUIDE.md → Testing |

### Key Files in Codebase

| File | Purpose |
|------|---------|
| `libEM/emdata.h` | EMData class definition |
| `libEM/processor.h` | Processor base class |
| `libEM/processor.cpp` | Processor implementations |
| `libEM/io/imageio.h` | ImageIO base class |
| `libEM/io/mrcio.cpp` | MRC format handler (good example) |
| `libpyEM/EMAN2.py` | Main Python API |
| `libpyEM/libpyEMData2.cpp` | EMData Python bindings |
| `programs/e2proc2d.py` | Example command-line tool |
| `CMakeLists.txt` | Root build configuration |

## 📖 Additional Resources

### Official Resources
- **Wiki**: http://blake.bcm.edu/emanwiki
- **Homepage**: http://eman2.org
- **GitHub**: https://github.com/cryoem/eman2

### In-Code Documentation
- **Doxygen**: Build with `ENABLE_AUTODOC=ON` for API docs
- **Docstrings**: Many Python functions have docstrings
- **Comments**: C++ code has inline comments

### Community
- **Mailing List**: Check EMAN2 wiki for details
- **Issues**: GitHub issue tracker

## 🎓 Learning Tips

1. **Start Small**: Don't try to understand everything at once
2. **Follow the Data**: Trace how an EMData object flows through the system
3. **Read Examples**: The `examples/` directory has many small, focused scripts
4. **Use the Tests**: Tests show how to use the API correctly
5. **Build It**: Actually building from source helps you understand the structure
6. **Ask Questions**: The EMAN2 community is helpful

## 📝 Documentation Maintenance

These documentation files were created to help developers learn the EMAN2 codebase. They should be updated when:

- Major architectural changes occur
- New core features are added
- APIs change significantly
- Build process changes

To update:
1. Edit the relevant .md file
2. Keep examples current
3. Update cross-references
4. Test code examples

## 🏆 Success Criteria

You'll know you understand EMAN2 when you can:

- [ ] Explain what EMData is and how it's used
- [ ] Describe the Factory pattern and why it's used
- [ ] Write a simple Processor
- [ ] Load, process, and save images using Python
- [ ] Navigate the codebase to find relevant code
- [ ] Build EMAN2 from source
- [ ] Add a new feature and test it
- [ ] Understand how Python and C++ interact

---

**Version**: EMAN 2.99.70  
**Documentation Created**: 2025  
**License**: GPL2, BSD 3-Clause

**Commit Message:**
docs: create comprehensive learning documentation for EMAN2 codebase
