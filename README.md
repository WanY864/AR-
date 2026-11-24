# AR Glasses Virtual Prototype: Optical Design and Gaussian Avatar Try-On

An AR glasses virtual prototype combining **Zemax optical system design**, **Blender frame and housing modeling**, and a **GaussianAvatars virtual try-on application**. The final demonstration brings optical design, head-motion tracking, automatic fitting, and interactive visualization together.

## Motivation

AR glasses need both a feasible optical system and a wearable structure. Field of view, eye relief, eyebox, ray paths, and optical component placement influence how the device should be designed; the frame must also fit the wearer's head and remain aligned during movement.

This project connects optical simulation with a photorealistic virtual prototype **before physical manufacturing**. The optical work provides a validated layout and 3D housing; the Gaussian Splatting work demonstrates how the proposed glasses fit and follow a moving head.

The Gaussian Splatting application reuses the original [GaussianAvatars](https://github.com/ShenhanQian/GaussianAvatars) method without changing its reconstruction algorithm.

## Project Goals

- Design and assess an AR optical system in Zemax, including FOV, eye relief, eyebox, ray paths, distortion, and image quality.
- Create an optical-layout-consistent glasses frame and housing in Blender.
- Reconstruct and display a realistic dynamic human head using GaussianAvatars.
- Import **one** AR glasses model designed in Blender, informed by a Zemax optical layout.
- Automatically align the glasses with facial reference points.
- Attach the glasses to the head so they follow head movement.
- Integrate Gaussian rendering and conventional mesh rendering in one viewer.
- Provide interactive controls and basic geometric fitting feedback.
- Deliver a working demonstration with documented tests.

## Core Features

| Feature | Description |
| --- | --- |
| **Dynamic head visualization** | Display a reconstructed head with recorded head movement and free camera navigation. |
| **Automatic glasses fitting** | Estimate the initial glasses position using FLAME eye and nose reference regions. |
| **Head-motion attachment** | Keep the glasses synchronized with the moving head. |
| **Wear / Remove** | Show or hide the glasses instantly. |
| **Frame color** | Change the frame material color without retraining the head. |
| **Manual fit adjustment** | Fine-tune glasses position and rotation when needed. |
| **Fit inspection** | Inspect eye alignment, nose-bridge clearance, and visible mesh intersections. |
| **Joint rendering** | Render the Gaussian head and Blender glasses together with coherent camera alignment and occlusion. |

The viewer will concentrate on **head motion and eyewear fitting**, rather than facial-expression editing or glasses-style selection.

## Technical Stack

| Component | Tool / Source | Role |
| --- | --- | --- |
| Dynamic Gaussian avatar | [GaussianAvatars](https://github.com/ShenhanQian/GaussianAvatars) | Human-head reconstruction and rendering |
| Head geometry and pose | [FLAME](https://flame.is.tue.mpg.de/) | Head reference coordinates and motion |
| Baseline dataset | [NeRSemble (preprocessed)](https://github.com/ShenhanQian/GaussianAvatars/blob/main/doc/download.md) | Official reproduction data |
| Additional video data | [VHAP](https://github.com/ShenhanQian/VHAP) | Head tracking and dataset preparation |
| Glasses frame and housing | Blender | Frame, bridge, temples, optical-module enclosure, and mesh export |
| Optical system | Zemax OpticStudio | Optical layout, ray tracing, performance analysis, and parameter export |
| Application | Python + GaussianAvatars Viewer | Interactive controls and combined rendering |
| Training / rendering GPU | AutoDL | Reproduction, testing, and offline rendering |

Unity is not required for the main demonstration. The initial application will build on the existing Python viewer.

## System Workflow

```text
NeRSemble / Head Video
          |
          v
GaussianAvatars + FLAME
          |
          v
Dynamic Gaussian Head ------> Head Pose / Reference Points
                                       |
Zemax Optical Design                   |
          |                            |
          v                            |
Blender AR Glasses --------------------+
                                       |
                                       v
                          Automatic Fit + Attachment
                                       |
                                       v
                           Gaussian + Mesh Rendering
                                       |
                                       v
                         Interactive Try-On Application
                                       |
                                       v
                           Fit Inspection + Demo
```

The avatar and glasses are **independent assets**. Glasses are added after head reconstruction; changing their color or fitting position does not require retraining GaussianAvatars.

## Development Plan

The optical and GS workstreams run **in parallel**. A provisional glasses mesh can be used for early integration while the final optical housing is being designed.

### Phase 1 — GaussianAvatars Reproduction (GS)

1. Set up GaussianAvatars on AutoDL and run the pretrained example.
2. Reproduce one subject from the official preprocessed NeRSemble dataset.
3. Export head-motion data, sample renders, and a short animation.

**Result:** A working dynamic Gaussian head and rendering pipeline.

### Phase 2 — Optical Architecture and Layout (Optics)

1. Compare feasible AR optical architectures, such as birdbath and waveguide, and select one.
2. Build the optical layout in Zemax, including the display, optical surfaces, and element positions.
3. Trace the light path and establish FOV, eye relief, eyebox, and packaging constraints.

**Result:** A defined optical system with reference geometry and target specifications.

### Phase 3 — Optical Simulation and Verification (Optics)

1. Evaluate the design with ray tracing, spot diagrams, MTF, and distortion analysis.
2. Optimize the selected optical parameters and check ray-path clearances.
3. Export the optical geometry, key parameters, and simulation results for integration.

**Result:** An evaluated Zemax design and an optical-parameter file (JSON/CSV).

### Phase 4 — Glasses Frame and Housing Design (Optics / Modeling)

1. Build the front frame, bridge, temples, and optical-module housing in Blender.
2. Arrange optical elements and mounting structures according to the Zemax layout.
3. Check that the frame does not obstruct the designed light path and export the 3D mesh.

**Result:** A dimensioned AR glasses model consistent with the optical design.

### Phase 5 — Glasses Integration and Head Binding (GS)

1. Import the glasses model and align its scale, axes, and reference points with FLAME.
2. Read the FLAME head pose and update the glasses transform during recorded head motion.
3. Verify stable attachment during head rotations and translations.

The glasses follow the head through a head-local transform:

```text
T_glasses_world(t) = T_head_world(t) × T_glasses_local
```

**Result:** A dynamic Gaussian head wearing a synchronized AR glasses model.

### Phase 6 — Automatic Fitting and Geometric Inspection (GS)

1. Estimate initial glasses placement using eye and nose-bridge reference regions.
2. Add one-click automatic fitting and manual position/rotation adjustment.
3. Check eye alignment, clearances, and visible intersections with the head mesh.
4. Use the exported optical positions and eye-relief reference for geometric checks.

**Result:** Automatic fitting with visual measurements and warnings.

### Phase 7 — Interactive Viewer and Mixed Rendering (GS)

1. Render the Gaussian head and glasses mesh with consistent camera and depth handling.
2. Address face/frame occlusion and lens-material visualization.
3. Add Wear/Remove, frame-color, fitting, playback, and camera controls.
4. Export demonstration videos.

**Result:** An interactive virtual try-on application.

### Phase 8 — Integration, Testing, and Demonstration (Joint)

1. Verify that the Blender housing, optical references, and virtual fitting coordinates agree.
2. Demonstrate head-motion tracking, automatic fitting, and the final optical-glasses geometry.
3. Document fit measurements, rendering performance, and Zemax optical results.
4. Prepare the final demonstration and graduation presentation.

**Result:** A combined optical-design and Gaussian-avatar virtual prototype.

## Main Challenges

| Challenge | Engineering approach |
| --- | --- |
| **Gaussian–mesh rendering** | Share camera parameters and use consistent depth/occlusion handling for the head and glasses. |
| **Coordinate and scale matching** | Establish one coordinate convention for FLAME, Blender, and Zemax. |
| **Stable motion attachment** | Use a head-local rigid reference frame derived from FLAME pose. |
| **Automatic fitting** | Align to eye/nose geometry, check clearance, and provide manual refinement. |
| **Real-time interaction** | Extend the Python viewer incrementally, starting with verified offline rendering. |
| **Dataset compatibility** | Begin with official NeRSemble data; use VHAP when adding new head videos. |
| **Optical architecture and image quality** | Choose a feasible optical layout and validate FOV, aberrations, MTF, and distortion in Zemax. |
| **Housing and optical clearance** | Model frame and mounting geometry around the ray path and optical element positions. |

## Expected Deliverables

- Zemax optical design with ray-tracing, image-quality, and distortion results.
- Blender AR glasses frame, housing, and optical component model.
- Optical specification and component-position export for integration.
- Reproducible GaussianAvatars setup and dynamic head model.
- Blender AR glasses asset integrated with FLAME head motion.
- Automatic fitting and geometric inspection module.
- Interactive viewer with on/off, frame-color, playback, and camera controls.
- Combined Gaussian + mesh rendering with occlusion handling.
- Demo video, functional tests, and final graduation presentation.

## Team Responsibilities

| Gaussian Splatting & Application | Optical Design & Glasses Modeling |
| --- | --- |
| GaussianAvatars reproduction and head-motion data | Optical architecture selection and Zemax layout |
| FLAME-based glasses attachment | Ray tracing, FOV, eyebox, eye relief, MTF, and distortion evaluation |
| Automatic fitting and geometric inspection | Optical system refinement and component positioning |
| Gaussian + mesh rendering and viewer controls | Blender frame, temples, and optical-module housing |
| Application tests, fit evaluation, and demonstration | Optical verification, exported parameters, and design documentation |

The two workstreams meet at the **3D glasses model, coordinate system, and optical reference parameters**.

## References

- [GaussianAvatars — Official Repository](https://github.com/ShenhanQian/GaussianAvatars)
- [GaussianAvatars — Dataset Download](https://github.com/ShenhanQian/GaussianAvatars/blob/main/doc/download.md)
- [VHAP — Head Tracking](https://github.com/ShenhanQian/VHAP)
- [NeRSemble](https://tobias-kirschstein.github.io/nersemble/)
- [FLAME Head Model](https://flame.is.tue.mpg.de/)

*Research models, datasets, and third-party 3D assets remain subject to their respective licenses.*
