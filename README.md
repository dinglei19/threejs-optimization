# Three.js Optimization Skill

`$threejs-optimization` 是用于 Codex 的 Three.js 性能优化 Skill。在保留现有画质、业务规则、状态更新顺序和交互结果的前提下，定位瓶颈、实施改动，并用同一场景比较优化前后的表现。

- **运行时**用 Performance trace 查主线程热点，分析 GPU 耗时、draw calls、JS 堆和显存。按实测决定是否合批、实例化，或在主线程、Worker 和 GPU 之间调整计算。
- **GLB / glTF**按目标选择纹理缩放、KTX2 压缩、Meshopt、Draco、减面或 LOD，并验证材质、节点、动画和加载器兼容性。

## 安装到 Codex

先克隆仓库。

```sh
git clone https://github.com/dinglei19/threejs-optimization.git
cd threejs-optimization
```

**供当前用户的所有项目使用**

```sh
mkdir -p "$HOME/.agents/skills"
cp -R skills/threejs-optimization "$HOME/.agents/skills/"
```

**仅供某个项目使用**

在目标项目根目录运行，替换示例中的仓库路径。

```sh
mkdir -p .agents/skills
cp -R /path/to/threejs-optimization/skills/threejs-optimization .agents/skills/
```

两个安装范围任选其一。复制整个 Skill 目录以保留引用文件。如果技能列表里没有出现 `threejs-optimization`，重启 Codex。 [Codex Skill 安装说明](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills)

GLB 转换按需使用项目已有工具或 [glTF Transform CLI](https://gltf-transform.dev/cli)。KTX2 编码需要与 CLI 版本匹配的 [Khronos KTX-Software](https://github.com/KhronosGroup/KTX-Software)。

## 使用

在目标 Three.js 项目的 Codex 对话中用 `$threejs-optimization` 调用。提供场景入口、卡顿的操作、目标设备或 GLB 路径，便于复现和验收。

**优化场景**

> 使用 $threejs-optimization 优化角色房间。进房间后旋转相机时会掉帧。先抓 Performance trace 定位瓶颈，再实施改动。保留现有动画和拾取逻辑，给出同设备的帧间隔 p95 和 draw calls 前后对比。

**优化模型**

> 使用 $threejs-optimization 优化 assets/character.glb。保留骨骼、动画和挂点，生成独立的候选 GLB，并报告文件大小、所需解码器和验证结果。

**只做审计**

> 使用 $threejs-optimization 审计当前场景的 CPU、GPU 和内存开销。只提供证据和按收益排序的优化清单。

## Skill 文件

- [SKILL.md](skills/threejs-optimization/SKILL.md) 定义工作流程和验收条件。
- [运行时指南](skills/threejs-optimization/references/runtime.md) 说明性能分析、CPU/GPU 分工和资源释放。
- [GLB 指南](skills/threejs-optimization/references/glb.md) 说明模型检查、转换命令和接入验证。
