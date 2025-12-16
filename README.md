# Aquarium Simulation

A realistic 3D aquarium simulation built with OpenGL, featuring animated fish, dynamic lighting effects, and an interactive camera system.

## Table of Contents

- [Description](#description)
- [Features](#features)
- [Requirements](#requirements)
- [How to Build](#how-to-build)
  - [Linux](#linux)
  - [Windows](#windows)
- [How to Use](#how-to-use)
  - [Interactive Controls](#interactive-controls)
  - [Lighting Controls](#lighting-controls)
- [Project Structure](#project-structure)
- [Technical Details](#technical-details)
- [Performance Notes](#performance-notes)

## Description

This project implements a fully interactive 3D aquarium simulation using modern OpenGL 3.3 core profile. The simulation features autonomous fish behavior with realistic movement patterns, dynamic lighting with directional and spotlight effects, textured models with bump mapping support, and fog effects for underwater ambiance.

The aquarium contains 100 animated tropical fish, 20 randomly placed rocks, a sandy floor, and a treasure chest. Fish move autonomously using path-finding algorithms, avoiding boundaries and creating natural schooling behavior.

## Features

### Graphics & Rendering
- **Modern OpenGL 3.3**: Core profile with GLSL shaders
- **Phong Lighting Model**: Ambient, diffuse, and specular components
- **Dynamic Lighting**: Directional light (sunlight) and spotlight (focused beam)
- **Texture Mapping**: All models use high-quality JPG textures
- **Fog Effect**: Underwater atmosphere with distance-based fog
- **Real-time Rendering**: 120 FPS target with smooth animations

### Fish Behavior
- **Autonomous Movement**: Fish navigate independently using randomized pathfinding
- **Smooth Rotation**: Fish orient themselves toward their movement direction
- **Boundary Avoidance**: Constrained to aquarium bounds (-15 to +15 on X/Z, 2.5 to 7.5 on Y)
- **Variable Speed**: Randomized velocity (0.01-0.05 units/frame)
- **15 Fish Models**: Diverse tropical fish species with unique textures

### Interactive Features
- **Free Camera System**: Mouse-controlled orbital camera with click-to-enable
- **Keyboard Navigation**: WASD movement through the aquarium
- **Runtime Toggles**: Enable/disable fog, directional light, and spotlight
- **Debug Controls**: Real-time adjustment of lighting parameters
- **Smooth Scrolling**: Mouse wheel zoom

## Requirements

### Build Dependencies
- **CMake** (3.10+) or compatible build system
- **C++ Compiler** supporting C++11 (GCC 7+, Clang 5+, MSVC 2017+)

### Runtime Libraries
- **OpenGL** 3.3+ (Core Profile)
- **GLFW3** 3.2+ (window management and input)
- **GLAD** (OpenGL function loader)
- **GLM** 0.9.9+ (mathematics library)
- **stb_image** (texture loading, included)

### Platform-Specific
- **Linux**: X11 development libraries, Mesa drivers
- **Windows**: Visual Studio 2017+ or MinGW-w64

## How to Build

### Linux

**Install dependencies (Fedora):**
```bash
sudo dnf install cmake gcc-c++ pkgconfig
sudo dnf install glfw-devel glm-devel
```

**Build the project:**
```bash
mkdir build && cd build
cmake --preset linux-vcpkg
cmake --build build
./aquarium_simulation
```

### Windows

**Using Visual Studio:**
1. Install Visual Studio 2017+ with C++ tools
2. Install vcpkg package manager:
   ```cmd
   git clone https://github.com/Microsoft/vcpkg.git
   cd vcpkg
   .\bootstrap-vcpkg.bat
   .\vcpkg integrate install
   ```
   
3. Open project in Visual Studio or use CMake:
   ```cmd
   mkdir build && cd build
   cmake --preset win-vcpkg
   cmake --build build --config Release
   ```

## How to Use

### Running the Application

```bash
./aquarium_simulation
```

The window will open showing the aquarium from a default camera position. You can immediately interact with the scene.

### Interactive Controls

#### Camera Controls
| Control | Action |
|---------|--------|
| **Left Mouse Button** | Hold to enable camera rotation (cursor disappears) |
| **Release Left Mouse** | Disable camera rotation (cursor reappears) |
| **Mouse Movement** | Rotate camera around aquarium (when enabled) |
| **Mouse Scroll** | Zoom in/out (FOV adjustment) |
| **W** | Move camera forward |
| **S** | Move camera backward |
| **A** | Move camera left |
| **D** | Move camera right |

#### Feature Toggles
| Key | Action |
|-----|--------|
| **F1** | Toggle underwater fog effect |
| **F2** | Toggle spotlight (focused beam from above) |
| **F3** | Toggle directional light (ambient sunlight) |
| **ESC** | Exit application |

### Lighting Controls

#### Debug Controls (Advanced)
These controls allow real-time adjustment of lighting parameters:

**Spotlight Position & Cutoff:**
| Key | Action |
|-----|--------|
| **R** | Move spotlight up (+0.1) |
| **F** | Move spotlight down (-0.1) |
| **T** | Increase inner cutoff angle (+0.01) |
| **G** | Decrease inner cutoff angle (-0.01) |
| **Y** | Increase outer cutoff angle (+0.01) |
| **H** | Decrease outer cutoff angle (-0.01) |

**Directional Light Ambient Color:**
| Key | Action |
|-----|--------|
| **U** | Increase red component (+0.01) |
| **J** | Decrease red component (-0.01) |
| **I** | Increase green component (+0.01) |
| **K** | Decrease green component (-0.01) |
| **O** | Increase blue component (+0.01) |
| **L** | Decrease blue component (-0.01) |


## Technical Details

### Architecture

#### Core Classes

**Camera**
- Orbital camera system centered at origin
- Spherical coordinate system (radius, yaw, pitch)
- Mouse sensitivity: 0.1, movement speed: 2.5 units/sec
- Constrained pitch: 1° to 89°

**Renderable (Abstract Base)**
- OBJ model loading with vertex/UV/normal parsing
- Texture mapping via stb_image
- Transformation matrix (translate, rotate, scale)
- Virtual `draw()` and `behave()` methods

**Fish (Derived from Renderable)**
- Autonomous navigation with random waypoint selection
- Distance-based movement: `steps = distance / velocity`
- Rotation calculation using angle between vectors
- 15 unique models, randomized on instantiation

**Floor**
- 17x17 unit plane with 80x tiled texture
- Static geometry with element buffer optimization
- Switchable sand/gravel textures

**Lights**
- **DirLight**: Directional light with RGB ambient/diffuse/specular
- **SpotLight**: Position, direction, cutoff angles, attenuation

### Rendering Pipeline

1. **Initialization** (`initOpenGLProgram`)
   - Enable depth testing
   - Set clear color (0.2, 0.3, 0.3) for underwater tint
   - Configure GLFW callbacks

2. **Update Loop** (120 FPS target)
   - Process keyboard/mouse input
   - Update fish behavior (`behave()` for each fish)
   - Calculate delta time for smooth animation

3. **Render Pass** (`drawScene`)
   - Clear color and depth buffers
   - Activate shader program
   - Set view/projection matrices
   - Upload lighting uniforms
   - Draw floor (indexed drawing)
   - Draw all fish (instanced with transformations)
   - Draw rocks and treasure
   - Swap buffers

### Shader System

**Vertex Shader (`lightEffects.vs`)**
- Transforms vertices to clip space
- Passes world position and normals to fragment shader
- Applies texture coordinates

**Fragment Shader (`lightEffects.fs`)**
- Phong lighting model implementation
- Directional light contribution
- Spotlight with cutoff angle
- Fog calculation: `mix(color, fogColor, fogFactor)`
- Material properties: shininess = 32.0

### Fish Behavior Algorithm

```cpp
if (steps == 0) {
    // Choose random destination within bounds
    wantToGo = vec3(random(-MAX_X, MAX_X), 
                    random(MIN_Y, MAX_Y), 
                    random(-MAX_Z, MAX_Z));
    
    // Calculate movement steps
    distance = length(wantToGo - position);
    velocity = random(0.01, 0.05);
    steps = round(distance / velocity);
    
    // Calculate rotation angles to face destination
    rx = degrees(angleBetween(position, wantToGo).x);
    ry = degrees(angleBetween(position, wantToGo).y);
} else {
    // Move toward destination
    position += normalize(wantToGo - position) * velocity;
    steps--;
}
```

### OBJ File Loader

The custom OBJ parser supports:
- Vertex positions (`v`)
- Texture coordinates (`vt`)
- Normal vectors (`vn`)
- Triangular faces (`f v/vt/vn`)
- Automatic normalization to unit scale

### Model Specifications

**Fish Models:**
- Format: Wavefront OBJ
- Texture: JPEG (RGB)
- Typical polygon count: 500-2000 triangles
- Pre-normalized to unit scale

**Environment Models:**
- Rocks: 2 variations, randomly selected
- Treasure: Single static model at (1, 1, 1)
- Floor: Procedurally generated quad

## Performance Notes

### Optimization Strategies

1. **Static Geometry Caching**
   - VAO/VBO uploaded once during initialization
   - No dynamic mesh generation during render loop

2. **Instanced Rendering Preparation**
   - Each fish stores transformation matrix
   - Minimal CPU-GPU data transfer

3. **Texture Management**
   - stb_image loads textures once
   - Mipmaps generated for smooth LOD transitions

4. **Efficient Collision Avoidance**
   - Boundary checks only (no fish-fish collision)
   - O(1) complexity per fish

### Scalability Recommendations

- **Low-end systems**: Reduce `fishNum` to 50 in main.cpp
- **High-end systems**: Increase to 200+ for dense schools
- **4K displays**: Ensure GPU supports 3840x2160 framebuffer

**Libraries Used:**
- GLFW - Window and input management
- GLAD - OpenGL function loader
- GLM - OpenGL Mathematics
- stb_image - Texture loading

**3D Models:**
- Tropical fish models: Various sources
- Rock and treasure models: Custom created

## License

This project is developed as coursework for computer graphics education. All code is provided as-is for educational purposes.
