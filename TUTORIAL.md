# EMAN2 Learning Tutorial

## 📚 Complete Learning Documentation

A comprehensive tutorial for understanding and working with the EMAN2 codebase is available in the **`tutorial/`** directory.

### 🚀 Get Started

**New to EMAN2?** Start here:

👉 **[tutorial/README.md](tutorial/README.md)** - Tutorial overview and learning paths

👉 **[tutorial/LEARNING_GUIDE.md](tutorial/LEARNING_GUIDE.md)** - Begin your learning journey

---

## 📖 What's in the Tutorial?

The tutorial includes 8 comprehensive documents covering:

1. **[LEARNING_GUIDE.md](tutorial/LEARNING_GUIDE.md)** - Overview and navigation (START HERE)
2. **[ARCHITECTURE.md](tutorial/ARCHITECTURE.md)** - System architecture and design
3. **[CORE_CONCEPTS.md](tutorial/CORE_CONCEPTS.md)** - Key abstractions (EMData, Factory pattern)
4. **[DEVELOPER_GUIDE.md](tutorial/DEVELOPER_GUIDE.md)** - Building, testing, extending
5. **[API_REFERENCE.md](tutorial/API_REFERENCE.md)** - Quick API reference
6. **[FOURIER_TRANSFORMS.md](tutorial/FOURIER_TRANSFORMS.md)** - Complete FFT guide
7. **[DOCUMENTATION_INDEX.md](tutorial/DOCUMENTATION_INDEX.md)** - Complete index
8. **[DOCS_README.md](tutorial/DOCS_README.md)** - Quick start summary

---

## 🎯 Quick Learning Paths

### For Users (1-2 days)
Want to **use EMAN2** for image processing?
```
→ tutorial/LEARNING_GUIDE.md
→ tutorial/API_REFERENCE.md
→ Start coding!
```

### For Algorithm Developers (3-5 days)
Want to **add new algorithms**?
```
→ tutorial/LEARNING_GUIDE.md
→ tutorial/ARCHITECTURE.md
→ tutorial/CORE_CONCEPTS.md (Factory pattern)
→ tutorial/DEVELOPER_GUIDE.md (Adding New Features)
→ Build and extend!
```

### For Core Contributors (1-2 weeks)
Want to **contribute to EMAN2 core**?
```
→ Read all tutorial docs in order
→ Build from source
→ Study the codebase
→ Contribute!
```

---

## 💡 What You'll Learn

By completing this tutorial, you will:

- ✅ Understand EMAN2 architecture and design
- ✅ Master core concepts (EMData, Factory pattern, etc.)
- ✅ Learn to build and extend EMAN2
- ✅ Be able to add new algorithms and features
- ✅ Write effective Python scripts using EMAN2
- ✅ Navigate the codebase confidently

---

## 🚀 Quick Start (30 minutes)

In a hurry? Here's the express path:

1. **Read**: [tutorial/LEARNING_GUIDE.md](tutorial/LEARNING_GUIDE.md) (15 min)
2. **Skim**: [tutorial/CORE_CONCEPTS.md](tutorial/CORE_CONCEPTS.md) - EMData section (10 min)
3. **Reference**: [tutorial/API_REFERENCE.md](tutorial/API_REFERENCE.md) - Examples (5 min)

Then start coding and refer back as needed!

---

## 📊 What is EMAN2?

EMAN2 is an open-source software suite for **cryo-electron microscopy** (CryoEM) image processing:

- **Single particle analysis** - Determine 3D structures from 2D images
- **Electron tomography** - 3D reconstruction from tilt series
- **Image processing** - Filtering, alignment, classification
- **3D reconstruction** - Multiple algorithms for volume reconstruction

**Architecture**:
```
Applications (e2*.py programs)
        ↓
Python API (EMAN2.py, EMAN3.py)
        ↓
C++ Core Library (libEM)
        ↓
Dependencies (HDF5, FFTW3, GSL, etc.)
```

---

## 🔗 Additional Resources

- **Tutorial**: [tutorial/](tutorial/) directory
- **Official Wiki**: http://blake.bcm.edu/emanwiki
- **Homepage**: http://eman2.org
- **GitHub**: https://github.com/cryoem/eman2
- **Examples**: [examples/](examples/) directory

---

## ✅ Success Checklist

You understand EMAN2 when you can:

- [ ] Explain what EMAN2 is and what it's used for
- [ ] Describe the layered architecture
- [ ] Explain the EMData class
- [ ] Describe the Factory pattern
- [ ] Load, process, and save images using Python
- [ ] Navigate the codebase
- [ ] Build EMAN2 from source
- [ ] Add a new Processor
- [ ] Write tests

---

## 📝 Tutorial Information

**Created**: 2025  
**EMAN2 Version**: 2.99.70  
**License**: GPL2, BSD 3-Clause  
**Status**: Complete and current

---

## 🎓 Ready to Learn?

**Start here**: [tutorial/README.md](tutorial/README.md)

**Begin learning**: [tutorial/LEARNING_GUIDE.md](tutorial/LEARNING_GUIDE.md)

**Happy Learning!** 🚀

---

**Commit Message:**
docs: organize learning documentation into tutorial folder

All learning documentation has been consolidated into the tutorial/
folder with a comprehensive README and cross-referenced guides for
learning the EMAN2 codebase.

