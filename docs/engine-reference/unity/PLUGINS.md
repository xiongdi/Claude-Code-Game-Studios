# Unity 6.3 LTS — 可选包与系统

**最后验证时间：** 2026-02-13

本文档索引 Unity 6.3 LTS 中可用的**可选包和系统**。
这些不是核心引擎的一部分，但常用于特定游戏类型。

---

## 如何使用本指南

**✅ 有详细文档** — 参见 `plugins/` 目录获取全面指南
**🟡 仅有简要概述** — 链接到官方文档，使用 WebSearch 获取详情
**⚠️ 预览版** — 未来版本可能有破坏性变更
**📦 需要安装包** — 通过 Package Manager 安装

---

## 生产就绪包（有详细文档）

### ✅ Cinemachine
- **用途：** 虚拟相机系统（动态相机、过场动画、相机混合）
- **何时使用：** 第三人称游戏、电影摄影、复杂相机行为
- **知识缺口：** Cinemachine 3.0+（Unity 6）相比 2.x 有重大 API 变更
- **状态：** 生产就绪
- **包：** `com.unity.cinemachine`（Package Manager）
- **详细文档：** [plugins/cinemachine.md](plugins/cinemachine.md)
- **官方：** https://docs.unity3d.com/Packages/com.unity.cinemachine@3.0/manual/index.html

---

### ✅ Addressables
- **用途：** 高级资源管理（异步加载、远程内容、内存控制）
- **何时使用：** 大型项目、DLC、远程内容交付
- **知识缺口：** Unity 6 改进，更好的性能
- **状态：** 生产就绪
- **包：** `com.unity.addressables`（Package Manager）
- **详细文档：** [plugins/addressables.md](plugins/addressables.md)
- **官方：** https://docs.unity3d.com/Packages/com.unity.addressables@2.0/manual/index.html

---

### ✅ DOTS / Entities (ECS)
- **用途：** 面向数据的技术栈（用于大规模的高性能 ECS）
- **何时使用：** 拥有数千个实体的游戏、RTS、模拟
- **知识缺口：** Entities 1.3+（Unity 6）已生产就绪，从 0.x 完全重写
- **状态：** 生产就绪（截至 Unity 6.3 LTS）
- **包：** `com.unity.entities`（Package Manager）
- **详细文档：** [plugins/dots-entities.md](plugins/dots-entities.md)
- **官方：** https://docs.unity3d.com/Packages/com.unity.entities@1.3/manual/index.html

---

## 其他生产就绪包（简要概述）

### 🟡 Input System（已涵盖）
- **用途：** 现代输入处理（可重绑定、跨平台）
- **状态：** 生产就绪（Unity 6 中默认）
- **包：** `com.unity.inputsystem`
- **文档：** 参见 [modules/input.md](../modules/input.md)
- **官方：** https://docs.unity3d.com/Packages/com.unity.inputsystem@1.11/manual/index.html

---

### 🟡 UI Toolkit（已涵盖）
- **用途：** 现代运行时 UI（类似 HTML/CSS，高性能）
- **状态：** 生产就绪（Unity 6）
- **包：** 内置
- **文档：** 参见 [modules/ui.md](../modules/ui.md)
- **官方：** https://docs.unity3d.com/Packages/com.unity.ui@2.0/manual/index.html

---

### 🟡 Visual Effect Graph (VFX Graph)
- **用途：** GPU 加速粒子系统（数百万粒子）
- **何时使用：** 大规模 VFX、火焰、烟雾、魔法、爆炸
- **状态：** 生产就绪
- **包：** `com.unity.visualeffectgraph`（仅 URP/HDRP）
- **官方：** https://docs.unity3d.com/Packages/com.unity.visualeffectgraph@17.0/manual/index.html

---

### 🟡 Shader Graph
- **用途：** 可视化 shader 编辑器（基于节点的 shader 创建）
- **何时使用：** 无需 HLSL 编码的自定义 shader
- **状态：** 生产就绪
- **包：** `com.unity.shadergraph`（URP/HDRP）
- **官方：** https://docs.unity3d.com/Packages/com.unity.shadergraph@17.0/manual/index.html

---

### 🟡 Timeline
- **用途：** 电影序列（过场动画、脚本事件）
- **何时使用：** 故事驱动游戏、电影摄影、脚本序列
- **状态：** 生产就绪
- **包：** `com.unity.timeline`（内置）
- **官方：** https://docs.unity3d.com/Packages/com.unity.timeline@1.8/manual/index.html

---

### 🟡 Animation Rigging
- **用途：** 运行时 IK、程序化动画
- **何时使用：** 脚部 IK、瞄准偏移、程序化肢体放置
- **状态：** 生产就绪（Unity 6）
- **包：** `com.unity.animation.rigging`
- **官方：** https://docs.unity3d.com/Packages/com.unity.animation.rigging@1.3/manual/index.html

---

