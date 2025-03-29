---
layout: post
title: "How to Statically Compile a Python Program into a Binary Executable File"
date: 2019-02-07   
tag: Python 
---

### How to Statically Compile a Python Program into a Binary Executable File

Creating a statically compiled binary for your Python program ensures it can run independently of the target system's libraries. Here's a step-by-step guide:

---

## **1. Install Necessary Tools**
Ensure the following tools are installed:

### **PyInstaller** - Create standalone executables
PyInstaller bundles the Python interpreter and all required libraries into a single, self-contained executable.
```bash
pip install pyinstaller
```

### **Staticx** - Make binaries fully static
Staticx wraps the PyInstaller-generated executable with a minimal runtime, bundling shared libraries into the binary.
```bash
pip install staticx
```

### **patchelf** - Modify ELF binaries (Linux only)
Patchelf is used to manipulate ELF (Executable and Linkable Format) binaries, which is useful for making dynamically linked executables more portable.

- Install on Ubuntu/Debian:
```bash
sudo apt install patchelf
```

---

## **2. Create an Executable with PyInstaller**
1. Generate a single-file executable:
   ```bash
   pyinstaller --onefile Lio2CSV.py
   ```
2. If your script depends on shared libraries (like `libpython`), locate and include them:
   - Find the `libpython` file:
     ```bash
     find /usr -name "libpython*.so*"
     ```
   - Add it to the binary:
     ```bash
     pyinstaller --onefile --add-binary "/path/to/libpython.so:." Lio2CSV.py
     ```

---

## **3. Make the Executable Fully Static**
1. Use `Staticx` to convert the PyInstaller output into a static binary:
   ```bash
   staticx dist/Lio2CSV dist/Lio2CSV-static
   ```
2. Verify that the binary is fully static:
   ```bash
   ldd dist/Lio2CSV-static
   ```
   - If successful, the output should be: `not a dynamic executable`.

---

## **4. Debug Missing Dependencies**
If the static binary fails to run:
- Inspect dependencies of the intermediate binary:
  ```bash
  ldd dist/Lio2CSV
  ```
- Use `strace` to debug runtime issues:
  ```bash
  strace ./Lio2CSV-static
  ```

---

## **Alternative/Optional Steps**

### **1. Alternative: Use a Musl-Based Python (Linux Only)**
For full static linking on Linux, you can use a Musl-based Python:
1. Install Musl libc and Python:
   ```bash
   apk add python3 py3-pip musl-dev
   ```
2. Compile your Python program in this environment using PyInstaller.

### **2. Optional: Containerize for Maximum Portability**
If static compilation is too complex, package the program in a Docker container:
1. Create a `Dockerfile`:
   ```dockerfile
   FROM python:3.10-slim
   COPY Lio2CSV.py .
   RUN pip install pyinstaller && pyinstaller --onefile Lio2CSV.py
   CMD ["dist/Lio2CSV"]
   ```
2. Build and run the container:
   ```bash
   docker build -t lio2csv .
   docker run --rm lio2csv
   ```

---

## **Summary**
1. Use **PyInstaller** to create a single-file executable (`--onefile`).
2. Use **Staticx** to make it fully static.
3. Debug shared library issues with `ldd` or `strace` if necessary.
4. Alternatively, use **Musl libc** for static linking or **Docker** for easy portability.

This process ensures you get a robust, standalone executable that runs on most systems without dependency issues.

