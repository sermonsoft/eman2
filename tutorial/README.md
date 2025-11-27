# EMAN2 Tutorial - Complete Learning Guide

Welcome to the comprehensive EMAN2 learning tutorial! This folder contains everything you need to understand and work with the EMAN2 codebase.

## 🎯 What You'll Learn

By completing this tutorial, you will:
- Understand the EMAN2 architecture and design
- Master the core concepts (EMData, Factory pattern, etc.)
- Learn to build and extend EMAN2
- Be able to add new algorithms and features
- Write effective Python scripts using EMAN2
- Navigate the codebase confidently

## 📚 Tutorial Structure

This tutorial consists of 7 comprehensive documents:

### 1. 🚀 [LEARNING_GUIDE.md](LEARNING_GUIDE.md) - **START HERE**
**Time**: 30 minutes  
**Your entry point** to the EMAN2 codebase.

**What's inside**:
- Overview of EMAN2 and its capabilities
- High-level architecture diagram
- Directory structure walkthrough
- Recommended learning paths
- Navigation guide to other documents

**Start here** to get oriented and understand what EMAN2 does.

---

### 2. 🏗️ [ARCHITECTURE.md](ARCHITECTURE.md)
**Time**: 1-2 hours  
**Deep dive** into system design.

**What's inside**:
- Layered architecture (Applications → Python → C++ → Dependencies)
- Core components (EMData, Factory, I/O system)
- Data flow through the system
- Design patterns (Factory, Strategy, Template Method)
- Module relationships and dependencies
- SPARX and SPHIRE integration

**Read this** to understand how everything fits together.

---

### 3. 🔑 [CORE_CONCEPTS.md](CORE_CONCEPTS.md)
**Time**: 2-3 hours  
**Master** the fundamental abstractions.

**What's inside**:
- **EMData** - The core image/volume class
- **Factory Pattern** - Plugin architecture for algorithms
- **Modular Classes** - Processors, Aligners, Averagers, Cmps, Projectors, Reconstructors
- **Image I/O** - File format handling
- **Transforms** - Rotations, translations, FFT
- **Python-C++ Integration** - Boost.Python bindings

**Read this** to understand the key concepts you'll use every day.

---

### 4. 🛠️ [DEVELOPER_GUIDE.md](DEVELOPER_GUIDE.md)
**Time**: 2-4 hours (plus hands-on practice)  
**Practical guide** for building and extending.

**What's inside**:
- Building from source with Anaconda
- CMake build system overview
- **Adding new Processors** (step-by-step)
- **Adding new file formats** (step-by-step)
- Testing framework
- Debugging tips
- Contributing guidelines

**Read this** when you're ready to build and extend EMAN2.

---

### 5. 📖 [API_REFERENCE.md](API_REFERENCE.md)
**Time**: Reference (use as needed)  
**Quick lookup** for APIs and examples.

**What's inside**:
- EMData class methods
- Factory classes (Processors, Aligners, etc.)
- Geometry classes (Vec3f, Transform, Region)
- Utility functions
- Common processors with parameters
- File I/O operations
- CTF handling
- NumPy integration
- Common workflows with code examples

**Use this** as a quick reference while coding.

---

### 6. 📑 [DOCUMENTATION_INDEX.md](DOCUMENTATION_INDEX.md)
**Time**: 15 minutes  
**Complete index** and navigation.

**What's inside**:
- Index of all documentation
- Multiple learning paths (User, Algorithm Developer, Core Developer)
- Quick reference tables
- Key files in codebase
- Success criteria checklist

**Use this** to navigate and find specific information.

---

### 7. 📝 [DOCS_README.md](DOCS_README.md)
**Time**: 10 minutes  
**Quick start** summary.

**What's inside**:
- Quick start instructions
- Documentation overview table
- Key concepts summary
- Quick code examples
- Success checklist

**Use this** for a quick overview or refresher.

---

## 🎓 Recommended Learning Paths

### Path 1: User/Scripter (1-2 days)
**Goal**: Use EMAN2 for image processing tasks

```
Day 1 Morning:   LEARNING_GUIDE.md (overview)
Day 1 Afternoon: API_REFERENCE.md (EMData, common processors)
Day 2 Morning:   Explore examples/ directory
Day 2 Afternoon: Write simple Python scripts
```

**Outcome**: Can write Python scripts to process images

---

### Path 2: Algorithm Developer (3-5 days)
**Goal**: Add new processing algorithms

```
Day 1: LEARNING_GUIDE.md + ARCHITECTURE.md
Day 2: CORE_CONCEPTS.md (focus on Factory pattern)
Day 3: DEVELOPER_GUIDE.md (focus on "Adding New Features")
Day 4: Build EMAN2, add a simple Processor
Day 5: Study existing processors, add complex features
```

