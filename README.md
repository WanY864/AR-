# AR Glasses Try-On with Dynamic Gaussian Avatars

An interactive virtual try-on system that places a Blender-designed AR glasses model on a photorealistic Gaussian head avatar. The system supports **automatic fitting, motion tracking, material customization, and geometric inspection**.

## Motivation

Designing AR glasses requires more than building an optical system. The frame and optical components must also be positioned correctly relative to the eyes, nose, and head, and remain aligned as the wearer moves.

This project provides a practical way to examine an AR glasses design **before manufacturing a physical prototype**. By combining dynamic Gaussian Splatting with a conventional 3D glasses model, we can visualize wearing conditions, detect geometric misalignment, and demonstrate design adjustments interactively.

Our focus is **system integration and application development**. We use the original [GaussianAvatars](https://github.com/ShenhanQian/GaussianAvatars) method without modifying its reconstruction algorithm.

## Project Goals

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
| Glasses model | Blender | Frame geometry and materials |
| Optical parameters | Zemax OpticStudio | Optical design, performance simulation, and reference values |
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
Zemax Optical Parameters               |
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

### Phase 1 — GaussianAvatars Reproduction

1. Set up the official GaussianAvatars environment on AutoDL.
2. Run the pretrained example and verify dynamic head rendering.
3. Reproduce one subject from the preprocessed NeRSemble dataset.
4. Export sample renders, head-pose data, and a short motion sequence.

**Result:** A reproducible dynamic Gaussian head and working render pipeline.

### Phase 2 — Glasses Preparation and Head Binding

1. Prepare the Blender glasses model with correct scale, orientation, and separate frame/lens materials.
2. Define eye, nose, and glasses reference points.
3. Use FLAME head pose to update the glasses transform during animation.
4. Validate attachment across head rotations and translations.

The glasses follow the head through a head-local transform:

```text
T_glasses_world(t) = T_head_world(t) × T_glasses_local
```

**Result:** The glasses stay attached while the Gaussian head moves.

### Phase 3 — Automatic Fit and Geometric Inspection

1. Calculate an initial glasses placement from FLAME head references.
2. Align the glasses bridge and optical reference points with the face.
3. Add manual adjustment controls for position and rotation.
4. Estimate clearances and flag intersections with the head mesh.
5. Display fit measurements and warnings in the viewer.

**Result:** One-click initial fitting with visible geometric feedback.

### Phase 4 — Interactive Viewer and Mixed Rendering

1. Integrate the Blender mesh into the GaussianAvatars viewing workflow.
2. Match the camera, scale, depth, and rendering order of both representations.
3. Handle face/frame occlusion and transparent lens materials.
4. Add Wear/Remove, color, fitting, playback, and camera controls.
5. Prepare an offline video export for presentations.

**Result:** A usable interactive AR glasses try-on application.

### Phase 5 — Testing and Demonstration

Test different head poses, viewing angles, and fitting adjustments. Record:

- Glasses alignment and attachment stability during motion.
- Automatic fitting results before and after manual correction.
- Visible collisions and occlusion behavior from different angles.
- Viewer responsiveness, rendering FPS, and GPU memory usage.
- Successful operation of each interactive control.

**Result:** Final demo, screenshots/videos, test results, and graduation documentation.

## Optical Design Work (Teammate)

In parallel with the GS application, the teammate will:

1. Design and simulate the AR glasses optical system in **Zemax**.
2. Determine key optical parameters, including **field of view (FOV), eye relief, eyebox**, and relevant optical-element positions.
3. Evaluate optical performance and provide a concise **JSON/CSV parameter summary** for geometric alignment checks and the final project report.

The glasses 3D asset and its integration into the avatar viewer belong to the GS/application workstream.

## Main Challenges

| Challenge | Engineering approach |
| --- | --- |
| **Gaussian–mesh rendering** | Share camera parameters and use consistent depth/occlusion handling for the head and glasses. |
| **Coordinate and scale matching** | Establish one coordinate convention for FLAME, Blender, and Zemax. |
| **Stable motion attachment** | Use a head-local rigid reference frame derived from FLAME pose. |
| **Automatic fitting** | Align to eye/nose geometry, check clearance, and provide manual refinement. |
| **Real-time interaction** | Extend the Python viewer incrementally, starting with verified offline rendering. |
| **Dataset compatibility** | Begin with official NeRSemble data; use VHAP when adding new head videos. |

## Expected Deliverables

- Reproducible GaussianAvatars setup and dynamic head model.
- Blender AR glasses asset integrated with FLAME head motion.
- Automatic fitting and geometric inspection module.
- Interactive viewer with on/off, frame-color, playback, and camera controls.
- Combined Gaussian + mesh rendering with occlusion handling.
- Demo video, functional tests, and final graduation presentation.

## Team Responsibilities

| Gaussian Splatting & application | Optics & glasses design |
| --- | --- |
| GaussianAvatars reproduction | Zemax optical system design |
| FLAME pose extraction and attachment | FOV, eye relief, eyebox, and other optical parameter calculations |
| Glasses mesh preparation, auto-fit, and geometric inspection | Optical layout and performance verification |
| Joint rendering and viewer controls | Optical reference positions and parameter export |
| Demonstration and application testing | Optical-design results and documentation |

## References

- [GaussianAvatars — Official Repository](https://github.com/ShenhanQian/GaussianAvatars)
- [GaussianAvatars — Dataset Download](https://github.com/ShenhanQian/GaussianAvatars/blob/main/doc/download.md)
- [VHAP — Head Tracking](https://github.com/ShenhanQian/VHAP)
- [NeRSemble](https://tobias-kirschstein.github.io/nersemble/)
- [FLAME Head Model](https://flame.is.tue.mpg.de/)

*Research models, datasets, and third-party 3D assets remain subject to their respective licenses.*
