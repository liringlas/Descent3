# Descent 3 — Codebase Research Report

## What is Descent 3?

**Descent 3** is a 1999 six-degrees-of-freedom (6DOF) space shooter developed by [Parallax Software](https://en.wikipedia.org/wiki/Parallax_Software) (later renamed Outrage Entertainment) and published by Interplay. It was the third game in the _Descent_ series. The player pilots a spacecraft through mine shafts and alien environments, fighting robots with full rotational freedom — no "up" or "down".

The source code was released as **open source under GPL-3.0** by Parallax Software on **April 16, 2024**. The community group [DescentDevelopers](https://github.com/DescentDevelopers/Descent3) began immediately modernizing it. Version 1.5.0 (August 2024) is the first community release; version 1.6.0 is currently in development.

> **Important:** The repo is an **engine only** — game assets (textures, sounds, levels, videos) must be purchased separately from GOG or Steam.

---

## Language & Standards

| Attribute              | Detail                                                |
| ---------------------- | ----------------------------------------------------- |
| **Primary language**   | **C++17** (strictly enforced via CMake)               |
| **Secondary language** | C (a few third-party files: `libacm`, `libmve`)       |
| **Compiler support**   | MSVC 2022, GCC, Clang / Apple Clang                   |
| **Code style**         | `clang-format` enforced                               |
| **Type system**        | `stdint.h` types (`uint32_t`, etc.) throughout        |
| **Standard library**   | `std::filesystem`, `std::chrono`, `std::string`, etc. |

The codebase has been modernized from its original 1999 C++ dialect. Old platform-specific assembly (`x86` SSE/Katmai intrinsics) has been removed. Old 32-bit assumptions have been cleaned up. It now targets **64-bit only**.

---

## Codebase Size

| Metric                              | Value                         |
| ----------------------------------- | ----------------------------- |
| Source files (`.cpp` / `.h` / `.c`) | **~1,096**                    |
| Total lines of code                 | **~900,000**                  |
| Main game module alone              | ~131 `.cpp` + ~141 `.h` files |

---

## Build System

- **CMake 3.20+** with **Ninja** (default)
- **vcpkg** for third-party dependency management
- Presets for `win`, `mac`, `linux`
- CI via **GitHub Actions** (Linux GCC/Clang, Windows MSVC, macOS Apple-Clang on x64 & arm64)
- Cross-compilation supported (e.g., Linux ARM64)

---

## Platform Support

| Platform                             | Status             |
| ------------------------------------ | ------------------ |
| Windows (x64)                        | ✅ Fully supported |
| Linux (x64, ARM64)                   | ✅ Fully supported |
| macOS (x64 + ARM64 universal binary) | ✅ Fully supported |
| 32-bit                               | ❌ Dropped         |

---

## Major Subsystems / Modules

These are the independent directories, each compiled as a CMake sub-target:

| Module           | Description                                                                                                                   |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `Descent3/`      | **Core game engine** — game loop, AI, physics integration, player, weapons, missions, cinematics, HUD, menus, networking glue |
| `renderer/`      | OpenGL renderer; renders to an FBO at game resolution; no legacy software renderer                                            |
| `physics/`       | Rigid-body physics, collision, movement                                                                                       |
| `sndlib/`        | Sound system (SDL3 audio backend)                                                                                             |
| `stream_audio/`  | Streaming audio playback                                                                                                      |
| `music/`         | Music subsystem                                                                                                               |
| `libmve/`        | MVE video decoder (intro/cutscene movies)                                                                                     |
| `2dlib/`         | 2D drawing primitives                                                                                                         |
| `bitmap/`        | Bitmap/texture loading and management                                                                                         |
| `grtext/`        | Text rendering                                                                                                                |
| `ui/`            | In-game UI widgets                                                                                                            |
| `model/`         | 3D model loading (`.POF` Parallax Object Format)                                                                              |
| `ddio/`          | **Device/Display I/O** — keyboard, mouse, joystick, filesystem; wraps SDL3                                                    |
| `ddebug/`        | Debug utilities                                                                                                               |
| `networking/`    | Low-level UDP/TCP networking                                                                                                  |
| `netcon/`        | Network connection management                                                                                                 |
| `netgames/`      | Multiplayer game modes (CTF, Anarchy, Hoard, etc.)                                                                            |
| `scripts/`       | Level scripts (compiled as shared libs / HOG files per level)                                                                 |
| `manage/`        | Asset management (HOG archive files)                                                                                          |
| `cfile/`         | Cross-platform file I/O (`std::filesystem` based)                                                                             |
| `vecmat/`        | Vector/matrix math library                                                                                                    |
| `fix/`           | Fixed-point math (legacy)                                                                                                     |
| `misc/`          | Miscellaneous utilities (string helpers, etc.)                                                                                |
| `mem/`           | Memory management wrappers                                                                                                    |
| `module/`        | Dynamic library loading (for scripts)                                                                                         |
| `md5/`           | MD5 checksum                                                                                                                  |
| `unzip/`         | ZIP/HOG archive reading                                                                                                       |
| `AudioEncode/`   | Audio encoding utilities                                                                                                      |
| `rtperformance/` | Runtime performance counters                                                                                                  |
| `logger/`        | Logging (wraps `plog`)                                                                                                        |
| `overlays/`      | Screen overlay effects                                                                                                        |
| `tools/`         | Build utilities (e.g., `HogMaker`)                                                                                            |
| `editor/`        | Internal level editor (Windows-only MFC app, unstable, not shipped)                                                           |
| `linux/`         | Linux-specific platform shims                                                                                                 |
| `lib/`           | Shared header-only definitions and interfaces                                                                                 |

---

## Third-Party Dependencies

| Library                        | Purpose                                                     |
| ------------------------------ | ----------------------------------------------------------- |
| **SDL3**                       | Window, input, audio backend (primary platform abstraction) |
| **OpenGL**                     | Rendering API                                               |
| **glm**                        | OpenGL math (vectors, matrices)                             |
| **zlib**                       | Compression                                                 |
| **cpp-httplib**                | HTTP client (for downloading missions/levels)               |
| **plog**                       | Logging                                                     |
| **gtest**                      | Unit testing (optional)                                     |
| **libacm** (vendored)          | InterPlay ACM audio decoding                                |
| **libmve** (vendored)          | Interplay MVE video decoding                                |
| **stb_image_write** (vendored) | PNG screenshot saving                                       |

---

## Architecture Highlights

- **HOG files** — game assets are packed in proprietary HOG archive files. Level scripts are compiled per-platform into HOG files at build time.
- **OSIRIS scripting** — levels load C++ compiled shared libraries dynamically for scripted behavior (`module/` + `OsirisLoadandBind.cpp`). There is no interpreted scripting language — scripts are compiled C++.
- **6DOF engine** — full 6DOF movement with a BSP/room-based indoor renderer and a terrain engine for outdoor areas.
- **Multiplayer** — TCP/IP-based multiplayer, dedicated server mode, GameSpy integration (legacy).
- **AI** — autonomous robot AI with goals, pathfinding, awareness (`AIMain.cpp`, `AIGoal.cpp`, `aipath.cpp`).
- **Original code dates** — many files have SourceSafe log comments dating to 1997–1999 (the original development era).

---

## What a Language Port Would Involve

> **Warning:** This is a very large codebase. A full port is an enormous undertaking.

Key challenges:

1. **~900k lines of C++17** — game logic, math, rendering, networking, AI, physics, audio are all deeply entangled.
2. **C++ idioms everywhere** — raw pointers, manual memory management, `reinterpret_cast`, union types, fixed-point math, bitwise tricks.
3. **Platform ABI assumptions** — struct layouts affect save files, demo files, and network packets (endian-sensitivity).
4. **Dynamic linking (OSIRIS)** — level scripts are compiled C++ shared libraries loaded at runtime. A port would need an equivalent plugin/scripting mechanism.
5. **OpenGL calls** — the renderer is OpenGL-specific; you'd likely map these to a graphics API of the target language (e.g., `wgpu`/Vulkan bindings in Rust, or a C# wrapper).
6. **SDL3** — SDL3 has bindings for many languages (Rust `sdl3`, Python `pysdl3`, etc.), which helps.
7. **No garbage collection** — everything is manually managed; GC-based languages need extra care.

**Best candidate target languages** (opinion):

- **Rust** — zero-cost abstractions, no GC, excellent SDL3 and OpenGL bindings, but requires rewriting all unsafe pointer patterns.
- **Zig** — more C-like, easier C interop, but smaller ecosystem.
- **C#** (.NET 9) — easiest FFI with existing C++ (via P/Invoke), but GC pauses could affect the game loop.
- **Go** — simpler but GC and lack of unions/bitfields make it harder for low-level game engine code.

A realistic **incremental porting strategy** would be to use FFI to interop the target language with existing C++ modules, porting subsystem by subsystem (e.g., start with `vecmat`, `misc`, `cfile`, then `renderer`, then core game logic).