**Outcome**: Can add new Processors, Aligners, etc.

---

### Path 3: Core Developer (1-2 weeks)
**Goal**: Contribute to EMAN2 core

```
Week 1:
  - Read all documentation in order
  - Study build system (CMakeLists.txt)
  - Study Python bindings (libpyEM/*.cpp)
  - Build from source, run tests

Week 2:
  - Study core classes (emdata.*, processor.*)
  - Fix bugs, add features
  - Write tests
  - Submit contributions
```

**Outcome**: Can contribute to EMAN2 core development

---

### Path 4: Format Support Developer (2-4 days)
**Goal**: Add support for new file formats

```
Day 1: LEARNING_GUIDE.md + ARCHITECTURE.md (I/O system)
Day 2: CORE_CONCEPTS.md (Image I/O section)
Day 3: DEVELOPER_GUIDE.md ("Adding a New File Format")
Day 4: Study libEM/io/, implement format handler
```

**Outcome**: Can add new file format support

---

## 🚀 Quick Start (30 minutes)

If you're in a hurry, here's the absolute minimum:

1. **Read**: [LEARNING_GUIDE.md](LEARNING_GUIDE.md) (15 min)
2. **Skim**: [CORE_CONCEPTS.md](CORE_CONCEPTS.md) - EMData section (10 min)
3. **Reference**: [API_REFERENCE.md](API_REFERENCE.md) - Quick examples (5 min)

Then start coding and refer back as needed!

---

## 📊 Tutorial Progress Tracker

Track your progress through the tutorial:

- [ ] Read LEARNING_GUIDE.md
- [ ] Read ARCHITECTURE.md
- [ ] Read CORE_CONCEPTS.md
- [ ] Read DEVELOPER_GUIDE.md
- [ ] Built EMAN2 from source
- [ ] Wrote a simple Python script
- [ ] Added a new Processor
- [ ] Ran the test suite
- [ ] Understand EMData class
- [ ] Understand Factory pattern
- [ ] Can navigate the codebase

**Completed all?** Congratulations! You now understand EMAN2! 🎉

---

## 💡 Study Tips

1. **Don't rush** - Take time to understand each concept
2. **Code along** - Try examples as you read
3. **Build it** - Actually building from source helps tremendously
4. **Read code** - Study existing processors and examples
5. **Ask questions** - Use the EMAN2 community resources
6. **Take breaks** - This is a lot of information!

---

## 🔗 Additional Resources

### Official Resources
- **Wiki**: http://blake.bcm.edu/emanwiki
- **Homepage**: http://eman2.org
- **GitHub**: https://github.com/cryoem/eman2

### In the Codebase
- **Examples**: `../examples/` directory
- **Tests**: `../tests/` and `../rt/` directories
- **Programs**: `../programs/` directory (e2*.py)

### Build Documentation
- **Doxygen**: Build with `ENABLE_AUTODOC=ON` for API docs

---

## ✅ Success Criteria

You've mastered EMAN2 when you can:

- [ ] Explain what EMAN2 is and what it's used for
- [ ] Describe the layered architecture
- [ ] Explain what EMData is and how it's used
- [ ] Describe the Factory pattern and why it's used
- [ ] Load, process, and save images using Python
- [ ] Navigate the codebase to find relevant code
- [ ] Build EMAN2 from source
- [ ] Add a new Processor and test it
- [ ] Understand how Python and C++ interact
- [ ] Read and understand existing code

---

## 📝 Tutorial Information

**Created**: 2025  
**EMAN2 Version**: 2.99.70  
**License**: GPL2, BSD 3-Clause  
**Status**: Complete and current

**Tutorial Files**:
- ✅ LEARNING_GUIDE.md (Overview & navigation)
- ✅ ARCHITECTURE.md (System design)
- ✅ CORE_CONCEPTS.md (Key abstractions)
- ✅ DEVELOPER_GUIDE.md (Building & extending)
- ✅ API_REFERENCE.md (API quick reference)
- ✅ DOCUMENTATION_INDEX.md (Complete index)
- ✅ DOCS_README.md (Quick start)

---

## 🎯 Ready to Start?

**Begin your journey**: Open [LEARNING_GUIDE.md](LEARNING_GUIDE.md) and start learning!

**Questions?** Check [DOCUMENTATION_INDEX.md](DOCUMENTATION_INDEX.md) for navigation help.

**Need quick reference?** Jump to [API_REFERENCE.md](API_REFERENCE.md).

---

**Happy Learning!** 🚀

**Commit Message:**
docs: organize learning documentation into tutorial folder

All learning documentation has been consolidated into the tutorial/
folder for better organization and accessibility.
