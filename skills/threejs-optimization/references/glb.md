# GLB / glTF 模型优化

## 检查输入与使用契约

- 找到模型在程序里的实际入口。搜索 `getObjectByName`、动画绑定与外置动画、骨骼挂点、材质替换、拾取、碰撞和自定义属性。记录依赖的节点名、层级与变换、mesh/primitive、skin、morph、clip、UV 与 `extras`。角色与可交互模型优先试结构变更较少的方案。
- 优先用项目固定版本的工具。记录 `gltf-transform --version`，用 `gltf-transform inspect input.glb` 查文件扩展、场景、网格、纹理与动画，确认原始文件能被目标加载器读取。离线 primitive 数和 GPU 内存估算需要结合实例数、渲染 pass 与实际 GPU 格式解读。
- 仓库已有 GLB 脚本时，先看帮助和真实实现，优先复用符合约束的步骤。例如 `vue-three-web` 的 `glb-optimize/glb-optimize.sh` 会尝试多个转换，部分失败时沿用前一步产物，`--texture-only` 也仍会去重或清理资源。需要原层级、命名与 mesh 映射时，用单项纹理操作，并检查转换前后的契约。

## 按指标选择转换

| 目标 | 转换 | 核对 |
| --- | --- | --- |
| 降低传输量或解析时间 | 检查重复或无用数据。可分别试 Meshopt 或 Draco，测文件体积、实际 HTTP 传输量及解码耗时。Meshopt 可处理几何、morph 和动画数据，比较生产环境启用 Brotli 或 gzip 时的结果 | 压缩与量化后验证形变、蒙皮、UV 和动画。几何压缩可保留三角形和 draw calls 数量，帧率需另测 |
| 降低顶点成本 | 按屏幕可见误差试 weld、simplify 或独立 LOD，逐档检查近景轮廓。量化可减少顶点存储，检查位置、UV 与权重的精度 | 法线、切线、UV 缝、变形、边界与小部件 |
| 降低 draw calls | 实例化同网格的重复物体，或在同材质、同更新方式且空间相近时合并静态 primitive | 运行时 draw calls、裁剪粒度、独立交互、对象变换和动画绑定 |
| 降低显存和上传成本 | 先按实际显示分辨率与用途缩小纹理，再试 KTX2/Basis。色彩图可试 ETC1S，法线、粗糙度金属度等线性数据优先试 UASTC | 目标设备的实际 GPU 转码格式、色彩空间、mip、法线细节及透明边缘。PNG/JPEG/WebP 体积下降主要反映传输收益 |
| 降低动画包和运行开销 | 按独立目标处理关键帧去冗余、数值压缩或更新频率，保留业务使用的 clip 与轨道 | 时长、循环衔接、根运动、blend、骨骼名与跨文件绑定 |

`prune`、`dedup`、`join`、`instance` 与通用 `optimize` 的影响按安装版本和选项确认。清理前检查应用运行时才引用的空节点、自定义属性、UV 与材质。涉及多个步骤时分阶段输出，先做结构与纹理处理，最后编码几何并检查最终扩展。遇到未知扩展时先确认工具能保留其数据。

## 候选命令

以下为 glTF Transform 4.x 的单项示例。根据目标选步骤，在独立目录生成新文件。对照安装版本的 `help <command>`，把示例参数换成符合目标的质量预算。

```sh
gltf-transform inspect input.glb

# 减面候选
gltf-transform weld input.glb welded.glb
gltf-transform simplify welded.glb lod.glb --ratio 0.7 --error 0.001

# 几何编码候选，分开比较
gltf-transform meshopt input.glb meshopt.glb --level medium
gltf-transform draco input.glb draco.glb --method edgebreaker

# 纹理候选，需要与 CLI 版本匹配的 Khronos KTX-Software
gltf-transform resize input.glb resized.glb --width 1024 --height 1024
gltf-transform uastc resized.glb data-textures.glb --slots '{normalTexture,occlusionTexture,metallicRoughnessTexture}' --level 2 --jobs 2
gltf-transform etc1s data-textures.glb textures.glb --slots '{baseColorTexture,emissiveTexture}' --quality 128 --jobs 2

gltf-transform validate textures.glb
gltf-transform inspect textures.glb
```

纹理 slot 示例覆盖常见 PBR 贴图。混用同一图像的色彩与数据贴图、扩展材质贴图、已压缩纹理及带 alpha 的贴图需逐项检查。对缩放、量化或简化造成的质量变化使用实际相机和光照做 A/B 对照。

## 接入与验收

1. 重新检查候选的 `extensionsUsed`、`extensionsRequired` 和几何、纹理、动画统计。运行 glTF Validator 并比较基线与候选的错误和警告。对工具不支持的扩展使用实际加载与专用工具补验。可提取 KTX2 图像缓冲区，用 `ktx validate --gltf-basisu` 检查纹理是否符合扩展要求。
2. 对照应用的 Three.js 版本和后端复用加载器。Draco 通过 `GLTFLoader.setDRACOLoader` 注册，Meshopt 通过 `setMeshoptDecoder` 注册，KTX2 通过 `setKTX2Loader` 注册并在加载前完成对应 renderer 的支持探测。检查解码器资源的版本、路径、生产部署、Worker 与目标设备支持。若目标端无法加载，保留可用的资产路径。
3. 在业务入口运行候选，验证模型大小和朝向、动画与附件、交互、裁剪以及纹理质量。加载对比按相同网络和缓存条件采集，并计入解码器资源与首次初始化成本。需要判断帧率或显存收益时，再跑同场景的运行时对照。

## 文档

- [glTF Transform CLI](https://gltf-transform.dev/cli)
- [Khronos glTF Validator](https://github.com/KhronosGroup/glTF-Validator)
- [Khronos KTX2 / Basis 扩展规范](https://github.com/KhronosGroup/glTF/blob/main/extensions/2.0/Khronos/KHR_texture_basisu/README.md)
- [Three.js GLTFLoader](https://threejs.org/docs/pages/GLTFLoader.html)
- [Three.js KTX2Loader](https://threejs.org/docs/pages/KTX2Loader.html)
