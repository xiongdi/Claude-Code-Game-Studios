# Unreal Engine 5.7 — PCG (Procedural Content Generation)

**最后验证时间:** 2026-02-13
**状态:** 生产就绪（截至 UE 5.7）
**插件:** `PCG`（内置，在 Plugins 中启用）

---

## 概述

**程序化内容生成 (PCG)** 是 Unreal 的基于节点的框架，用于大规模生成程序化内容。
它专为填充大型开放世界而设计，包括 foliage、岩石、道具、建筑和其他环境细节。

**将 PCG 用于:**
- 程序化 foliage 放置（树木、草地、岩石）
- 基于生物群系的环境生成
- 道路/路径生成
- 建筑/结构放置
- 世界细节填充（道具、杂物）

**不要将 PCG 用于:**
- 游戏逻辑（使用 Blueprints/C++）
- 一次性手动放置（使用编辑器工具）

**⚠️ 注意:** PCG 在 UE 5.0-5.6 中为实验性功能，在 UE 5.7 中变为生产就绪。

---

## 核心概念

### 1. **PCG Graph**
- 基于节点的图（类似于 Material Editor）
- 定义生成规则

### 2. **PCG Component**
- 放置在关卡中，执行 PCG Graph
- 在定义的体积内生成内容

### 3. **PCG Data**
- Point data（位置、旋转、缩放）
- Spline data（路径、道路、河流）
- Volume data（密度、生物群系遮罩）

### 4. **Nodes**
- **Samplers**: 生成点（Grid、Poisson、Surface）
- **Filters**: 根据规则移除点（Density、Tag、Bounds）
- **Modifiers**: 变换点（Offset、Rotate、Scale）
- **Spawners**: 在点处实例化 meshes/actors

---

## 设置

### 1. 启用插件

`Edit > Plugins > PCG > Enabled > Restart`

### 2. 创建 PCG Volume

1. Place Actors > Volumes > PCG Volume
2. 将 volume 缩放到所需的生成区域

### 3. 创建 PCG Graph

1. Content Browser > PCG > PCG Graph
2. 打开 PCG Graph Editor

---

## 基本工作流

### 示例: 森林生成

#### 1. 创建 PCG Graph

**节点设置:**
```
Input (Volume)
  ↓
Surface Sampler (sample volume surface, points per m²: 0.5)
  ↓
Density Filter (use texture mask or noise)
  ↓
Static Mesh Spawner (tree meshes)
  ↓
Output
```

#### 2. 将 Graph 分配给 Volume

1. 选择 PCG Volume
2. Details Panel > PCG Component > Graph = 你的 PCG Graph
3. 点击 "Generate" 按钮

---

## 关键节点类型

### Samplers（点生成）

#### Grid Sampler
- 规则网格的点
- 配置:
  - **Grid Size**: 点之间的距离
  - **Offset**: 每个点的随机偏移

#### Poisson Disk Sampler
- 具有最小距离的随机点
- 配置:
  - **Points Per m²**: 密度
  - **Min Distance**: 点之间的间距

#### Surface Sampler
- 在 mesh 表面或 landscape 上的点
- 配置:
  - **Points Per m²**: 密度
  - **Surface Only**: 仅表面，不是体积

---

### Filters（点移除）

#### Density Filter
- 基于密度值移除点
- 输入: 纹理或噪声
- 用于: 生物群系遮罩、空地、路径

#### Tag Filter
- 按 tag 过滤点
- 用于: 条件生成

#### Bounds Filter
- 仅保留边界内的点
- 用于: 将生成限制在特定区域

---

### Modifiers（点变换）

#### Rotate
- 随机化点旋转
- 配置:
  - **Min/Max Rotation**: 每个轴的旋转范围

#### Scale
- 随机化点缩放
- 配置:
  - **Min/Max Scale**: 缩放范围

#### Project to Ground
- 将点吸附到 landscape 表面

---

### Spawners（Mesh/Actor 实例化）

