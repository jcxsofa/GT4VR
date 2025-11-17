# Gran Turismo 4 Spec II VR Project - VS Code Development Guide

Yes, absolutely! You can use VS Code for this entire project. Here's how to set up the development environment with VS Code instead of Visual Studio.

---

## VS Code Development Environment Setup

### Required Software

#### Core Development Tools

**1. Visual Studio Code**
- Download: https://code.visualstudio.com/
- Free, lightweight, cross-platform

**2. Visual Studio Build Tools** (Windows - compiler only, NOT full Visual Studio)
- Download: https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2022
- Select "Build Tools for Visual Studio 2022"
- During installation, check:
  - ✅ "Desktop development with C++"
  - ✅ MSVC v143 compiler
  - ✅ Windows 10/11 SDK
  - ✅ CMake tools for Windows
- This gives you the compiler without the full IDE (~6GB vs ~50GB)

**Alternative (Advanced):** Use MinGW-w64 or Clang on Windows
- MinGW-w64: https://www.mingw-w64.org/
- But MSVC is recommended for PCSX2 compatibility

**3. CMake 3.20+**
- Download: https://cmake.org/download/
- Make sure to add to PATH during installation

**4. Git**
- Download: https://git-scm.com/downloads/
- Comes with Git Bash (useful terminal)

**5. Python 3.8+**
- Download: https://www.python.org/downloads/
- Check "Add to PATH" during installation

---

## VS Code Extensions (Required)

Install these extensions in VS Code (Ctrl+Shift+X to open extensions):

### Essential Extensions

**1. C/C++ Extension Pack** (by Microsoft)
- Extension ID: `ms-vscode.cpptools-extension-pack`
- Includes: C/C++, CMake Tools, IntelliSense
- Link: https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools-extension-pack

**2. CMake Tools**
- Extension ID: `ms-vscode.cmake-tools`
- Essential for building PCSX2
- Link: https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools

**3. C/C++** (by Microsoft)
- Extension ID: `ms-vscode.cpptools`
- IntelliSense, debugging, code navigation
- Link: https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools

### Highly Recommended Extensions

**4. CMake Language Support**
- Extension ID: `twxs.cmake`
- Syntax highlighting for CMakeLists.txt
- Link: https://marketplace.visualstudio.com/items?itemName=twxs.cmake

**5. GitLens**
- Extension ID: `eamodio.gitlens`
- Enhanced Git integration
- Link: https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens

**6. Error Lens**
- Extension ID: `usernamehw.errorlens`
- Shows errors inline in editor
- Link: https://marketplace.visualstudio.com/items?itemName=usernamehw.errorlens

**7. Code Spell Checker**
- Extension ID: `streetsidesoftware.code-spell-checker`
- Catches typos in code/comments
- Link: https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker

---

## Project Setup with VS Code

### Step 1: Configure VS Code for C++ Development

Create a `.vscode` folder in your project root with these configuration files:

#### `.vscode/settings.json`
```json
{
  "cmake.configureOnOpen": false,
  "cmake.buildDirectory": "${workspaceFolder}/build",
  "cmake.configureSettings": {
    "CMAKE_BUILD_TYPE": "Release"
  },
  "C_Cpp.default.configurationProvider": "ms-vscode.cmake-tools",
  "files.associations": {
    "*.h": "cpp",
    "*.cpp": "cpp",
    "*.inl": "cpp"
  },
  "editor.formatOnSave": false,
  "cmake.generator": "Ninja",
  "terminal.integrated.env.windows": {
    "PATH": "${env:PATH}"
  }
}
```

#### `.vscode/c_cpp_properties.json`
```json
{
  "configurations": [
    {
      "name": "Win32",
      "includePath": [
        "${workspaceFolder}/**",
        "${workspaceFolder}/pcsx2",
        "${workspaceFolder}/common",
        "C:/VulkanSDK/*/Include",
        "C:/Program Files/OpenVR/headers"
      ],
      "defines": [
        "_DEBUG",
        "UNICODE",
        "_UNICODE"
      ],
      "windowsSdkVersion": "10.0.22621.0",
      "compilerPath": "C:/Program Files (x86)/Microsoft Visual Studio/2022/BuildTools/VC/Tools/MSVC/*/bin/Hostx64/x64/cl.exe",
      "cStandard": "c17",
      "cppStandard": "c++20",
      "intelliSenseMode": "windows-msvc-x64",
      "configurationProvider": "ms-vscode.cmake-tools"
    }
  ],
  "version": 4
}
```

