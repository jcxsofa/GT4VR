# Gran Turismo 4 Spec II VR Project - Complete Development Guide

## Project Overview

**Goal:** Create a fully immersive VR experience for Gran Turismo 4 Spec II running in PCSX2, featuring head tracking, stereoscopic 3D rendering, and dynamic foveated rendering for optimal performance.

**Approach:** Fork PCSX2 to modify the Vulkan renderer for VR support, combined with an external head-tracking bridge using PINE IPC.

**Target Platform:** Windows (primary), with potential Linux support
**Target Game:** Gran Turismo 4 Spec II (CRC: `4CE521F2`)
**Target Renderer:** Vulkan (best performance for GT4)

---

## Development Environment Setup

### Required Software

#### Core Development Tools
- **Visual Studio 2022 Community Edition** (Windows)
  - Download: https://visualstudio.microsoft.com/downloads/
  - Required workloads: "Desktop development with C++"
  - Optional: Linux development with C++ (for cross-platform)

- **CMake 3.20+**
  - Download: https://cmake.org/download/
  - Used for PCSX2 build configuration

- **Git**
  - Download: https://git-scm.com/downloads
  - For version control and cloning repositories

- **Python 3.8+** (for build scripts)
  - Download: https://www.python.org/downloads/

#### VR SDKs (Choose One or Both)

- **OpenVR SDK** (Recommended - works with most headsets)
  - Repository: https://github.com/ValveSoftware/openvr
  - Supports: HTC Vive, Valve Index, Windows Mixed Reality, Quest (via Link/Air Link)
  
- **OpenXR SDK** (Alternative - newer standard)
  - Repository: https://github.com/KhronosGroup/OpenXR-SDK
  - More universal, required for eye-tracked foveated rendering
  - Documentation: https://www.khronos.org/openxr/

#### Emulator & Game

- **PCSX2** (latest stable or nightly)
  - Download: https://pcsx2.net/downloads/
  - Source: https://github.com/PCSX2/pcsx2
  - Enable PINE IPC in settings: Configuration → Network & HDD → Enable Pine

- **Gran Turismo 4 Spec II**
  - Information: https://www.gtplanet.net/forum/threads/gran-turismo-4-spec-ii.405537/
  - This is a community mod released September 2024
  - CRC Hash: `4CE521F2` (important for patches)
  - Requires base GT4 ISO + xDelta patch application

#### Graphics APIs & Extensions

- **Vulkan SDK**
  - Download: https://vulkan.lunarg.com/sdk/home
  - Required for PCSX2 Vulkan renderer modifications
  - Includes validation layers and debugging tools

- **NVIDIA Graphics Drivers** (if using NVIDIA GPU)
  - Latest drivers include VRSS (Variable Rate Supersampling) support
  - VRSS provides automatic foveated rendering without code changes

---

## Essential Technical Resources

### PCSX2 Documentation & Code

- **Main PCSX2 Repository**
  - URL: https://github.com/PCSX2/pcsx2
  - Key directories:
    - `pcsx2/GS/` - Graphics Synthesizer emulation
    - `pcsx2/GS/Renderers/Vulkan/` - Vulkan renderer (your main target)
    - `pcsx2/GS/Renderers/HW/` - Hardware renderer base classes
    - `pcsx2/IPC.h` - PINE IPC protocol definitions

- **PINE IPC Documentation**
  - File: `pcsx2/IPC_Readme.md` in PCSX2 repo
  - Protocol: Socket-based IPC for memory read/write
  - Key commands: `MsgRead32`, `MsgWrite32`, `MsgRead64`, `MsgWrite64`

### GT4 Modding Resources

- **Gran Turismo Modding Hub** (PRIMARY RESOURCE)
  - URL: https://nenkai.github.io/gt-modding-hub/
  - Created by Nenkai, contains all GT4 modding documentation

