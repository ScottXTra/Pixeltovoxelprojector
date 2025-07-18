# Pixel to Voxel Projector

This repository contains utilities for turning 2D image sequences into a 3D voxel representation and visualising the result. It includes both C++ and Python code for processing images and FITS files, generating voxel grids and displaying them interactively.

## Features

- **ray_voxel.cpp** – standalone C++ tool that loads camera metadata and images, detects motion between frames and casts rays into a voxel grid.
- **voxelmotionviewer.py** – PyVista based viewer for voxel grids stored in `voxel_grid.bin`.
- **process_image.cpp** – pybind11 module used by `spacevoxelviewer.py` for processing astronomical FITS images.
- **spacevoxelviewer.py** – example pipeline for accumulating brightness from FITS files into a voxel grid and visualising the result.

## Building and Running

### Compiling the motion voxeliser

1. Ensure you have a C++17 compiler and the headers `nlohmann/json.hpp` and `stb_image.h` available.
2. Compile the program:
   ```bash
   g++ -std=c++17 -O2 ray_voxel.cpp -o ray_voxel
   ```
3. Run it with your metadata file and image folder:
   ```bash
   ./ray_voxel metadata.json path/to/images voxel_grid.bin
   ```
   The program builds an NxNxN grid where each voxel stores accumulated motion brightness and writes it to `voxel_grid.bin`.

### Building the Python extension

The file `process_image.cpp` is a pybind11 extension. Build it in place with:

```bash
pip install pybind11
python setup.py build_ext --inplace
```

This creates `process_image_cpp.so` which is imported by `spacevoxelviewer.py`.

### Viewing the voxel grid

After `voxel_grid.bin` is generated, run the viewer:

```bash
python voxelmotionviewer.py
```

The script extracts the brightest voxels, applies an optional rotation and opens an interactive PyVista window. Closing the window saves a screenshot in the `screenshots/` directory.

### Processing FITS images

`spacevoxelviewer.py` demonstrates a more advanced workflow for astronomical data. It reads FITS files, updates a voxel grid using the `process_image_cpp` module and visualises both the 3D grid and slices through it. Edit the configurable parameters near the top of the script to match your dataset (paths, voxel grid size, RA/Dec centre etc.).

## Example metadata entry

The C++ tool expects a JSON array where each entry describes a frame:

```json
[
  {
    "camera_index": 0,
    "frame_index": 0,
    "yaw": 0.0,
    "pitch": 0.0,
    "roll": 0.0,
    "fov_degrees": 60.0,
    "image_file": "frame000.png",
    "camera_position": [0.0, 0.0, 0.0]
  }
]
```

Each successive entry typically points to another frame of the same or a different camera.

## Repository layout

```
ray_voxel.cpp                   # C++ implementation of the basic motion voxeliser
process_image.cpp               # pybind11 extension for FITS processing
spacevoxelviewer.py             # High level FITS processing pipeline
voxelmotionviewer.py            # PyVista viewer for voxel_grid.bin
setup.py                        # Build script for process_image_cpp
examplebuildvoxelgridfrommotion.bat  # Example batch file to compile and run ray_voxel
```

These scripts can be adapted for your own datasets and workflows.