#### `.vscode/launch.json` (for debugging)
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug PCSX2",
      "type": "cppvsdbg",
      "request": "launch",
      "program": "${workspaceFolder}/build/pcsx2-qt/Release/pcsx2-qt.exe",
      "args": [],
      "stopAtEntry": false,
      "cwd": "${workspaceFolder}/build/pcsx2-qt/Release",
      "environment": [],
      "console": "integratedTerminal"
    },
    {
      "name": "Debug Head Tracking Bridge",
      "type": "cppvsdbg",
      "request": "launch",
      "program": "${workspaceFolder}/head-tracking-bridge/build/gt4_vr_bridge.exe",
      "args": [],
      "stopAtEntry": false,
      "cwd": "${workspaceFolder}/head-tracking-bridge/build",
      "environment": [],
      "console": "integratedTerminal"
    }
  ]
}
```

#### `.vscode/tasks.json` (build tasks)
```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "CMake Configure",
      "type": "cmake",
      "command": "configure",
      "problemMatcher": [],
      "detail": "CMake configure"
    },
    {
      "label": "CMake Build",
      "type": "cmake",
      "command": "build",
      "problemMatcher": ["$msCompile"],
      "detail": "CMake build"
    },
    {
      "label": "CMake Clean",
      "type": "cmake",
      "command": "clean",
      "problemMatcher": [],
      "detail": "CMake clean"
    },
    {
      "label": "Build Release",
      "type": "shell",
      "command": "cmake",
      "args": [
        "--build",
        "build",
        "--config",
        "Release",
        "-j",
        "8"
      ],
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "problemMatcher": ["$msCompile"]
    }
  ]
}
```

---

## Building PCSX2 with VS Code

### Method 1: Using CMake Tools Extension (Recommended)

**Step 1: Open PCSX2 folder in VS Code**
```bash
git clone https://github.com/PCSX2/pcsx2.git
cd pcsx2
code .
```

**Step 2: Configure CMake**
- Press `Ctrl+Shift+P` to open Command Palette
- Type "CMake: Configure"
- Select your compiler kit (Visual Studio Build Tools 2022)
- CMake will generate build files

**Step 3: Build**
- Press `Ctrl+Shift+P`
- Type "CMake: Build"
- Or click "Build" in the bottom status bar
- Or press `F7`

**Step 4: Build Target Selection**
- Click the target selector in status bar
- Choose "pcsx2-qt" (the main executable)

**Step 5: Run/Debug**
- Press `F5` to debug
- Or `Ctrl+F5` to run without debugging

### Method 2: Using Terminal Commands

**Open integrated terminal** (`Ctrl+~`)

```bash
# Configure
mkdir build
cd build
cmake .. -G Ninja -DCMAKE_BUILD_TYPE=Release

# Build
cmake --build . --config Release -j 8

# Or use ninja directly
ninja
```

### Method 3: Using Tasks (Keyboard Shortcuts)

- `Ctrl+Shift+B` - Run default build task
- Configure tasks in `.vscode/tasks.json` (shown above)

---

## Building Head Tracking Bridge with VS Code

### Project Structure
```
head-tracking-bridge/
├── .vscode/
│   ├── settings.json
│   ├── c_cpp_properties.json
│   ├── launch.json
│   └── tasks.json
├── src/
│   ├── main.cpp
│   ├── pine_client.cpp
│   ├── pine_client.h
│   ├── vr_reader.cpp
│   ├── vr_reader.h
│   ├── mapper.cpp
│   └── mapper.h
├── include/
├── external/
│   ├── openvr/          # OpenVR SDK
│   └── (other libs)
├── CMakeLists.txt
├── config.ini
└── README.md
```

### CMakeLists.txt for Bridge Project
```cmake
cmake_minimum_required(VERSION 3.20)
project(GT4_VR_Bridge)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Find OpenVR
find_path(OPENVR_INCLUDE_DIR openvr.h
  PATHS
    ${CMAKE_SOURCE_DIR}/external/openvr/headers
    "C:/Program Files/OpenVR/headers"
)

