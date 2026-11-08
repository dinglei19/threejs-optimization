# Three.js Optimization Skill

[简体中文](README.md) · English

`$threejs-optimization` is a Codex skill for improving Three.js performance. It identifies bottlenecks, implements targeted changes, and measures the same scene before and after, preserving visual quality, business rules, state update order, and interaction behavior.

- **Runtime** uses a Performance trace to find main-thread hotspots and examines GPU time, draw calls, the JavaScript heap, and GPU memory. Measurements guide decisions about mesh merging, instancing, and moving work between the main thread, workers, and the GPU.
- **GLB and glTF** covers texture resizing, KTX2 compression, Meshopt, Draco, mesh simplification, and level of detail. Choose techniques based on your performance goals and verify materials, nodes, animations, and loader compatibility.

## Install in Codex

Clone the repository.

```sh
git clone https://github.com/dinglei19/threejs-optimization.git
cd threejs-optimization
```

**Install for all your projects**

```sh
mkdir -p "$HOME/.agents/skills"
cp -R skills/threejs-optimization "$HOME/.agents/skills/"
```

**Install for one project**

Run these commands from the target project's root and substitute the path to your clone.

```sh
mkdir -p .agents/skills
cp -R /path/to/threejs-optimization/skills/threejs-optimization .agents/skills/
```

Choose one installation scope. Copy the entire skill directory so Codex can access its reference files. Restart Codex if `threejs-optimization` is missing from the skill list. [Codex skill installation docs](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills)

For GLB conversion, use your project's existing tools or the [glTF Transform CLI](https://gltf-transform.dev/cli). KTX2 encoding requires a compatible version of [Khronos KTX-Software](https://github.com/KhronosGroup/KTX-Software).

## Usage

Invoke `$threejs-optimization` in Codex from the target Three.js project. Include the scene entry point, the action that triggers the slowdown, the target device, or the path to the GLB so Codex can reproduce the issue and verify the result.

**Optimize a scene**

> Use $threejs-optimization to improve the character room scene. It drops frames when I rotate the camera after entering the room. Capture a Performance trace to identify the bottleneck, then apply a targeted fix. Preserve the existing animation and object picking behavior, and report p95 frame intervals and draw calls before and after on the same device.

**Optimize a model**

> Use $threejs-optimization to optimize assets/character.glb. Preserve the skeleton, animations, and attachment points. Save a separate GLB candidate and report its file size, decoder requirements, and validation results.

**Audit performance**

> Use $threejs-optimization to audit CPU, GPU, and memory costs in the current scene. Return the evidence and rank potential optimizations by expected impact.

## Skill files

- [SKILL.md](skills/threejs-optimization/SKILL.md) defines the workflow and acceptance criteria.
- [Runtime guide](skills/threejs-optimization/references/runtime.md) covers profiling, CPU and GPU workload placement, and resource disposal.
- [GLB guide](skills/threejs-optimization/references/glb.md) covers model inspection, conversion commands, and integration checks.