- **GT4 Camera Patches**
  - URL: https://nenkai.github.io/gt-modding-hub/ps2/gt4/patches/
  - Shows exact memory addresses for camera control
  - CRITICAL: Contains working camera manipulation code
  - Key addresses (GT4 Online, similar for Spec II):
    - Gamepad input: `0x89B0DE`
    - Camera computation: `0x2063D8` (computeViewAngle function)

- **GT4 Modding Tools**
  - URL: https://nenkai.github.io/gt-modding-hub/ps2/gt4/tools/
  - Includes:
    - GT4FS: VOL file extractor/repacker
    - GT4Hooks: Code injection tools
    - GT4 Save Editor
    - Database editors

### Reference Implementation

- **ps2_cam_acolyte** (WORKING EXAMPLE OF PINE IPC USAGE)
  - URL: https://github.com/moonlessformless/ps2_cam_acolyte
  - This is a FREE CAMERA tool for PS2 horror games using PINE
  - Shows exactly how to:
    - Connect to PCSX2 via PINE socket
    - Read/write PS2 memory addresses
    - Implement real-time camera control
    - Handle controller input
  - Written in C++, perfect reference for Phase 1

### Stereoscopic 3D & VR Resources

- **Vulkan Multiview Extension** (VR rendering)
  - Spec: https://registry.khronos.org/vulkan/specs/1.3-extensions/man/html/VK_KHR_multiview.html
  - Allows rendering both eyes in single pass (efficient)

- **Variable Rate Shading (Foveated Rendering)**
  - NVIDIA VRS Overview: https://developer.nvidia.com/vrworks/graphics/variablerateshading
  - Vulkan VRS Extensions:
    - `VK_KHR_fragment_shading_rate`
    - `VK_QCOM_fragment_density_map_offset` (for eye-tracked foveation)

- **OpenVR Documentation**
  - API Docs: https://github.com/ValveSoftware/openvr/wiki/API-Documentation
  - Getting started: https://github.com/ValveSoftware/openvr/wiki/Tutorial

- **Stereoscopic Rendering Theory**
  - NVIDIA 3D Vision Developer Guide (archived but math still applies)
  - Key concepts: IPD (Inter-Pupillary Distance), projection matrix offsets, convergence

---

## Project Phases - Detailed Breakdown

### Phase 1: Head Tracking Bridge (Weeks 1-3)
**Scope:** External C++ application that reads VR headset orientation and controls GT4 camera via PINE IPC

**Deliverables:**
- Standalone executable that connects to PCSX2
- Reads VR headset rotation (pitch, yaw, roll)
- Writes camera angles to GT4's memory
- Configurable via INI file or command-line args