find_library(OPENVR_LIBRARY
  NAMES openvr_api
  PATHS
    ${CMAKE_SOURCE_DIR}/external/openvr/lib/win64
    "C:/Program Files/OpenVR/lib/win64"
)

# Source files
set(SOURCES
  src/main.cpp
  src/pine_client.cpp
  src/vr_reader.cpp
  src/mapper.cpp
)

set(HEADERS
  src/pine_client.h
  src/vr_reader.h
  src/mapper.h
)

# Create executable
add_executable(gt4_vr_bridge ${SOURCES} ${HEADERS})

# Include directories
target_include_directories(gt4_vr_bridge PRIVATE
  ${CMAKE_SOURCE_DIR}/src
  ${CMAKE_SOURCE_DIR}/include
  ${OPENVR_INCLUDE_DIR}
)

# Link libraries
target_link_libraries(gt4_vr_bridge PRIVATE
  ${OPENVR_LIBRARY}
  ws2_32  # Windows sockets for PINE IPC
)

# Copy OpenVR DLL to output directory (Windows)
if(WIN32)
  add_custom_command(TARGET gt4_vr_bridge POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
    "${CMAKE_SOURCE_DIR}/external/openvr/bin/win64/openvr_api.dll"
    $<TARGET_FILE_DIR:gt4_vr_bridge>
  )
endif()

# Copy config.ini to build directory
add_custom_command(TARGET gt4_vr_bridge POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy_if_different
  "${CMAKE_SOURCE_DIR}/config.ini"
  $<TARGET_FILE_DIR:gt4_vr_bridge>
)
```

### Building the Bridge in VS Code

**Same as PCSX2:**
1. Open `head-tracking-bridge` folder in VS Code
2. `Ctrl+Shift+P` → "CMake: Configure"
3. `Ctrl+Shift+P` → "CMake: Build"
4. Or use `Ctrl+Shift+B` for quick build

---

## Debugging in VS Code

### Setting Breakpoints
- Click in the gutter (left of line numbers) to set breakpoints
- Red dots appear where breakpoints are set

### Starting Debug Session
- Press `F5` or click "Run and Debug" in sidebar
- Select configuration from `launch.json`
- Program will pause at breakpoints

### Debug Controls
- `F5` - Continue
- `F10` - Step Over
- `F11` - Step Into
- `Shift+F11` - Step Out
- `Ctrl+Shift+F5` - Restart
- `Shift+F5` - Stop

### Debug Views
- **Variables** - View local/global variables
- **Watch** - Add expressions to watch
- **Call Stack** - See function call hierarchy
- **Breakpoints** - Manage all breakpoints
- **Debug Console** - Execute commands during debugging

---

## Terminal Integration

### Using Multiple Terminals in VS Code

VS Code supports multiple terminal instances:

**Open terminals:**
- `` Ctrl+` `` - Toggle integrated terminal
- `Ctrl+Shift+~` - Create new terminal
- Click `+` in terminal panel

**Useful terminal setups:**
1. **Terminal 1:** PCSX2 build commands
2. **Terminal 2:** Head tracking bridge build
3. **Terminal 3:** Git operations
4. **Terminal 4:** Running PCSX2/testing

### Recommended Shells

**Windows:**
- **Git Bash** (comes with Git for Windows) - Unix-like experience
- **PowerShell** - Native Windows, good CMake support
- **Command Prompt** - Traditional, works fine

**Configure default shell:**
- `Ctrl+Shift+P` → "Terminal: Select Default Profile"
- Choose your preferred shell

---

## Build System Comparison

### Ninja (Recommended for VS Code)

**Advantages:**
- Much faster than MSBuild
- Better terminal output
- Parallel builds by default
- Smaller build files

**Install Ninja:**
```bash
# Via chocolatey (if installed)
choco install ninja

# Or download directly
# https://github.com/ninja-build/ninja/releases
# Add to PATH
```

**Use Ninja with CMake:**
```bash
cmake .. -G Ninja -DCMAKE_BUILD_TYPE=Release
ninja
```

### MSBuild (Default with Visual Studio Build Tools)

**Use if Ninja not available:**
```bash
cmake .. -G "Visual Studio 17 2022" -A x64
cmake --build . --config Release
```

---

## Useful VS Code Keyboard Shortcuts

