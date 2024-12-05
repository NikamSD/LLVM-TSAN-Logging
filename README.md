# Building LLVM with ThreadSanitizer (TSan) Support

This guide outlines the steps to build the LLVM project with ThreadSanitizer (TSan) enabled and use it to compile and detect threading issues in your code.

---

## System Requirements

- **RAM:** 32 GB  
- **CPU:** 8 cores  (more the cores, faster the compilation)
- **Architecture:** x86_64  
- **OS:** Ubuntu 22.04  or higher
- **CMake Version:** 3.22.1 or higher  
- **Clang Version:** 14.0.0  or higher

---

## Steps to Build LLVM and Enable TSan

### 1. Clone LLVM Project
Ensure the LLVM source code is available. If not already cloned:
```bash
git clone https://github.com/llvm/llvm-project.git
```

### 2. Configure Build Using CMake

To configure the LLVM project build with ThreadSanitizer (TSan) support, execute the following command:

```bash
CC=clang CXX=clang++ cmake -G "Unix Makefiles" \
    -DCMAKE_BUILD_TYPE=Debug \
    -DLLVM_ENABLE_PROJECTS="clang;compiler-rt" \
    -DLLVM_TARGETS_TO_BUILD="X86" \
    -DCOMPILER_RT_BUILD_SANITIZERS=ON \
    -DCOMPILER_RT_BUILD_TSAN=ON \
    -DCOMPILER_RT_DEBUG=ON \
    ../llvm-project/llvm
```
#### Explanation of Flags:  

* `CC=clang` and `CXX=clang++`: Specifies the Clang compiler for building LLVM.  
* `-G "Unix Makefiles"`: Generates Unix Makefiles for the build system.  
* `-DCMAKE_BUILD_TYPE=Debug`: Sets the build type to Debug for detailed debugging information.  
* `-DLLVM_ENABLE_PROJECTS="clang;compiler-rt"`: Includes the Clang and compiler-rt sub-projects in the build.  
* `-DLLVM_TARGETS_TO_BUILD="X86"`: Specifies the target architecture for the build (x86 in this case).  
* `-DCOMPILER_RT_BUILD_SANITIZERS=ON`: Enables the build of sanitizers in the compiler-rt project.  
* `-DCOMPILER_RT_BUILD_TSAN=ON`: Specifically enables ThreadSanitizer (TSan).  
* `-DCOMPILER_RT_DEBUG=ON`: Enables debug symbols for the runtime libraries.

### 3. Build LLVM Project
Build the clang and compiler-rt sub-projects:

```bash
make -j8 clang compiler-rt
```

### 4. Install TSan Runtime Libraries  

Copy the necessary runtime libraries to the system directory for ThreadSanitizer:  

```bash  
sudo cp <ProjectDir>/build/lib/clang/<available version>/lib/x86_64-unknown-linux-gnu/libclang_rt.tsan-x86_64.a /usr/lib/clang/<available version>/lib/linux/  
sudo cp <ProjectDir>/build/lib/clang/<available version>/lib/x86_64-unknown-linux-gnu/libclang_rt.tsan_cxx-x86_64.a /usr/lib/clang/<available version>/lib/linux/
```
* `libclang_rt.tsan-x86_64.a`: Provides runtime support for ThreadSanitizer in C programs.
* `libclang_rt.tsan_cxx-x86_64.a`: Provides runtime support for ThreadSanitizer in C++ programs.


### 5. Compile and Link with TSan Enabled #### (Using ThreadSanitizer to Detect Threading Issues)
Compile your C++ file (e.g., example.cpp) with ThreadSanitizer enabled:

```bash
clang++ -fsanitize=thread -fPIE -pie -g -O1 -o example example.cpp
```

* `-fsanitize=thread`: Enables ThreadSanitizer.
* `-fPIE`: Generates position-independent code (PIC) for executables.
* `-pie`: Creates a position-independent executable.
* `-g`: Generates debug information for better diagnostics.
* `-O1`: Sets optimization level to 1.

### 6. Run the Executable
Execute the binary to identify threading issues:

```bash
./example
```
ThreadSanitizer will analyze runtime behavior and report data races, synchronization issues, and other threading bugs.

---

## Tsan Logging