#### Static Mesh Spawner
- 在点处生成静态 meshes
- 配置:
  - **Mesh List**: mesh 数组（随机选择）
  - **Culling Distance**: LOD/culling 设置

#### Actor Spawner
- 在点处生成 Blueprint actors
- 用于: Gameplay actors、可交互对象

---

## 数据源

### Landscape
- 将 landscape 作为采样输入
- 自动投影到 landscape 高度

### Splines
- 沿 spline 生成内容（道路、河流、路径）
- 示例: 沿路径的树木

### Textures
- 将纹理用作密度遮罩
- 绘制生物群系、空地、区域

---

## 生物群系示例（混合森林）

### Graph 设置

```
Input (Landscape)
  ↓
Surface Sampler (density: 1.0)
  ↓
┌─────────────────┬─────────────────┐
│ Tree Biome      │ Rock Biome      │
│ (density > 0.5) │ (density < 0.5) │
├─────────────────┼─────────────────┤
│ Tree Spawner    │ Rock Spawner    │
└─────────────────┴─────────────────┘
  ↓
Merge
  ↓
Output
```

---

## 基于 Spline 的生成（带树木的道路）

### 1. 创建 PCG Graph

```
Spline Input
  ↓
Spline Sampler (sample along spline)
  ↓
Offset (offset from spline path)
  ↓
Tree Spawner
  ↓
Output
```

### 2. 向 PCG Volume 添加 Spline Component

1. PCG Volume > Add Component > Spline
2. 绘制 spline 路径
3. PCG Graph 读取 spline 数据

---

## 运行时生成

### 从 C++ 触发生成

```cpp
#include "PCGComponent.h"

UPCGComponent* PCGComp = /* Get PCG Component */;
PCGComp->Generate(); // Execute PCG graph
```

### 流式生成（大型世界）

- PCG 自动随 World Partition 流式加载
- 仅在加载的单元格中生成内容

---

## 性能

### 优化技巧

- 在生成的 meshes 上使用 **culling distance**（LOD）
- 限制 **density**（更少的点 = 更好的性能）
- 对重复的 meshes 使用 **Hierarchical Instanced Static Meshes (HISM)**
- 为大型世界启用 **streaming**

### 调试性能

```cpp
// Console commands:
// pcg.graph.debug 1 - Show PCG debug info
// stat pcg - Show PCG performance stats
```

---

## 常见模式

### 带空地的森林

```
Surface Sampler
  ↓
Density Filter (noise texture with clearings)
  ↓
Tree Spawner (pine, oak, birch)
```

---

### 陡坡上的岩石

```
Landscape Input
  ↓
Surface Sampler
  ↓
Slope Filter (angle > 30°)
  ↓
Rock Spawner
```

---

### 沿道路的道具

```
Spline Input (road spline)
  ↓
Spline Sampler
  ↓
Offset (side of road)
  ↓
Street Light Spawner
```

---

## 调试

### PCG 调试可视化

```cpp
// Console commands:
// pcg.debug.display 1 - Show points and generation bounds
// pcg.debug.colormode points - Color-code points
```

### Graph 调试

- PCG Graph Editor > Debug > Show Debug Points
- 可视化图中每个节点处的点

---

## 从 UE 5.6（实验性）迁移到 5.7（生产版）

### API 变更

```cpp
// ❌ OLD (5.6 experimental API):
// Some nodes renamed, API unstable

// ✅ NEW (5.7 production API):
// Stable node types, documented API
```

**迁移:** 使用稳定的 5.7 节点重建 PCG graphs。彻底测试。

---

## 限制

- **不用于游戏逻辑**: 使用 Blueprints/C++ 处理游戏规则
- **大型图可能很慢**: 使用 filters 和密度降低进行优化
- **运行时生成开销**: 尽可能预生成

---
## 来源
- https://docs.unrealengine.com/5.7/en-US/procedural-content-generation-in-unreal-engine/
- https://docs.unrealengine.com/5.7/en-US/pcg-quick-start-in-unreal-engine/
- UE 5.7 Release Notes（PCG 生产就绪公告）
