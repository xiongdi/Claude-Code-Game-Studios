# Unity Engine — 版本参考

| Field | Value |
|-------|-------|
| **Engine Version** | Unity 6.3 LTS |
| **Release Date** | 2025 年 12 月 |
| **Project Pinned** | 2026-02-13 |
| **Last Docs Verified** | 2026-02-13 |
| **LLM Knowledge Cutoff** | 2025 年 5 月 |

## 知识缺口警告

LLM 的训练数据大约只覆盖到 Unity ~2022 LTS（2022.3）。整个 Unity 6 系列版本（原 Unity 2023 Tech Stream）引入了重大变更，模型对此并不了解。在建议 Unity API 调用之前，务必交叉参考本目录。

## 截止时间后的版本时间线

| Version | Release | Risk Level | Key Theme |
|---------|---------|------------|-----------|
| 6.0 | 2024 年 10 月 | HIGH | Unity 6 品牌重塑、新渲染特性、Entities 1.3、DOTS 改进 |
| 6.1 | 2024 年 11 月 | MEDIUM | Bug 修复、稳定性改进 |
| 6.2 | 2024 年 12 月 | MEDIUM | 性能优化、新输入系统改进 |
| 6.3 LTS | 2025 年 12 月 | HIGH | 自 6.0 以来首个 LTS、生产就绪的 DOTS、增强的图形特性 |

## 从 2022 LTS 到 Unity 6.3 LTS 的主要变更

### 破坏性变更
- **Entities/DOTS**：Entities 1.0+ 中的 API 全面革新，ECS 模式完全重新设计
- **Input System**：旧版 Input Manager 已弃用，新 Input System 为默认
- **Rendering**：URP/HDRP 重大升级，SRP Batcher 改进
- **Addressables**：资产管理流程变更
- **Scripting**：C# 9 支持、新的 API 模式

### 新特性（截止时间后）
- **DOTS**：生产就绪的 Entity Component System（Entities 1.3+）
- **Graphics**：增强的 URP/HDRP 管线、GPU Resident Drawer
- **Multiplayer**：Netcode for GameObjects 改进
- **UI Toolkit**：生产就绪的运行时 UI（替代新项目的 UGUI）
- **Async Asset Loading**：Addressables 性能改进
- **Web**：WebGPU 支持

### 已弃用系统
- **Legacy Input Manager**：使用新的 Input System 包
- **Legacy Particle System**：使用 Visual Effect Graph
- **UGUI**：仍受支持，但新项目推荐使用 UI Toolkit
- **旧版 ECS（GameObjectEntity）**：被现代 DOTS/Entities 替代

## 已验证来源

- 官方文档：https://docs.unity3d.com/6000.0/Documentation/Manual/index.html
- Unity 6 发布：https://unity.com/releases/unity-6
- Unity 6.3 LTS 公告：https://unity.com/blog/unity-6-3-lts-is-now-available
- 迁移指南：https://docs.unity3d.com/6000.0/Documentation/Manual/upgrade-guides.html
- Unity 6 支持：https://unity.com/releases/unity-6/support
- C# API 参考：https://docs.unity3d.com/6000.0/Documentation/ScriptReference/index.html
