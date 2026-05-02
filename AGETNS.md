# Blender Build Troubleshooting Experience

This document records the solutions to common build issues encountered while setting up the Blender NPR port in a GitHub Actions environment using GCC 14.

## 1. X11 and OpenGL Dependency Issues
**Problem:** CMake fails with `Could NOT find X11 (missing: X11_X11_LIB)`.
**Cause:** Missing development headers and libraries for X11 components and OpenGL.
**Solution:** Install expanded dependencies via `apt-get`:
- `libx11-dev`, `libxxf86vm-dev`, `libxcursor-dev`, `libxi-dev`, `libxrandr-dev`, `libxinerama-dev`, `libegl1-mesa-dev`, `libglvnd-dev`
- `libxext-dev`, `libxrender-dev`, `libxfixes-dev`, `libgl1-mesa-dev`, `libglu1-mesa-dev`

## 2. Conda Compiler Isolation
**Problem:** Micromamba/Conda GCC cannot find system-installed libraries.
**Solution:** Help CMake find system paths without forcing them globally on the compiler:
```yaml
echo "CMAKE_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:/usr/lib" >> $GITHUB_ENV
echo "CMAKE_INCLUDE_PATH=/usr/include/x86_64-linux-gnu:/usr/include" >> $GITHUB_ENV
echo "C_INCLUDE_PATH=/usr/include/x86_64-linux-gnu:/usr/include" >> $GITHUB_ENV
echo "CPLUS_INCLUDE_PATH=/usr/include/x86_64-linux-gnu:/usr/include" >> $GITHUB_ENV
```

## 3. `_Float128` Redeclaration Error
**Problem:** `error: redeclaration of C++ built-in type '_Float128'`.
**Cause:** Using `CPATH` environment variable. `CPATH` forces system headers (`/usr/include`) to be searched *before* the compiler's internal headers, causing a conflict between the system's `glibc` and the new GCC 14 built-ins.
**Solution:** **NEVER** use `CPATH`. Use `CMAKE_INCLUDE_PATH` for CMake discovery or `C_INCLUDE_PATH`/`CPLUS_INCLUDE_PATH` if absolutely necessary (as they are searched *after* internal headers).

## 4. `stdlib.h` Not Found (oneAPI Conflict)
**Problem:** `fatal error: 'stdlib.h' file not found` during oneAPI/DPC++ compilation.
**Cause:** Conflicts between the Intel DPC++ compiler and custom include environment variables, or missing standard library hooks in the DPC++ toolchain provided by Blender's pre-compiled libraries.
**Solution:** Disable oneAPI if not strictly required for the target build (e.g., NPR port focuses on stylization, not Intel GPU acceleration):
```bash
make release ninja BUILD_CMAKE_ARGS="-DWITH_CYCLES_DEVICE_ONEAPI=OFF -DWITH_CYCLES_ONEAPI=OFF"
```

## Summary of Environment Best Practices
- **Compiler:** Use Micromamba for GCC 14 if PPA is unavailable.
- **Paths:** Use `CMAKE_INCLUDE_PATH` and `CMAKE_LIBRARY_PATH` for dependency discovery.
- **Environment:** Avoid global header variables like `CPATH` to prevent standard library and built-in type conflicts.
