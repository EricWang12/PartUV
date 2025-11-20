
<h1 align="center">PartUV: Part-Based UV Unwrapping of 3D Meshes</h1> 
<h3 align="center">SIGGRAPH Asia 2025</h3> 

<p align="center">
<a href="#"><img src="https://img.shields.io/badge/arXiv-Paper-red?logo=arxiv&logoColor=white" alt="arXiv"></a>
<a href="https://www.zhaoningwang.com/PartUV"><img src="https://img.shields.io/badge/Project_Page-Website-green?logo=googlechrome&logoColor=white" alt="Project Page"></a>
</p>

Official implementation for ***PartUV: Part-Based UV Unwrapping of 3D Meshes***.
<p align="center"><img src="doc/partuv_teaser.png" width="100%"></p>
---

<!-- TOC -->
<details open>
  <summary><strong>Table of Contents</strong></summary>

- [Installation](#installation)
  - [PartUV (for UV Unwrapping)](#partuv-for-uv-unwrapping)
  - [Packing with bpy (optional)](#packing-with-bpy-optional)
- [Demo](#demo)
  - [Step 1: UV Unwrapping](#step-1-uv-unwrapping)
  - [Step 2: Packing](#step-2-packing)
- [Part-Based Packing with UVPackMaster](#part-based-packing-with-uvpackmaster)
- [Benchmarking 🚧](#benchmarking)
- [Building](#building-from-source)
- [Common Problems, Acknowledgement, and BibTeX](#common-problems)
</details>
<!-- /TOC -->




## 🚧 TODO List 
- [ ] Resolve the handling of non-2-manifold meshes, see [Known Issues](#known-issues)
- [ ] Release benchmark code and data
- [ ] Multi-atlas packing with uvpackmaster
- [ ] Blender plugin for PartUV


# Installation

## PartUV (for UV Unwrapping)

```bash
# 1) Create and activate environment
conda create -y --name partuv python=3.11
conda activate partuv

# 2) Install PyTorch 2.7.1 (CUDA 12.8 wheels) and torch-scatter
# It should work with other PyTorch/CUDA versions, but those are not tested.
# conda install nvidia/label/cuda-12.8.1::cuda-toolkit
pip install torch==2.7.1 --index-url https://download.pytorch.org/whl/cu128
pip install torch-scatter -f https://data.pyg.org/whl/torch-2.7.1+cu128.html

# 3) Install project requirements
pip install -r requirements.txt
pip install partuv
```

Download the PartField checkpoint from [PartField](https://github.com/nv-tlabs/PartField):

```bash
wget https://huggingface.co/mikaelaangel/partfield-ckpt/resolve/main/model_objaverse.ckpt ./
```

## Packing with bpy (optional)

```bash
# For Python 3.11+
pip install bpy
# For Python 3.10
pip install bpy==4.0.0 --extra-index-url https://download.blender.org/pypi/
```

---




# Demo

## TL;DR

```bash
python demo/partuv_demo.py --mesh_path {input_mesh_path} --save_visuals
```

<!-- # Step 1: UV unwrapping  
# Step 2: Pack the UV results with bpy
python -m pack.pack --partuv_output_path {partuv_output_folder} --save_visuals -->

---

## Step 1: UV Unwrapping

The demo takes a 3D mesh (e.g., .obj or .glb) as input and outputs the mesh with unwrapped UVs in a non-packed format.

### Input Requirement

We recommend using meshes without 3D self-intersections and non-2-manifold edges, as they may result in highly fragmented UVs.

### Preprocessing

The input mesh is first preprocessed, including:

* Mesh repair (e.g., merging nearby vertices, fixing non–2-manifold meshes, etc.)
* (Optional) Exporting the mesh to a `.obj` file
* Running PartField to obtain the hierarchical part tree

### Unwrapping
We then call our pre-built pip wheels for unwrapping. Two main API versions are provided:

* **`pipeline_numpy`**: The default version. It takes mesh NumPy arrays (`V` and `F`), the PartField dictionary (a hierarchical tree), a configuration file path, and a distortion threshold as input. Note that the distortion threshold specified here will override the value defined in the configuration file.
* **`pipeline`**: Similar to `pipeline_numpy`, but it takes file paths as input and performs I/O operations directly from disk.

### Output Results
Both APIs save the results to the output folder.
The final mesh with unwrapped UVs is saved as `final_components.obj`.
Each chart is flattened to a unit square, but inter-chart arrangement is not yet solved.

Individual parts are also saved as `part_{i}.obj`, which can be used with UVPackMaster to produce part-based UV packing (where charts belonging to the same part are grouped nearby). See the later section for more details.

The saving behavior can be configured in the [`save_results`](demo/partuv_demo.py#L119) function.

If you specify the `--pack_method` flags, the code will pack the UVs and save the final mesh in `final_packed.obj`.

### Hyperparameters

By default, the API reads all hyperparameters from `config/config.yaml`.
See [config.md](doc/config.md) for more details on hyperparameters and usage examples for customizing them to suit your needs.

---

## Step 2: Packing

The unwrapping API outputs UVs in a non-packed format.
You can pack all UV charts together to create a UV map for the input mesh. Two packing methods are supported:

* **`blender`**: The default packing method. We provide a script (`pack/pack_blender.py`) that uses `bpy` for packing, which is called by default in the demo file.
* **`uvpackmaster`**: A paid Blender add-on. We use this to achieve part-based packing (charts from the same part are packed close together) or automatic multi-atlas packing. Please see more details below.

---

# Part-Based Packing with UVPackMaster

In our results, we include both **part-based packing** (where charts from the same part are packed close together) and **automatic multi-atlas packing** (given *N* desired tiles, parts are assigned to tiles according to the hierarchical part tree).

These results are packed using [UVPackMaster](https://uvpackmaster.com/), which unfortunately is a paid tool. We provide scripts to pack the UVs with UVPackMaster.

## Installation

1. **Install BlenderProc:**
   We use BlenderProc to run this add-on within Blender. Please follow the instructions in the [BlenderProc repository](https://github.com/DLR-RM/blenderproc) to install it.

   ```bash
   pip install blenderproc
   ```

2. **Install UVPackMaster:**
   Follow the instructions on the [UVPackMaster website](https://uvpackmaster.com/) to obtain the Linux distribution. Download the ZIP file and place it in the `extern/uvpackmaster` folder.

3. **Install the add-on:**
   We provide a script to install the add-on:

   ```bash
   blenderproc run pack/install_uvp.py
   ```

## Usage

To pack UVs with UVPackMaster, use the same command as the default packing method, changing the `--pack_method` flag to `uvpackmaster`:

```bash
python demo/partuv_demo.py --mesh_path {input_mesh_path} --pack_method uvpackmaster --save_visuals
```

## Multi-Atlas Packing 🚧

---

# Benchmarking 🚧

---


# Building from Source

Please refer to [build.md](doc/build.md) for detailed build instructions.

---

# Known Issues

### Handling of non-2-manifold meshes

The ABF assumes the mesh is 2-manifold (each edge is incident to at most two faces). This is currently handled in the preprocessing step, by splitting vertices on non-manifold edges. However, this may create split faces which could result in single-face UV charts. We are working on a better solution to handle this.


# Common Problems

Below are common issues and their solutions:

#### 1. Problem with `cuda crt/math_functions.h`

Modify `math_functions.h` according to the fix described at:
[https://forums.developer.nvidia.com/t/error-exception-specification-is-incompatible-for-cospi-sinpi-cospif-sinpif-with-glibc-2-41/323591/3](https://forums.developer.nvidia.com/t/error-exception-specification-is-incompatible-for-cospi-sinpi-cospif-sinpif-with-glibc-2-41/323591/3)

#### 2. Floating-Point Error

Disable PAMO when running the pipeline on CPU machines.

#### 3. ImportError: `GLIBCXX_3.4.32` not found

```bash
export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libstdc++.so.6
python demo/partuv_demo.py
```

#### 4. (Build) `nvcc fatal: Unsupported gpu architecture 'compute_120'`

Remove `compute_120` from the `CMAKE_CUDA_ARCHITECTURES` in `CMakeLists.txt`.

---



---

# 🍀 Acknowledgement

We acknowledge the following repositories for their contributions and code:

* [PartField](https://github.com/nv-tlabs/PartField)
* [OpenABF](https://github.com/educelab/OpenABF)
* [MeshSimplificationForUnfolding](https://git.ista.ac.at/mbhargav/mesh-simplification-for-unfolding)
* [LSCM](https://github.com/icemiliang/lscm)
* [PAMO](https://github.com/SarahWeiii/pamo)

and all the libraries in the `extern/` folder.

---

# BibTeX

If this repository helps your research or project, please consider citing our work:

```bibtex
@inproceedings{wang2025partuv,
  title     = {PartUV: Part-Based UV Unwrapping of 3D Meshes},
  author    = {Wang, Zhaoning and Wei, Xinyue and Shi, Ruoxi and Zhang, Xiaoshuai and Su, Hao and Liu, Minghua},
  booktitle = {ACM SIGGRAPH Asia Conference and Exhibition on Computer Graphics and Interactive Techniques},
  year      = {2025}
}
```
