# WebGPU Blender CLI Rendering - Complete Documentation

> **Complete guide to understanding how the WebGPU Blender CLI rendering system works from top to bottom**

---

## Table of Contents

1. [System Overview](#system-overview)
2. [Architecture Diagram](#architecture-diagram)
3. [Server Architecture](#server-architecture)
4. [Python Blender Script](#python-blender-script)
5. [Data Flow](#data-flow)
6. [Client-Side Export](#client-side-export)
7. [Mesh Processing Pipeline](#mesh-processing-pipeline)
8. [Texture Application](#texture-application)
9. [Complete Workflow Example](#complete-workflow-example)

---

## System Overview

### What is this system?

This is a **WebGPU-based 3D rendering system** that integrates with **Blender CLI** to create high-quality, realistic baked textures. The system allows you to:
1. Create 3D scenes in a web browser using WebGPU
2. Export scenes as GLTF files
3. Send them to a backend server
4. Use Blender's powerful rendering engine to "bake" realistic lighting into textures
5. Import the baked textures back into your WebGPU scene for real-time rendering

### Key Components

```
┌─────────────────┐
│ WebGPU Frontend │
└────────┬────────┘
         │ GLTF Export
         ▼
┌─────────────────┐
│ Node.js Server  │◄──────────┐
└────────┬────────┘           │
         │ Spawn Process      │ Return Paths
         ▼                    │
┌─────────────────┐           │
│  Blender CLI    │           │
└────────┬────────┘           │
         │ Execute            │
         ▼                    │
┌─────────────────┐           │
│  Python Script  │           │
└────────┬────────┘           │
         │ Bake Textures      │
         ▼                    │
┌─────────────────┐           │
│  Cycles Engine  │           │
└────────┬────────┘           │
         │ Save               │
         ▼                    │
┌─────────────────┐           │
│ Baked Textures  ├───────────┘
└─────────────────┘
         │
         ▼
┌─────────────────┐
│ WebGPU Renderer │
└─────────────────┘
```

---

## Architecture Diagram

### System Architecture

```
User          WebGPU App      Node.js Server    Blender CLI    Python Script    Cycles Engine
 │                │                  │                │               │                 │
 │ Create 3D      │                  │                │               │                 │
 │  Scene         │                  │                │               │                 │
 ├───────────────>│                  │                │               │                 │
 │                │                  │                │               │                 │
 │ Click "Render  │                  │                │               │                 │
 │  High Quality" │                  │                │               │                 │
 ├───────────────>│                  │                │               │                 │
 │                │                  │                │               │                 │
 │                │ Export to GLTF   │                │               │                 │
 │                ├─────────────────>│                │               │                 │
 │                │                  │                │               │                 │
 │                │ POST /api/       │                │               │                 │
 │                │  render-blender  │                │               │                 │
 │                ├─────────────────>│                │               │                 │
 │                │                  │                │               │                 │
 │                │                  │ Save GLTF to   │               │                 │
 │                │                  │  uploads/      │               │                 │
 │                │                  ├───────────────>│               │                 │
 │                │                  │                │               │                 │
 │                │                  │ Spawn Blender  │               │                 │
 │                │                  │  Process       │               │                 │
 │                │                  ├───────────────>│               │                 │
 │                │                  │                │               │                 │
 │                │                  │                │ Execute       │                 │
 │                │                  │                │ blender_bake  │                 │
 │                │                  │                │  _pipeline.py │                 │
 │                │                  │                ├──────────────>│                 │
 │                │                  │                │               │                 │
 │                │                  │                │               │ Import GLTF     │
 │                │                  │                │               ├────────────────>│
 │                │                  │                │               │                 │
 │                │                  │                │               │ Setup Cycles    │
 │                │                  │                │               │  Engine         │
 │                │                  │                │               ├────────────────>│
 │                │                  │                │               │                 │
 │                │                  │                │               │ Setup Lighting  │
 │                │                  │                │               │  (Sun + HDRI)   │
 │                │                  │                │               ├────────────────>│
 │                │                  │                │               │                 │
 │                │                  │                │               │ Bake Diffuse    │
 │                │                  │                │               ├────────────────>│
 │                │                  │                │               │                 │
 │                │                  │                │               │ Bake Normal     │
 │                │                  │                │               ├────────────────>│
 │                │                  │                │               │                 │
 │                │                  │                │               │ Bake AO         │
 │                │                  │                │               ├────────────────>│
 │                │                  │                │               │                 │
 │                │                  │                │               │ Bake Emission   │
 │                │                  │                │               ├────────────────>│
 │                │                  │                │               │                 │
 │                │                  │                │               │ Textures Baked  │
 │                │                  │                │               │<────────────────┤
 │                │                  │                │               │                 │
 │                │                  │                │ Save to       │                 │
 │                │                  │                │  baked_output/│                 │
 │                │                  │                │<──────────────┤                 │
 │                │                  │                │               │                 │
 │                │                  │ Return Texture │               │                 │
 │                │                  │  Paths         │               │                 │
 │                │                  │<───────────────┤               │                 │
 │                │                  │                │               │                 │
 │                │ JSON Response    │                │               │                 │
 │                │<─────────────────┤                │               │                 │
 │                │                  │                │               │                 │
 │                │ Load Baked       │                │               │                 │
 │                │  Textures        │                │               │                 │
 │                ├─────────────────>│                │               │                 │
 │                │                  │                │               │                 │
 │ Display High-  │                  │                │               │                 │
 │  Quality Render│                  │                │               │                 │
 │<───────────────┤                  │                │               │                 │
 │                │                  │                │               │                 │
```

---

## Server Architecture

### Server File: `blender-server.js`

The server is a **Node.js Express application** that acts as a bridge between your WebGPU frontend and Blender. It handles file uploads, spawns Blender processes, and manages the baking workflow.

### Server APIs

#### 1. Health Check API

```javascript
GET /health
```

**Purpose:**
- Check if server is running

**Response:**
```json
{
  "status": "ok",
  "message": "Blender baking server is running"
}
```

---

#### 2. Render/Bake API (Main API)

```javascript
POST /api/render-blender
Content-Type: multipart/form-data
```

**Purpose:**
- Receives GLTF file from frontend
- Spawns Blender to bake textures
- Returns paths to baked textures

**Request:**
- `gltf`: GLTF file (multipart form data)

**What happens inside?**

```javascript
// 1. Save uploaded GLTF file
const gltfPath = req.file.path; // e.g., uploads/scene_2024-12-07.gltf

// 2. Create output directory for baked textures
const outputDir = path.join(__dirname, '../baked_output', gltfFilename);

// 3. Spawn Blender process
const blenderProcess = spawn(blenderPath, [
    '--background',              // Run without UI
    '--python', pythonScript,    // Execute Python script
    '--',                        // Separator for script args
    '--input', gltfPath,         // Input GLTF file
    '--output', outputDir        // Output directory
]);

// 4. Wait for Blender to complete
await new Promise((resolve, reject) => {
    blenderProcess.on('close', (code) => {
        if (code === 0) resolve();
        else reject(new Error(`Blender failed with code ${code}`));
    });
});

// 5. Scan output directory for baked textures
const bakedTextures = await scanBakedTextures(outputDir);

// 6. Return texture paths to frontend
res.json({
    success: true,
    bakedTextures: bakedTextures
});
```

**Response:**
```json
{
  "success": true,
  "message": "Baking completed successfully",
  "outputPath": "/path/to/baked_output/scene_timestamp",
  "bakedTextures": {
    "Cube": {
      "diffuse": "/baked_output/scene_timestamp/Cube_diffuse.png",
      "normal": "/baked_output/scene_timestamp/Cube_normal.png",
      "ao": "/baked_output/scene_timestamp/Cube_ao.png",
      "emission": "/baked_output/scene_timestamp/Cube_emission.png"
    },
    "Sphere": {
      "diffuse": "/baked_output/scene_timestamp/Sphere_diffuse.png",
      "normal": "/baked_output/scene_timestamp/Sphere_normal.png",
      "ao": "/baked_output/scene_timestamp/Sphere_ao.png",
      "emission": "/baked_output/scene_timestamp/Sphere_emission.png"
    }
  },
  "gltfFile": "scene_timestamp"
}
```

---

#### 3. Get Latest Baked Textures API

```javascript
GET /api/baked-textures/latest
```

**Purpose:**
- Get the most recently baked textures

---

#### 4. Delete Baked Textures API

```javascript
DELETE /api/baked-textures
```

**Purpose:**
- Delete all baked texture directories

---

### How Server Spawns Blender

The server uses Node.js `child_process.spawn()` to run Blender as a separate process. This is like opening Blender from command line, but programmatically.

```javascript
const blenderProcess = spawn(blenderPath, [
    '--background',              // No GUI
    '--python', pythonScript,    // Run Python script
    '--',                        // Args separator
    '--input', gltfPath,         // GLTF input
    '--output', outputDir,       // Output folder
    '--resolution', '2048',      // Texture size
    '--samples', '1024'          // Quality samples
]);
```

**What each argument does:**

| Argument | Purpose |
|----------|---------|
| `--background` | Run Blender without opening the UI |
| `--python` | Execute a Python script inside Blender |
| `--` | Separator between Blender args and script args |
| `--input` | Path to input GLTF file |
| `--output` | Directory to save baked textures |
| `--resolution` | Texture resolution (e.g., 2048x2048) |
| `--samples` | Rendering quality (higher = better) |

---

## Python Blender Script

### Script File: `blender_bake_pipeline.py`

This is the **heart of the baking process**. It runs inside Blender and controls everything - importing the GLTF, setting up lighting, configuring the Cycles render engine, and baking textures.

### Script Structure

```python
class GLTFBakingPipeline:
    def __init__(self, args):
        # Initialize with command line arguments
        self.input_file = args.input      # GLTF file path
        self.output_dir = args.output     # Output directory
        self.resolution = args.resolution # Texture size (2048)
        self.samples = args.samples       # Quality (1024)
        self.hdri_path = args.hdri        # HDRI environment
```

### Main Pipeline Steps

#### Step 1: Clear Scene

```python
def clear_scene(self):
    # Delete all default objects (cube, light, camera)
    bpy.ops.object.select_all(action='SELECT')
    bpy.ops.object.delete(use_global=False)
```

**Why?**
- Blender starts with default objects
- We need a clean scene for our GLTF

---

#### Step 2: Import GLTF

```python
def import_gltf(self):
    # Import GLTF file while preserving UVs
    bpy.ops.import_scene.gltf(filepath=self.input_file)
    
    # Verify UVs are preserved
    for obj in bpy.data.objects:
        if obj.type == 'MESH':
            if not obj.data.uv_layers:
                print(f"⚠️ WARNING: Mesh '{obj.name}' has no UV layers!")
```

**What are UVs?**

UVs are 2D coordinates that tell Blender how to "unwrap" a 3D mesh and map textures onto it. Think of it like unwrapping a gift box into a flat pattern.

```
3D Mesh (Cube)          UV Unwrap (Flat)
     +-----+                +--+--+
    /     /|               |  |  |
   +-----+ |               +--+--+
   |     | +               |  |  |
   |     |/                +--+--+
   +-----+
```

---

#### Step 3: Setup Cycles Engine

```python
def setup_cycles(self):
    # Set render engine to Cycles (realistic rendering)
    bpy.context.scene.render.engine = 'CYCLES'
    
    # Use GPU for faster rendering
    bpy.context.scene.cycles.device = 'GPU'
    
    # Set quality (samples = how many light rays to trace)
    bpy.context.scene.cycles.samples = self.samples  # 1024
    
    # Enable denoising (removes noise from render)
    bpy.context.scene.cycles.use_denoising = True
    
    # Global illumination settings
    bpy.context.scene.cycles.max_bounces = 8  # Light bounces
```

**What is Cycles?**

Cycles is Blender's **physically-based rendering engine**. It simulates real-world light physics by tracing light rays, creating realistic shadows, reflections, and lighting.

**Samples explained:**
- More samples = Better quality but slower
- 1024 samples = High quality
- Each sample traces a light ray

---

#### Step 4: Setup Lighting

```python
def setup_lighting(self):
    # Add sun light (directional light)
    bpy.ops.object.light_add(type='SUN', location=(0, 0, 10))
    sun = bpy.context.active_object
    sun.data.energy = 3.0  # Brightness
    sun.rotation_euler = (0.785398, 0, 0.785398)  # 45 degrees
    
    # Setup HDRI environment (360° image for realistic lighting)
    if self.hdri_path:
        world = bpy.context.scene.world
        world.use_nodes = True
        
        # Create environment texture node
        env_tex = nodes.new(type='ShaderNodeTexEnvironment')
        env_tex.image = bpy.data.images.load(self.hdri_path)
        
        # Connect to world output
        links.new(env_tex.outputs['Color'], background.inputs['Color'])
```

**What is HDRI?**

HDRI (High Dynamic Range Image) is a 360° photograph that contains lighting information. It's used to create realistic environment lighting, like outdoor scenes with sky and sun.

---

#### Step 5: Bake Textures

```python
def bake_texture(self, obj, bake_type, output_name):
    # Create empty image to bake into
    image = bpy.data.images.new(
        f"{obj.name}_{bake_type}", 
        width=self.resolution,   # 2048
        height=self.resolution   # 2048
    )
    
    # Setup material with image texture node
    self.setup_bake_material(obj, image)
    
    # Select only this object
    bpy.ops.object.select_all(action='DESELECT')
    obj.select_set(True)
    
    # Bake! (This is where the magic happens)
    bpy.ops.object.bake(type=bake_type)
    
    # Save baked image
    output_path = os.path.join(self.output_dir, output_name)
    image.filepath_raw = output_path
    image.file_format = 'PNG'
    image.save()
```

**What is Texture Baking?**

Texture baking is the process of **"burning" lighting and shadows into a texture image**. Instead of calculating lighting in real-time, we pre-calculate it and save it as an image. This makes rendering much faster.

**Texture Types Baked:**

| Type | What it stores |
|------|----------------|
| **Diffuse** | Base color + lighting |
| **Normal** | Surface details (bumps) |
| **AO** | Ambient occlusion (shadows in crevices) |
| **Emission** | Glowing parts |

---

#### Step 6: Bake All Meshes

```python
def bake_all_meshes(self):
    # Get all mesh objects
    meshes = [obj for obj in bpy.data.objects if obj.type == 'MESH']
    
    for obj in meshes:
        # Bake each texture type for this mesh
        self.bake_texture(obj, 'diffuse', f"{obj.name}_diffuse.png")
        self.bake_texture(obj, 'normal', f"{obj.name}_normal.png")
        self.bake_texture(obj, 'ao', f"{obj.name}_ao.png")
        self.bake_texture(obj, 'emission', f"{obj.name}_emission.png")
```

**Output Files:**

For a scene with a Cube and Sphere, you'll get:
```
baked_output/
  scene_2024-12-07/
    Cube_diffuse.png    ← Cube's color + lighting
    Cube_normal.png     ← Cube's surface details
    Cube_ao.png         ← Cube's ambient shadows
    Cube_emission.png   ← Cube's glowing parts
    Sphere_diffuse.png  ← Sphere's color + lighting
    Sphere_normal.png   ← Sphere's surface details
    Sphere_ao.png       ← Sphere's ambient shadows
    Sphere_emission.png ← Sphere's glowing parts
```

---

## Data Flow

### Complete Data Journey

```
┌──────────────────────────────────────┐
│ User creates 3D scene in WebGPU     │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ Click 'Render High Quality'          │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ GltfExport.ts: Export scene to GLTF │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ Convert meshes to GLTF format        │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ Embed textures as base64             │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ Create GLTF JSON + binary buffer     │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ Send to server via fetch POST        │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ Server: Save GLTF to uploads/        │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ Server: Spawn Blender process        │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ Blender: Execute Python script       │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ Python: Import GLTF                  │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ Python: Setup Cycles + Lighting      │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ Python: Bake textures for each mesh  │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ Python: Save PNGs to baked_output/   │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ Server: Scan baked_output/           │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ Server: Return texture paths as JSON │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ WebGPU: Receive texture paths        │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ WebGPU: Load baked textures          │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ WebGPU: Apply to meshes              │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ User sees high-quality render        │
└──────────────────────────────────────┘
```

---

## Client-Side Export

### File: `GltfExport.ts`

This TypeScript class handles exporting your WebGPU scene to GLTF format. It converts all meshes, materials, and textures into a standardized 3D format that Blender can understand.

### Export Process

#### Step 1: Process Meshes

```typescript
private async processMesh(mesh: MeshLike): Promise<number | null> {
    // Extract vertex data (positions)
    const vertices = mesh.vertices;  // Float32Array of XYZ positions
    
    // Generate normals (for lighting calculations)
    const normals = this.generateNormals(vertices, mesh.indices);
    
    // Get or generate UVs (texture coordinates)
    const texcoords = this.getTextureCoordinates(mesh);
    
    // Add to GLTF binary buffer
    const positionAccessor = this.addBufferView(vertices);
    const normalAccessor = this.addBufferView(normals);
    const texcoordAccessor = this.addBufferView(texcoords);
}
```

**What is a Vertex?**

A vertex is a **point in 3D space**. A mesh is made of many vertices connected by triangles. Each vertex has:
- **Position** (X, Y, Z coordinates)
- **Normal** (direction the surface faces)
- **UV** (texture coordinate)

```
Cube Mesh Example:
    v4------v5
   /|      /|
  v0------v1|
  | |     | |
  |v7-----|v6
  |/      |/
  v3------v2

8 vertices (v0-v7)
12 triangles (2 per face × 6 faces)
```

---

#### Step 2: Process Materials

```typescript
private async processMaterial(mesh: MeshLike): Promise<number> {
    const material = mesh.material;
    
    const gltfMaterial = {
        name: material.name,
        pbrMetallicRoughness: {
            baseColorFactor: material.color,    // RGB color
            metallicFactor: material.metallic,  // 0-1
            roughnessFactor: material.roughness // 0-1
        },
        emissiveFactor: material.emission       // RGB glow
    };
    
    // Process textures
    if (mesh.baseTexture) {
        gltfMaterial.pbrMetallicRoughness.baseColorTexture = 
            await this.processTexture(mesh.baseTexture);
    }
}
```

---

#### Step 3: Process Textures

```typescript
private async processTexture(textureData: any): Promise<number> {
    // Get image URL
    const imageUrl = textureData.url;
    
    // Convert to base64 data URI (embed in GLTF)
    const dataUri = await this.imageUrlToDataUri(imageUrl);
    
    // Add to GLTF images array
    this.images.push({ uri: dataUri });
    
    // Add texture reference
    this.textures.push({
        sampler: samplerIndex,
        source: imageIndex
    });
}
```

**Why convert to base64?**

Base64 encoding allows us to **embed images directly into the GLTF file** instead of having separate image files. This makes it easier to send a single file to the server.

---

#### Step 4: Create GLTF JSON

```typescript
async export(): Promise<Blob> {
    // Build GLTF JSON structure
    const gltf = {
        asset: { version: "2.0", generator: "WebGPU Exporter" },
        scene: 0,
        scenes: [{ nodes: [0, 1, 2, ...] }],
        nodes: this.nodes,
        meshes: this.meshes,
        materials: this.materials,
        textures: this.textures,
        images: this.images,
        accessors: this.accessors,
        bufferViews: this.bufferViews,
        buffers: [{ byteLength: this.binaryBuffer.byteLength }]
    };
    
    // Create GLB file (GLTF binary)
    const jsonChunk = JSON.stringify(gltf);
    const glbBlob = this.createGLB(jsonChunk, this.binaryBuffer);
    
    return glbBlob;
}
```

**GLTF File Structure:**

```json
{
  "asset": { "version": "2.0" },
  "scenes": [{ "nodes": [0, 1] }],
  "nodes": [
    { "mesh": 0, "translation": [0, 0, 0] },
    { "mesh": 1, "translation": [2, 0, 0] }
  ],
  "meshes": [
    {
      "name": "Cube",
      "primitives": [{
        "attributes": {
          "POSITION": 0,
          "NORMAL": 1,
          "TEXCOORD_0": 2
        },
        "indices": 3,
        "material": 0
      }]
    }
  ],
  "materials": [
    {
      "name": "Material",
      "pbrMetallicRoughness": {
        "baseColorTexture": { "index": 0 },
        "metallicFactor": 0.5,
        "roughnessFactor": 0.5
      }
    }
  ],
  "textures": [{ "source": 0 }],
  "images": [{ "uri": "data:image/png;base64,..." }]
}
```

---

## Mesh Processing Pipeline

### How Blender Processes Each Mesh

```
┌─────────────────┐     ┌──────────────┐     ┌──────────┐     ┌───────────────┐
│ Import GLTF     │────>│ Read         │────>│ Read UVs │────>│ Read Material │
│ Mesh            │     │ Vertices     │     │          │     │               │
└─────────────────┘     └──────────────┘     └──────────┘     └───────┬───────┘
                                                                       │
                                                                       ▼
┌─────────────────┐     ┌──────────────┐     ┌──────────────────────────┐
│ Save PNGs       │<────│ Bake Emission│<────│ Create Bake Image        │
│                 │     │              │     │                          │
└─────────────────┘     └──────┬───────┘     └──────────────────────────┘
                               │                          │
                               │                          ▼
                        ┌──────┴───────┐     ┌──────────────────────────┐
                        │ Bake AO      │<────│ Setup Image Node         │
                        │              │     │                          │
                        └──────┬───────┘     └──────────────────────────┘
                               │                          │
                               │                          ▼
                        ┌──────┴───────┐     ┌──────────────────────────┐
                        │ Bake Normal  │<────│ Select Mesh              │
                        │              │     │                          │
                        └──────┬───────┘     └──────────────────────────┘
                               │
                               │
                        ┌──────┴───────┐
                        │ Bake Diffuse │
                        │              │
                        └──────────────┘
```

### Detailed Baking Process

**For each mesh, Blender does this:**

1. **Create empty 2048×2048 image**
   ```python
   image = bpy.data.images.new("Cube_diffuse", 2048, 2048)
   ```

2. **Add image to material's shader nodes**
   ```python
   img_node = nodes.new(type='ShaderNodeTexImage')
   img_node.image = image
   nodes.active = img_node  # This is where baking will output
   ```

3. **Configure bake settings**
   ```python
   bpy.context.scene.render.bake.use_pass_direct = True    # Direct light
   bpy.context.scene.render.bake.use_pass_indirect = True  # Bounced light
   bpy.context.scene.render.bake.use_pass_color = True     # Material color
   ```

4. **Bake!**
   ```python
   bpy.ops.object.bake(type='COMBINED')  # Renders lighting into image
   ```

5. **Save image**
   ```python
   image.filepath_raw = "baked_output/Cube_diffuse.png"
   image.save()
   ```

**What happens during baking?**

Blender shoots light rays from the camera through each pixel of the UV map, calculates how light bounces around the scene, and saves the final color in the texture image.

```
UV Map (2D)          Light Calculation       Baked Texture
+--------+           Ray 1 → Bounce → Color  +--------+
|  /\  /\|           Ray 2 → Bounce → Color  |████████|
| /  \/  |    →      Ray 3 → Bounce → Color  |████░░░░|
|/      \|           ... (millions of rays)   |░░░░████|
+--------+                                    +--------+
```

---

## Texture Application

### File: `GltfBakedImport.ts`

After baking is complete, this class handles loading the baked textures back into your WebGPU scene and applying them to the correct meshes.

### Loading Baked Textures

```typescript
async loadBakedTexturesForMesh(meshName: string) {
    // Get texture paths from server response
    const textureSet = this.textureCache.get(meshName);
    
    const textures = {};
    
    // Load diffuse (contains baked lighting)
    if (textureSet.diffuse) {
        textures.baseTexture = await this.resourceManager
            .getOrLoadTexture(textureSet.diffuse);
    }
    
    // Load normal map
    if (textureSet.normal) {
        textures.normalMap = await this.resourceManager
            .getOrLoadTexture(textureSet.normal);
    }
    
    // Load AO (ambient occlusion)
    if (textureSet.ao) {
        textures.occlusionMap = await this.resourceManager
            .getOrLoadTexture(textureSet.ao);
    }
    
    // Load emission (glowing parts)
    if (textureSet.emission) {
        textures.emissiveMap = await this.resourceManager
            .getOrLoadTexture(textureSet.emission);
    }
    
    return textures;
}
```

### Creating Baked Material

```typescript
createBakedMaterial(device: GPUDevice, meshName: string) {
    // Create material with neutral properties
    const material = Material.createFromProperties(device, {
        color: [1, 1, 1],    // White - use texture color only
        metallic: 0.0,       // Lighting is baked
        roughness: 1.0,      // Lighting is baked
        emission: [0, 0, 0], // Use emissive texture
        specular: 0          // No additional specular
    });
    
    return material;
}
```

**Why neutral properties?**

Because all the lighting is already "baked" into the diffuse texture, we don't want the shader to add extra lighting calculations. We set metallic=0, roughness=1, and use white color so the texture shows exactly as baked.

---

## Complete Workflow Example

### Real-World Scenario

Let's walk through a complete example of rendering a scene with a cube and sphere.

### Step-by-Step

#### 1. User Creates Scene

```typescript
// User adds a cube
const cube = new Mesh(device, geometry, material);
cube.setName("Cube");
cube.position = [0, 0, 0];

// User adds a sphere
const sphere = new Mesh(device, sphereGeometry, material);
sphere.setName("Sphere");
sphere.position = [2, 0, 0];

// Add to scene
scene.add(cube);
scene.add(sphere);
```

#### 2. User Clicks "Render High Quality"

```typescript
async function renderHighQuality() {
    // Export scene to GLTF
    const exporter = new GltfExport(scene);
    const gltfBlob = await exporter.export();
    
    // Send to server
    const formData = new FormData();
    formData.append('gltf', gltfBlob, 'scene.gltf');
    
    const response = await fetch('http://localhost:3001/api/render-blender', {
        method: 'POST',
        body: formData
    });
    
    const result = await response.json();
    console.log('Baked textures:', result.bakedTextures);
}
```

#### 3. Server Receives Request

```javascript
// Server saves GLTF
const gltfPath = 'uploads/scene_2024-12-07.gltf';
fs.writeFileSync(gltfPath, gltfData);

// Server spawns Blender
const blender = spawn('blender', [
    '--background',
    '--python', 'scripts/blender_bake_pipeline.py',
    '--',
    '--input', gltfPath,
    '--output', 'baked_output/scene_2024-12-07'
]);
```

#### 4. Blender Bakes Textures

```python
# Python script runs in Blender
pipeline = GLTFBakingPipeline(args)
pipeline.clear_scene()
pipeline.import_gltf()  # Imports Cube and Sphere
pipeline.setup_cycles()
pipeline.setup_lighting()

# Bake Cube
pipeline.bake_texture(cube, 'diffuse', 'Cube_diffuse.png')
pipeline.bake_texture(cube, 'normal', 'Cube_normal.png')
pipeline.bake_texture(cube, 'ao', 'Cube_ao.png')
pipeline.bake_texture(cube, 'emission', 'Cube_emission.png')

# Bake Sphere
pipeline.bake_texture(sphere, 'diffuse', 'Sphere_diffuse.png')
pipeline.bake_texture(sphere, 'normal', 'Sphere_normal.png')
pipeline.bake_texture(sphere, 'ao', 'Sphere_ao.png')
pipeline.bake_texture(sphere, 'emission', 'Sphere_emission.png')
```

#### 5. Server Returns Texture Paths

```json
{
  "success": true,
  "bakedTextures": {
    "Cube": {
      "diffuse": "/baked_output/scene_2024-12-07/Cube_diffuse.png",
      "normal": "/baked_output/scene_2024-12-07/Cube_normal.png",
      "ao": "/baked_output/scene_2024-12-07/Cube_ao.png",
      "emission": "/baked_output/scene_2024-12-07/Cube_emission.png"
    },
    "Sphere": {
      "diffuse": "/baked_output/scene_2024-12-07/Sphere_diffuse.png",
      "normal": "/baked_output/scene_2024-12-07/Sphere_normal.png",
      "ao": "/baked_output/scene_2024-12-07/Sphere_ao.png",
      "emission": "/baked_output/scene_2024-12-07/Sphere_emission.png"
    }
  }
}
```

#### 6. WebGPU Loads Baked Textures

```typescript
// Load baked textures for Cube
const cubeTextures = await loadBakedTexturesForMesh('Cube');
cube.baseTexture = cubeTextures.baseTexture;
cube.normalMap = cubeTextures.normalMap;
cube.occlusionMap = cubeTextures.occlusionMap;
cube.emissiveMap = cubeTextures.emissiveMap;

// Load baked textures for Sphere
const sphereTextures = await loadBakedTexturesForMesh('Sphere');
sphere.baseTexture = sphereTextures.baseTexture;
sphere.normalMap = sphereTextures.normalMap;
sphere.occlusionMap = sphereTextures.occlusionMap;
sphere.emissiveMap = sphereTextures.emissiveMap;

// Render with baked lighting!
renderer.render(scene);
```

---

## Performance Comparison

### Before Baking vs After Baking

| Aspect | Before Baking | After Baking |
|--------|---------------|--------------|
| **Lighting Calculation** | Real-time (every frame) | Pre-calculated (once) |
| **FPS** | 30-60 FPS | 120+ FPS |
| **Quality** | Good | Excellent (realistic) |
| **Shadows** | Simple | Realistic with GI |
| **Reflections** | Limited | Baked reflections |

Baking trades **baking time** (one-time cost) for **runtime performance** (every frame). It's perfect for scenes where lighting doesn't change.

---

## Key Concepts Summary

### For Beginners

1. **WebGPU Frontend** = Your 3D app in the browser
2. **GLTF Export** = Converting your scene to a standard 3D format
3. **Node.js Server** = Middleman between browser and Blender
4. **Blender CLI** = Blender running without UI, controlled by commands
5. **Python Script** = Code that tells Blender what to do
6. **Cycles Engine** = Blender's realistic rendering engine
7. **Texture Baking** = Pre-calculating lighting and saving as images
8. **Baked Textures** = Images with lighting "burned in"

---

## Troubleshooting

### Common Issues

#### Issue 1: "Blender not found"

**Solution:**
```powershell
# Set Blender path
$env:BLENDER_PATH = "C:\Program Files\Blender Foundation\Blender 4.5\blender.exe"
```

#### Issue 2: "No UV layers"

**Cause:** Mesh doesn't have texture coordinates
**Solution:** Export generates default UVs automatically

#### Issue 3: "Baking failed"

**Check:**
1. Is Blender installed?
2. Is Python script path correct?
3. Are there any errors in server logs?

---

## Additional Resources

- **Blender Documentation**: https://docs.blender.org/
- **GLTF Specification**: https://registry.khronos.org/glTF/
- **WebGPU Guide**: https://webgpufundamentals.org/
- **Cycles Rendering**: https://docs.blender.org/manual/en/latest/render/cycles/

---

## Summary

This system creates a powerful pipeline for high-quality 3D rendering:
1. Create scenes in WebGPU (fast, interactive)
2. Export to GLTF (standard format)
3. Send to server (Node.js bridge)
4. Bake with Blender (realistic lighting)
5. Load back to WebGPU (fast rendering)

---

**Created by:** WebGPU Blender CLI Documentation Team
**Last Updated:** December 2024
**Version:** 1.0
