# Blender Build Troubleshooting Experience

This document records the solutions to common build issues encountered while setting up the Blender NPR port in a GitHub Actions environment using GCC 14 on Rocky Linux 9.

## 1. X11 and OpenGL Dependency Issues
**Problem:** CMake fails with `Could NOT find X11 (missing: X11_X11_LIB)`.
**Cause:** Missing development headers and libraries for X11 components and OpenGL.
**Solution:** Install expanded dependencies via `dnf`:
```bash
dnf -y install \
  gcc-toolset-14 make cmake ninja-build git git-lfs python3 rsync patch subversion curl tar xz \
  libX11-devel libXxf86vm-devel libXcursor-devel libXi-devel libXrandr-devel libXinerama-devel \
  mesa-libEGL-devel mesa-libGL-devel mesa-libGLU-devel \
  libXext-devel libXrender-devel libXfixes-devel \
  wayland-devel wayland-protocols-devel libxkbcommon-devel dbus-devel
```

## 2. `stdlib.h` Not Found (oneAPI Conflict)
**Problem:** `fatal error: 'stdlib.h' file not found` during oneAPI/DPC++ compilation.
**Cause:** Conflicts between the Intel DPC++ compiler and custom include environment variables, or missing standard library hooks in the DPC++ toolchain provided by Blender's pre-compiled libraries.
**Solution:** Disable oneAPI if not strictly required for the target build (e.g., NPR port focuses on stylization, not Intel GPU acceleration):
```bash
make release ninja BUILD_CMAKE_ARGS="-DWITH_CYCLES_DEVICE_ONEAPI=OFF -DWITH_CYCLES_ONEAPI=OFF"
```

## Summary of Environment Best Practices
- **Compiler:** Use `gcc-toolset-14` on Rocky Linux 9.
- **Paths:** Keep the build environment minimal unless a specific dependency fails to resolve.
- **Environment:** Avoid unnecessary global header variables like `CPATH`.