### Navigation
- `Ctrl+P` - Quick file open
- `Ctrl+Shift+O` - Go to symbol in file
- `Ctrl+T` - Go to symbol in workspace
- `F12` - Go to definition
- `Alt+F12` - Peek definition
- `Shift+F12` - Find all references
- `Ctrl+G` - Go to line
- `Alt+Left/Right` - Navigate back/forward

### Editing
- `Ctrl+/` - Toggle comment
- `Ctrl+Shift+K` - Delete line
- `Alt+Up/Down` - Move line up/down
- `Shift+Alt+Up/Down` - Copy line up/down
- `Ctrl+D` - Add selection to next find match
- `Ctrl+Shift+L` - Select all occurrences of current selection

### Build & Debug
- `Ctrl+Shift+B` - Run build task
- `F5` - Start debugging
- `Ctrl+F5` - Run without debugging
- `F9` - Toggle breakpoint
- `F10` - Step over
- `F11` - Step into

### Terminal
- `` Ctrl+` `` - Toggle terminal
- `Ctrl+Shift+~` - New terminal
- `Ctrl+C` - Kill terminal process

### Other
- `Ctrl+Shift+P` - Command palette
- `Ctrl+Shift+X` - Extensions
- `Ctrl+Shift+E` - Explorer
- `Ctrl+Shift+F` - Search
- `Ctrl+Shift+G` - Source control (Git)

---

## Workflow Comparison: VS Code vs Visual Studio

### What You Get with VS Code

✅ **Advantages:**
- Lightweight and fast (starts in seconds)
- Highly customizable
- Great terminal integration
- Works on Windows/Linux/Mac
- Free and open source
- Active extension ecosystem
- Better for multi-project workflows
- Excellent Git integration

❌ **What You Lose (vs full Visual Studio):**
- No visual debugger designer
- No GUI for project properties (use CMake instead)
- No built-in profiler (use external tools)
- Slightly more manual setup
- Less hand-holding for beginners

### Recommended: Use VS Code!

For this project, VS Code is actually **better** than Visual Studio because:
1. ✅ PCSX2 uses CMake anyway (no .sln benefits)
2. ✅ Faster iteration (build → test cycles)
3. ✅ Better for editing multiple projects simultaneously
4. ✅ Excellent terminal integration for PINE testing
5. ✅ More portable setup (works on Linux if you expand later)

---

## Complete Setup Checklist

### ☐ Install Base Software
- [ ] VS Code installed
- [ ] Visual Studio Build Tools 2022 installed (C++ workload)
- [ ] CMake installed and in PATH
- [ ] Git installed
- [ ] Python 3.8+ installed
- [ ] Ninja installed (optional but recommended)

### ☐ Install VS Code Extensions
- [ ] C/C++ Extension Pack
- [ ] CMake Tools
- [ ] GitLens
- [ ] Error Lens (optional)

### ☐ Install VR/Graphics SDKs
- [ ] Vulkan SDK installed
- [ ] OpenVR SDK downloaded
- [ ] Graphics drivers updated

### ☐ Setup PCSX2
- [ ] PCSX2 downloaded and working
- [ ] GT4 Spec II obtained and running
- [ ] PINE IPC enabled in PCSX2 settings
- [ ] Vulkan renderer working with GT4

### ☐ Clone Repositories
- [ ] PCSX2 cloned: `git clone https://github.com/PCSX2/pcsx2.git`
- [ ] ps2_cam_acolyte cloned (reference): `git clone https://github.com/moonlessformless/ps2_cam_acolyte.git`
- [ ] Create your project directories

### ☐ Test Build Environment
- [ ] PCSX2 builds successfully in VS Code
- [ ] Can run custom PCSX2 build
- [ ] Hello World C++ project compiles
- [ ] Debugger attaches successfully

### ☐ VR Hardware
- [ ] VR headset connected
- [ ] SteamVR or OpenXR runtime installed
- [ ] Headset tracking working
- [ ] Can run simple VR demos

---

## Quick Start Commands for VS Code

### Clone and Open PCSX2
```bash
# Clone
git clone https://github.com/PCSX2/pcsx2.git
cd pcsx2

# Create your VR branch
git checkout -b gt4-vr-stereo

# Open in VS Code
code .

# In VS Code:
# Ctrl+Shift+P → "CMake: Configure"
# Ctrl+Shift+P → "CMake: Build"
```

