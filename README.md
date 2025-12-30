# Map3DProject — 3D Terrain Reconstruction from SRTM (.hgt) + Voronoi Meshing (Java / LWJGL)

This project was written in the fall of 2020
**Demo video:** <a href="https://drive.google.com/drive/folders/1pwEKQi-77Xq55mWVG9d3NSWon0SXiFLw?usp=sharing" target="_blank">here</a>.

---

## What it does

1. **Reads SRTM `.hgt` elevation tiles**
   - Height samples are stored as 2-byte integers in a 1201×1201 grid (SRTM 3 arc-second tiles).
   - The tile name encodes its geographic bounds (e.g. `N20E100.hgt` covers 20°N–21°N and 100°E–101°E).
   - Elevation values can be mapped to colors (no-data areas remain transparent).

2. **Computes a Voronoi diagram (Fortune’s algorithm)**
   - Uses the Fortune sweep-line approach to split the plane into non-overlapping convex polygons.
   - Data structures:
     - Priority queues for site/circle events
     - A red–black tree to maintain the “beach line” (parabolic arcs)
   - This implementation is written in Java and was adapted from a C# codebase.

3. **Builds a renderable surface (mesh)**
   - For each Voronoi cell / site, determines the neighboring sites.
   - Sorts neighbors by angle around the site to build consistent triangles.
   - Produces a triangle mesh suitable for OpenGL rendering.

4. **Renders in OpenGL (LWJGL)**
   - GPU shaders are stored in `res/shaders`.
   - Meshes are optimized for rendering with OpenGL.

---

## Project structure

- `src/` — Java source code (Voronoi/Fortune algorithm, data loading, rendering)
- `res/shaders/` — GLSL shaders
- `data/` — input data (place `.hgt` tiles here)
- `img/` — images/screenshots for documentation
- `lwjgl_linux/` — LWJGL native binaries for Linux
- `lwjgl_win/` — LWJGL native binaries for Windows
- `out/` — build artifacts (recommended to exclude from Git)
- `Map3DProject.iml` — IntelliJ IDEA project file

---

## Requirements

- A recent **JDK** (11+ recommended)
- **IntelliJ IDEA** (recommended, since the repo contains an `.iml` project)
- OpenGL-capable GPU/driver

> Note: LWJGL native binaries appear to be bundled in `lwjgl_linux/` and `lwjgl_win/`.

---

## Quick start (IntelliJ IDEA)

1. **Clone** the repository.
2. Open the project in **IntelliJ IDEA** (it should pick up `Map3DProject.iml`).
3. Ensure `src/` is marked as **Sources Root**.
4. Configure LWJGL:
   - Add LWJGL JARs to the project/module dependencies (if not already configured).
   - Set VM option to point to the native libraries directory:
     - Linux:
       ```
       -Djava.library.path=./lwjgl_linux
       ```
     - Windows:
       ```
       -Djava.library.path=./lwjgl_win
       ```
5. Find the entry point:
   - Search in `src/` for a class containing:
     ```java
     public static void main(String[] args)
     ```
   - Run that class.

---

## Input data (.hgt)

1. Put your SRTM `.hgt` tiles into `data/`.
2. Update the code/config (where the tile name is selected) to point to your tile, e.g. `N20E100.hgt`.
3. Run the application.

---

## How it works (high level)

- **Elevation tile → height field**
  - Reads the 1201×1201 grid of 16-bit samples.
  - Optionally maps elevation to color/alpha for visualization.

- **Sampling / sites**
  - Selects/uses a set of points (sites) to build a Voronoi diagram over the area.

- **Voronoi (Fortune) → adjacency**
  - Fortune sweep-line constructs Voronoi edges and neighbor relationships.

- **Adjacency → mesh**
  - For each site:
    - take neighbors
    - sort by polar angle
    - triangulate fan around the site
  - Emit triangle list for GPU

- **Mesh → OpenGL**
  - Upload vertex/index buffers
  - Render with GLSL shaders (`res/shaders`)

---


## Credits

- Fortune’s algorithm implementation in Java (adapted from a C# codebase).
- SRTM elevation tiles are a NASA/USGS data product (see SRTM documentation for dataset licensing/usage).