**Key Tasks:**
1. Set up C++ project with CMake
2. Integrate OpenVR SDK
3. Implement PINE IPC client (reference: ps2_cam_acolyte)
4. Find GT4 Spec II camera memory addresses (adapt from Nenkai's patches)
5. Map VR orientation to GT4 camera values
6. Test with GT4 cockpit view

**Success Criteria:**
- Head movements in VR control in-game camera
- Smooth tracking with <20ms latency
- Stable connection to PCSX2
- Works in GT4 Spec II races

**Required Knowledge:**
- C++ programming
- Socket programming (PINE uses sockets)
- Basic 3D math (quaternions, euler angles)
- OpenVR API basics

**Estimated Complexity:** MODERATE - Well-documented APIs, working examples exist

---

### Phase 2: Stereoscopic 3D Rendering (Weeks 4-8)
**Scope:** Fork PCSX2 and modify Vulkan renderer to output side-by-side stereoscopic 3D

**Deliverables:**
- Modified PCSX2 with "Enable VR Stereo" option
- Side-by-side rendering output
- Configurable IPD and convergence
- Hotkeys for real-time adjustment

**Key Tasks:**
1. Fork PCSX2 repository
2. Build PCSX2 from source successfully
3. Study Vulkan renderer code structure:
   - `pcsx2/GS/Renderers/Vulkan/GSDeviceVK.cpp`
   - `pcsx2/GS/Renderers/Vulkan/GSDeviceVK.h`
   - `pcsx2/GS/Renderers/HW/GSRendererHW.cpp`
4. Identify where final frame is rendered
5. Implement dual rendering passes:
   - Left eye: render to left half of framebuffer
   - Right eye: render to right half of framebuffer
6. Add projection matrix offsets for each eye
7. Add UI settings for VR configuration
8. Handle UI elements (keep HUD/menus flat, not in 3D)

**Success Criteria:**
- GT4 renders in side-by-side 3D
- Adjustable depth/convergence
- Maintains 60 FPS (at minimum)
- No visual artifacts or crashes
- Works with VR headset in SBS mode

**Required Knowledge:**
- C++ programming (intermediate/advanced)
- Vulkan API fundamentals
- Graphics pipeline concepts
- 3D projection matrices
- Stereoscopic rendering theory

**Estimated Complexity:** HIGH - Requires understanding PCSX2 internals and Vulkan

**Important Files to Study:**
```
pcsx2/GS/Renderers/Vulkan/
├── GSDeviceVK.cpp          # Main Vulkan device
├── GSDeviceVK.h
├── VKBuilders.cpp          # Pipeline builders
├── VKStreamBuffer.cpp
├── VKShaderCache.cpp
└── (various shader files)

pcsx2/GS/Renderers/HW/
├── GSRendererHW.cpp        # Hardware renderer base
└── GSTextureCache.cpp

pcsx2/GS/
├── GSState.cpp             # Graphics state machine
└── GS.cpp                  # Main GS interface
```

**Build Instructions:**
```bash
# Clone and setup
git clone https://github.com/PCSX2/pcsx2.git
cd pcsx2
git checkout -b gt4-vr-stereo

# Create build directory
mkdir build
cd build

# Configure with CMake
cmake .. -DCMAKE_BUILD_TYPE=Release

# Build (Windows - Visual Studio)
cmake --build . --config Release

# Or open pcsx2.sln in Visual Studio
```

---

### Phase 3: Fixed Foveated Rendering (Weeks 9-12)
**Scope:** Add Variable Rate Shading to improve performance by rendering peripheral vision at lower resolution

**Deliverables:**
- Foveated rendering option in VR settings
- Configurable foveation pattern
- 20-30% performance improvement
- Higher resolution or better graphics at same FPS

**Key Tasks:**
1. **Easy Path:** Test NVIDIA VRSS first
   - Enable in NVIDIA Control Panel
   - May work automatically with your VR setup
   - Zero code changes required
   
2. **Implementation Path:** Add VRS to Vulkan renderer
   - Enable `VK_KHR_fragment_shading_rate` extension
   - Create shading rate image (circular foveation pattern)
   - Apply to render passes
   - Add UI controls for foveation intensity

**Success Criteria:**
- Measurable FPS improvement (20-30%)
- No noticeable quality loss in center vision
- Smooth foveation gradients (not jarring)
- Configurable intensity

**Required Knowledge:**
- Vulkan extensions
- Variable Rate Shading concepts
- Performance profiling
- Image processing basics

**Estimated Complexity:** MODERATE - Vulkan VRS is well-documented, but integration takes time

**Key Vulkan Extensions:**
```cpp
// Required extensions
VK_KHR_fragment_shading_rate
VK_KHR_create_renderpass2

// Query support
vkGetPhysicalDeviceFragmentShadingRatesKHR

// Apply shading rate
VkPipelineFragmentShadingRateStateCreateInfoKHR
```

---

### Phase 4: Eye-Tracked Foveated Rendering (Optional, Weeks 13+)
**Scope:** Dynamic foveation that follows eye gaze for maximum performance with eye-tracking VR headsets

**Requirements:**
- VR headset with eye tracking (Quest Pro, PSVR2, Vive Pro Eye, Vision Pro)
- OpenXR (required by most eye-tracking implementations)
- Advanced Vulkan knowledge

**Deliverables:**
- Eye tracking integration
- Dynamic foveation map that follows gaze
- 30-50% additional performance gain

**Key Tasks:**
1. Switch from OpenVR to OpenXR (if not already using it)
2. Enable eye tracking extensions
3. Read gaze position every frame
4. Update shading rate image based on gaze
5. Handle edge cases (blinks, tracking loss)

**Success Criteria:**
- Foveation follows eye gaze accurately
- <50ms latency from gaze to foveation update
- 30-50% performance improvement over fixed foveation
- Imperceptible quality difference

**Required Knowledge:**
- OpenXR API
- Eye tracking concepts
- Low-latency programming
- Advanced Vulkan

**Estimated Complexity:** VERY HIGH - Cutting edge VR tech, requires specific hardware

**Note:** This phase is OPTIONAL. Fixed foveation (Phase 3) provides most of the benefit.

---

## Technical Architecture

### System Overview
```
┌─────────────────┐
│   VR Headset    │
│  (OpenVR/XR)    │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│  Head Tracking Bridge (External Tool)   │
│  - Reads HMD orientation                │
│  - Converts to camera angles            │
│  - Sends via PINE IPC                   │
└────────┬────────────────────────────────┘
         │ PINE IPC Socket
         ▼
┌─────────────────────────────────────────┐
│  Modified PCSX2 (Your Fork)             │
│  ┌───────────────────────────────────┐  │
│  │  Vulkan Renderer (Modified)       │  │
│  │  - Stereoscopic rendering         │  │
│  │  - Foveated rendering (VRS)       │  │
│  │  - Side-by-side output            │  │
│  └───────────────────────────────────┘  │
│  ┌───────────────────────────────────┐  │
│  │  PS2 Emulation Core               │  │
│  │  - Receives camera data           │  │
│  │  - Runs GT4 Spec II               │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
         │ Stereo frames
         ▼
┌─────────────────┐
│  VR Compositor  │
│  (SteamVR/Meta) │
└─────────────────┘
```

### Data Flow

**Head Tracking:**
```
VR Headset Sensors → OpenVR API → Your Bridge App → PINE IPC → 
PCSX2 Memory (0x89B0DE) → GT4 Camera System → Updated View
```

**Rendering:**
```
GT4 Graphics Commands → PCSX2 GS Emulation → Vulkan Renderer →
[Left Eye Pass] + [Right Eye Pass] → Side-by-Side Framebuffer →
VR Compositor → VR Headset Display
```

**Foveation:**
```
Eye Position (center or tracked) → Shading Rate Image Generation →
VRS Applied to Render Passes → Reduced pixel work in periphery →
Performance Gain
```

---

## Memory Addresses & Game-Specific Data

### GT4 Spec II Information
- **CRC Hash:** `4CE521F2`
- **Base Version:** GT4 Online Public Beta (NTSC)
- **Region:** NTSC-U/C
- **Game ID:** SCUS-97436 (Online Beta base)

### Camera Control Memory (Approximate - Requires Verification)
These addresses are from GT4 Online Beta. GT4 Spec II is based on this version, so addresses should be similar or identical:

```
0x89B0DE  - Gamepad input value (R3 stick for camera)
0x2063D8  - computeViewAngle() function location
0xF1100   - Custom code injection area (used by patches)
```

**Important:** You'll need to verify these addresses for GT4 Spec II specifically by:
1. Using PCSX2's debugger
2. Applying Nenkai's patches and observing behavior
3. Using Cheat Engine to search for values

### PINE IPC Connection
```cpp
// Default PINE socket
Windows: "\\\\.\\pipe\\pcsx2.sock"
Linux/Mac: "/tmp/pcsx2.sock"

// Or configured in PCSX2 settings
// Configuration → Network & HDD → Pine Settings
```

---

## Configuration Files & Project Structure

### Recommended Project Structure
```
gt4-vr-project/
├── head-tracking-bridge/       # Phase 1: External tool
│   ├── src/
│   │   ├── main.cpp
│   │   ├── pine_client.cpp     # PINE IPC implementation
│   │   ├── vr_reader.cpp       # OpenVR integration
│   │   └── mapper.cpp          # VR → GT4 value mapping
│   ├── include/
│   ├── CMakeLists.txt
│   └── config.ini              # User settings
│
├── pcsx2-fork/                 # Phase 2+: Modified PCSX2
│   ├── pcsx2/
│   │   ├── GS/
│   │   │   └── Renderers/
│   │   │       └── Vulkan/     # Your modifications here
│   │   └── IPC.h               # PINE protocol
│   └── ... (standard PCSX2 structure)
│
├── docs/
│   ├── memory-addresses.md     # Documented GT4 Spec II addresses
│   ├── build-instructions.md
│   └── testing-notes.md
│
└── tools/
    ├── memory-scanner/         # Helper tools
    └── config-editor/
```

### Example Configuration File (config.ini)
```ini
[PCSX2]
pine_socket = \\\\.\\pipe\\pcsx2.sock
timeout_ms = 100

[VR]
tracking_source = openvr
ipd_meters = 0.063
update_rate_hz = 90

[GT4]
crc_hash = 4CE521F2
camera_address = 0x89B0DE
sensitivity = 1.0
invert_pitch = false
invert_yaw = false

[Performance]
enable_logging = false
show_fps = true
```

---

## Development Workflow

### Getting Started (Day 1)
1. Install all required software
2. Download GT4 Spec II
3. Install PCSX2 and get GT4 running perfectly
4. Enable PINE IPC in PCSX2 settings
5. Test that GT4 Spec II works with Vulkan renderer
6. Verify VR headset works with SteamVR/OpenXR

### Phase 1 Development Loop
1. Write code for head-tracking bridge
2. Compile and run alongside PCSX2
3. Start GT4 race in cockpit view
4. Test head tracking responsiveness
5. Adjust sensitivity/mapping
6. Iterate

### Phase 2 Development Loop
1. Modify PCSX2 Vulkan renderer code
2. Rebuild PCSX2 (can take 5-15 minutes)
3. Run your custom PCSX2 build
4. Load GT4 Spec II
5. Enable VR stereo mode
6. Test in VR headset
7. Debug issues
8. Iterate

### Testing Checklist
- [ ] Head tracking works smoothly in menus
- [ ] Head tracking works in races
- [ ] Stereoscopic 3D has proper depth
- [ ] No visual artifacts (tearing, flickering)
- [ ] Performance is 60+ FPS (90+ ideal for VR)
- [ ] No crashes or stability issues
- [ ] Works with different tracks
- [ ] Works with different cars
- [ ] UI elements are readable
- [ ] Long play sessions (30+ minutes) stable

---

## Common Challenges & Solutions

### Challenge 1: Finding GT4 Spec II Camera Addresses
**Problem:** Memory addresses from GT4 Online patches may not match exactly
**Solution:**
- Use PCSX2's built-in debugger
- Apply Nenkai's patches and observe which addresses they modify
- Use Cheat Engine to search for camera-related values while playing
- Cross-reference with GT4 Online Beta addresses as starting point

### Challenge 2: PINE IPC Connection Issues
**Problem:** Can't connect to PCSX2 from your bridge app
**Solution:**
- Verify PINE is enabled in PCSX2 settings
- Check firewall isn't blocking local socket connections
- Ensure PCSX2 is running before starting bridge app
- Verify socket path is correct for your OS
- Reference ps2_cam_acolyte for working connection code

### Challenge 3: Vulkan Build Errors
**Problem:** PCSX2 won't build or Vulkan SDK issues
**Solution:**
- Ensure Vulkan SDK is installed correctly
- Update graphics drivers
- Check CMake configuration output for missing dependencies
- Verify Visual Studio has C++ workload installed
- Check PCSX2 Discord/forums for build help

### Challenge 4: Stereoscopic Rendering Artifacts
**Problem:** Double vision, incorrect depth, distortion
**Solution:**
- Verify IPD settings match your physical IPD
- Check projection matrix offset calculations
- Ensure viewport is split correctly (exact half)
- Test convergence values (where 3D plane sits)
- Start with conservative depth and increase gradually

### Challenge 5: Performance Too Low for VR
**Problem:** Can't maintain 60+ FPS
**Solution:**
- Reduce internal resolution in PCSX2
- Disable texture filtering/enhancements
- Enable speedhacks in PCSX2 (carefully)
- Implement foveated rendering (Phase 3)
- Consider hardware upgrade

### Challenge 6: Head Tracking Latency
**Problem:** Noticeable delay between head movement and camera
**Solution:**
- Increase PINE IPC update rate
- Reduce sleep/wait times in your bridge loop
- Use OpenVR's prediction features
- Profile code to find bottlenecks
- Consider direct memory access instead of PINE (advanced)

---

## Testing & Validation

### Phase 1 Testing
**Manual Tests:**
- Head movements translate to camera movements
- Sensitivity feels natural (not too fast/slow)
- No jitter or stuttering
- Works in different camera views (cockpit, chase)
- Reconnects properly after PCSX2 restart

**Performance Metrics:**
- Update rate: 90+ Hz ideal, 60 Hz minimum
- Latency: <20ms from head movement to camera update
- CPU usage: <5% for bridge app
- Memory: <50MB for bridge app

### Phase 2 Testing
**Manual Tests:**
- 3D depth looks correct (not too shallow/deep)
- Objects at different distances have proper parallax
- HUD/UI readable and positioned correctly
- No double-vision or eye strain
- Works in VR theater mode or direct SBS output

**Performance Metrics:**
- FPS: 60+ minimum, 90+ ideal
- Frame time: <16.7ms (for 60 FPS)
- GPU usage: Monitor for bottlenecks
- VRAM usage: Track memory consumption

### Phase 3 Testing
**Manual Tests:**
- Foveation not noticeable in peripheral vision
- Center of view remains sharp
- No jarring transitions in quality
- Performance improvement measurable

**Performance Metrics:**
- FPS improvement: 20-30% target
- Shading rate pattern coverage
- GPU load reduction

---

## Resources for Help

### Communities
- **PCSX2 Discord:** https://discord.com/invite/TCz3t9k
- **PCSX2 Forums:** https://forums.pcsx2.net/
- **GT Modding Discord:** (ask in GT modding hub)
- **r/emulation Reddit:** https://reddit.com/r/emulation
- **Vulkan Discord:** https://discord.gg/vulkan

### Documentation to Read
1. **PCSX2 Build Guide:** https://github.com/PCSX2/pcsx2/wiki/Building-on-Windows
2. **Vulkan Tutorial:** https://vulkan-tutorial.com/
3. **OpenVR Quick Start:** https://github.com/ValveSoftware/openvr/wiki/API-Documentation
4. **Stereoscopic Rendering Basics:** Search for "stereoscopic 3D rendering tutorial"
5. **PINE IPC Examples:** Study ps2_cam_acolyte source code

### Debugging Tools
- **RenderDoc:** Vulkan frame debugger - https://renderdoc.org/
- **NVIDIA Nsight:** Graphics profiler for NVIDIA GPUs
- **Cheat Engine:** Memory scanner for finding game addresses
- **PCSX2 Debugger:** Built into PCSX2, access via Debug menu
- **Visual Studio Debugger:** For C++ debugging

---

## Expected Timeline & Milestones

### Week 1: Environment Setup ✓
- All software installed
- PCSX2 building from source successfully
- GT4 Spec II running perfectly
- VR headset connected and working

### Week 2-3: Phase 1 Complete ✓
- Head tracking bridge application working
- Smooth camera control in GT4
- Configurable and stable

### Week 4-6: Phase 2 In Progress
- PCSX2 fork with stereoscopic rendering
- Basic SBS output working
- Debugging visual artifacts

### Week 7-8: Phase 2 Complete ✓
- Stereoscopic 3D fully functional
- Playable in VR headset
- Performance acceptable

### Week 9-12: Phase 3 (Foveation)
- Fixed foveated rendering implemented
- 20-30% performance gain
- Polish and optimization

### Week 13+: Optional Phase 4
- Eye-tracked foveation (if hardware available)
- Advanced features and polish

---

## Success Criteria - Final Deliverable

### Minimum Viable Product (MVP)
- ✅ GT4 Spec II runs in PCSX2
- ✅ Head tracking controls camera in races
- ✅ Stereoscopic 3D rendering works
- ✅ Viewable in VR headset
- ✅ 60 FPS minimum in races
- ✅ Stable for 30+ minute play sessions

### Full Feature Set
- ✅ All MVP criteria
- ✅ Fixed foveated rendering
- ✅ Configurable settings UI
- ✅ Hotkeys for adjustments
- ✅ 90 FPS in VR
- ✅ Works with multiple tracks/cars
- ✅ Installation/setup documentation

### Dream Version
- ✅ All above
- ✅ Eye-tracked foveation
- ✅ Haptic feedback integration
- ✅ Motion platform support
- ✅ Multiplayer VR support
- ✅ Video capture of VR gameplay

---

## Quick Start Commands

### Clone and Build PCSX2
```bash
git clone https://github.com/PCSX2/pcsx2.git
cd pcsx2
git checkout -b gt4-vr
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
cmake --build . --config Release
```

### Create Head Tracking Bridge Project
```bash
mkdir gt4-vr-bridge
cd gt4-vr-bridge
# Create CMakeLists.txt
# Add OpenVR dependency
# Implement PINE client
# Build and test
```

### Apply GT4 Spec II Patch
```bash
# Download xDelta patcher
# Apply Spec II patch to GT4 ISO
# Verify CRC: 4CE521F2
# Load in PCSX2
```

---

## Final Notes

**This is a complex project** but broken into manageable phases. Each phase delivers value independently:

- **Phase 1 alone** gives you head-look in VR theater mode (already awesome!)
- **Phase 2** adds true 3D depth (major immersion boost)
- **Phase 3** optimizes performance (enables higher quality)
- **Phase 4** is optional luxury (for enthusiasts with hardware)

**Don't try to do everything at once.** Get Phase 1 working first, celebrate that achievement, then move to Phase 2.

**You're building on solid foundations:**
- PCSX2 is mature and well-documented
- GT4 Spec II modding community is active
- Vulkan VR rendering is proven technology
- Working examples exist (ps2_cam_acolyte)

**This project is achievable!** Take it step by step, ask for help when needed, and enjoy the process of bringing GT4 into VR.

Good luck, and have fun racing in virtual reality! 🏎️🥽

---

## Quick Reference Links

| Resource | URL |
|----------|-----|
| PCSX2 Source | https://github.com/PCSX2/pcsx2 |
| GT Modding Hub | https://nenkai.github.io/gt-modding-hub/ |
| GT4 Patches | https://nenkai.github.io/gt-modding-hub/ps2/gt4/patches/ |
| ps2_cam_acolyte | https://github.com/moonlessformless/ps2_cam_acolyte |
| OpenVR SDK | https://github.com/ValveSoftware/openvr |
| Vulkan SDK | https://vulkan.lunarg.com/ |
| GT4 Spec II Thread | https://www.gtplanet.net/forum/threads/gran-turismo-4-spec-ii.405537/ |
| PCSX2 Discord | https://discord.com/invite/TCz3t9k |