### 🟡 ProBuilder
- **用途：** 编辑器内 3D 建模（关卡原型、灰盒）
- **何时使用：** 快速原型、关卡 blockout
- **状态：** 生产就绪
- **包：** `com.unity.probuilder`
- **官方：** https://docs.unity3d.com/Packages/com.unity.probuilder@6.0/manual/index.html

---

### 🟡 Netcode for GameObjects
- **用途：** 官方 Unity 多人游戏网络
- **何时使用：** 多人游戏（客户端-服务器架构）
- **状态：** 生产就绪
- **包：** `com.unity.netcode.gameobjects`
- **官方：** https://docs-multiplayer.unity3d.com/netcode/current/about/

---

### 🟡 Burst Compiler
- **用途：** 用于 C# Jobs 的 LLVM 编译器（巨大性能提升）
- **何时使用：** 性能关键代码、DOTS、Jobs System
- **状态：** 生产就绪
- **包：** `com.unity.burst`（随 DOTS 自动安装）
- **官方：** https://docs.unity3d.com/Packages/com.unity.burst@1.8/manual/index.html

---

### 🟡 Jobs System
- **用途：** 多线程作业调度（CPU 并行）
- **何时使用：** 性能优化、并行处理
- **状态：** 生产就绪
- **包：** 内置
- **官方：** https://docs.unity3d.com/Manual/JobSystem.html

---

### 🟡 Mathematics
- **用途：** SIMD 数学库（为 Burst 优化）
- **何时使用：** DOTS、高性能数学
- **状态：** 生产就绪
- **包：** `com.unity.mathematics`
- **官方：** https://docs.unity3d.com/Packages/com.unity.mathematics@1.3/manual/index.html

---

### 🟡 ML-Agents（机器学习）
- **用途：** 通过强化学习训练 AI
- **何时使用：** 高级 AI 训练、程序化行为
- **状态：** 生产就绪
- **包：** `com.unity.ml-agents`
- **官方：** https://github.com/Unity-Technologies/ml-agents

---

### 🟡 Recorder
- **用途：** 捕获游戏画面、截图、动画片段
- **何时使用：** 预告片、回放、调试录制
- **状态：** 生产就绪
- **包：** `com.unity.recorder`
- **官方：** https://docs.unity3d.com/Packages/com.unity.recorder@5.0/manual/index.html

---

## 预览/实验性包（谨慎使用）

### ⚠️ Splines
- **用途：** 运行时 spline 创建和编辑
- **何时使用：** 道路、路径、程序化内容
- **状态：** 生产就绪（Unity 6）
- **包：** `com.unity.splines`
- **官方：** https://docs.unity3d.com/Packages/com.unity.splines@2.6/manual/index.html

---

### ⚠️ Muse（AI 助手）
- **用途：** AI 驱动的资源创建（纹理、精灵、动画）
- **状态：** 预览版（Unity 6）
- **包：** `com.unity.muse.*`
- **官方：** https://unity.com/products/muse

---

### ⚠️ Sentis（神经网络推理）
- **用途：** 在 Unity 中运行神经网络（AI 推理）
- **状态：** 预览版
- **包：** `com.unity.sentis`
- **官方：** https://docs.unity3d.com/Packages/com.unity.sentis@2.0/manual/index.html

---

## 已弃用包（新项目避免使用）

### ❌ UGUI (Canvas UI)
- **已弃用：** 仍然支持，但推荐使用 UI Toolkit
- **改用：** UI Toolkit

---

### ❌ Legacy Particle System
- **已弃用：** 使用 Visual Effect Graph (VFX Graph)
- **改用：** VFX Graph

---

### ❌ Legacy Animation
- **已弃用：** 使用 Animator (Mecanim)
- **改用：** Animator Controller

---

## 按需 WebSearch 策略

对于上面未列出的包，当用户询问时使用以下方法：

1. **WebSearch** 最新文档：`"Unity 6.3 [包名]"`
2. 验证包是否：
   - 在截止日期之后（超出 2025 年 5 月训练数据）
   - 预览版 vs 生产就绪
   - 在 Unity 6.3 LTS 中仍然受支持
3. 可选地将发现缓存到 `plugins/[package-name].md` 供将来参考

---

## 快速决策指南

**我需要虚拟相机** → **Cinemachine**
**我需要异步资源加载 / DLC** → **Addressables**
**我需要数千个实体（RTS、模拟）** → **DOTS/Entities**
**我需要现代输入** → **Input System**（参见 modules/input.md）
**我需要 GPU 粒子** → **Visual Effect Graph**
**我需要可视化 shader** → **Shader Graph**
**我需要电影摄影** → **Timeline**
**我需要运行时 IK** → **Animation Rigging**
**我需要关卡原型** → **ProBuilder**
**我需要多人游戏** → **Netcode for GameObjects**

---

**最后更新：** 2026-02-13
**引擎版本：** Unity 6.3 LTS
**LLM 知识截止日期：** 2025 年 5 月