### Create Bridge Project
```bash
# Create project structure
mkdir gt4-vr-bridge
cd gt4-vr-bridge
mkdir src include external build

# Create basic files
touch CMakeLists.txt
touch src/main.cpp
touch config.ini

# Open in VS Code
code .
```

### Build Commands (in VS Code terminal)
```bash
# Configure
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release

# Build
cmake --build build --config Release

# Or use ninja directly
cd build
ninja

# Run
./build/gt4_vr_bridge.exe
```

---

## Troubleshooting VS Code Setup

### Issue: CMake Can't Find Compiler

**Solution:**
```bash
# Open VS Code terminal
# Set environment for MSVC
"C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Auxiliary\Build\vcvars64.bat"

# Then configure
cmake -B build -G Ninja
```

Or add to `.vscode/settings.json`:
```json
{
  "cmake.configureEnvironment": {
    "CC": "cl.exe",
    "CXX": "cl.exe"
  }
}
```

### Issue: IntelliSense Not Working

**Solution:**
1. Press `Ctrl+Shift+P`
2. Type "C/C++: Edit Configurations (UI)"
3. Set compiler path manually
4. Set C++ standard to C++20
5. Reload window (`Ctrl+Shift+P` → "Reload Window")

### Issue: Can't Find Vulkan/OpenVR Headers

**Solution:**
Update `.vscode/c_cpp_properties.json` with correct paths:
```json
{
  "includePath": [
    "${workspaceFolder}/**",
    "C:/VulkanSDK/1.3.xxx/Include",
    "C:/path/to/openvr/headers"
  ]
}
```

### Issue: Build Takes Forever

**Solutions:**
1. Use Ninja instead of MSBuild
2. Increase parallel jobs: `ninja -j 16`
3. Use `Release` not `Debug` for faster builds
4. Close other applications
5. Add to `.vscode/settings.json`:
```json
{
  "cmake.parallelJobs": 16
}
```

### Issue: Can't Debug - Symbols Not Loading

**Solution:**
Build in Debug mode:
```bash
cmake -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build
```

Update `launch.json`:
```json
{
  "program": "${workspaceFolder}/build/Debug/pcsx2-qt.exe"
}
```

---

## Recommended VS Code Themes & Fonts for C++

### Themes (Optional, Personal Preference)
- **Dark+** (default) - Good contrast
- **One Dark Pro** - Popular, easy on eyes
- **Dracula Official** - High contrast
- **GitHub Dark** - Clean, modern
- **Monokai Pro** - Classic, vibrant

### Fonts for Coding (Optional)
- **Cascadia Code** (Microsoft) - Has ligatures
- **Fira Code** - Popular ligatures
- **JetBrains Mono** - Clean, readable
- **Source Code Pro** - Adobe's monospace

Enable ligatures in `settings.json`:
```json
{
  "editor.fontFamily": "Cascadia Code, Consolas, 'Courier New', monospace",
  "editor.fontLigatures": true,
  "editor.fontSize": 14
}
```

---

## Final Notes on VS Code Setup

**VS Code is perfect for this project!** You get:
- Professional C++ development environment
- Fast, lightweight IDE
- Great CMake integration
- Excellent debugging support
- All the tools needed for PCSX2 development

**You DON'T need Visual Studio** - The Build Tools give you the compiler, and VS Code gives you the IDE features.

**Recommended workflow:**
1. Use CMake Tools extension for configuration
2. Use integrated terminal for builds
3. Use built-in debugger for testing
4. Use GitLens for version control
5. Keep multiple terminals open for parallel work

**This setup works great for:**
- ✅ Building and modifying PCSX2
- ✅ Creating the head tracking bridge
- ✅ Debugging both applications
- ✅ Managing Git branches
- ✅ Testing and iteration

You're all set to develop the entire GT4 VR project in VS Code! 🚀

---

## Additional Resources

- **VS Code C++ Documentation:** https://code.visualstudio.com/docs/languages/cpp
- **CMake Tools Extension Guide:** https://github.com/microsoft/vscode-cmake-tools/blob/main/docs/README.md
- **VS Code Tips & Tricks:** https://code.visualstudio.com/docs/getstarted/tips-and-tricks
- **C++ on Windows Guide:** https://code.visualstudio.com/docs/cpp/config-msvc

---

**Ready to start coding!** Open VS Code and begin with Phase 1. Good luck! 🎮