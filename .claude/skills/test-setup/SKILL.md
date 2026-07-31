---
name: test-setup
description: "Scaffold the test framework and CI/CD pipeline for the project's engine. Creates the tests/ directory structure, engine-specific test runner configuration, and GitHub Actions workflow. Run once during Technical Setup phase before the first sprint begins."
argument-hint: "[force]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Bash, Write
model: sonnet
---

# Test Setup

此 skill 为项目搭建自动化测试基础设施。它检测配置的引擎，生成适当的测试运行器配置，创建标准目录布局，并连接 CI/CD 以便在每次推送时运行测试。

在 Technical Setup 阶段运行一次，在任何实现开始之前。在 sprint 开始时安装测试框架需要 30 分钟。在 sprint 四时安装测试框架需要 3 个 sprint。

**输出：** `tests/` 目录结构 + `.github/workflows/tests.yml`

---

## 阶段 1：检测引擎和现有状态

1. **读取引擎配置**：
   - 读取 `.claude/docs/technical-preferences.md` 并提取 `Engine:` 值。
   - 如果引擎未配置（`[TO BE CONFIGURED]`），停止：
     "Engine not configured. Run `/setup-engine` first, then re-run `/test-setup`."

2. **检查现有测试基础设施**：
   - Glob `tests/` — 目录是否存在？
   - Glob `tests/unit/` 和 `tests/integration/` — 子目录是否存在？
   - Glob `.github/workflows/` — CI 工作流文件是否存在？
   - Glob `tests/gdunit4_runner.gd`（Godot）或 `tests/EditMode/`（Unity）或 `Source/Tests/`（Unreal）查找引擎特定产物。

3. **报告发现**：
   - "Engine: [engine]. Test directory: [found / not found]. CI workflow: [found / not found]."
   - 如果一切已存在且未传入 `force` 参数：
     "Test infrastructure appears to be in place. Re-run with `/test-setup force` to regenerate. Proceeding will not overwrite existing test files."

如果传入了 `force` 参数，跳过"已存在"的早期退出并继续 — 但仍然不要覆盖给定路径上已存在的文件。只创建缺失的文件。

---

## 阶段 2：展示计划

根据检测到的引擎和现有状态，展示计划：

```
## Test Setup Plan — [Engine]

I will create the following (skipping any that already exist):

tests/
  unit/           — Isolated unit tests for formulas, state, and logic
  integration/    — Cross-system tests and save/load round-trips
  smoke/          — Critical path test list (15-minute manual gate)
  evidence/       — Screenshot and manual test sign-off records
  README.md       — Test framework documentation

[Engine-specific files — see per-engine details below]

.github/workflows/tests.yml  — CI: run tests on every push to main

Estimated time: ~5 minutes to create all files.
```

询问："May I create these files? I will not overwrite any test files that already exist at these paths."

未经批准不要继续。

---

## 阶段 3：创建目录结构

批准后，创建以下文件：

### `tests/README.md`

```markdown
# Test Infrastructure

**Engine**: [engine name + version]
**Test Framework**: [GdUnit4 | Unity Test Framework | UE Automation]
**CI**: `.github/workflows/tests.yml`
**Setup date**: [date]

## Directory Layout

```
tests/
  unit/           # Isolated unit tests (formulas, state machines, logic)
  integration/    # Cross-system and save/load tests
  smoke/          # Critical path test list for /smoke-check gate
  evidence/       # Screenshot logs and manual test sign-off records
```

## Running Tests

[Engine-specific command — see below]

## Test Naming

- **Files**: `[system]_[feature]_test.[ext]`
- **Functions**: `test_[scenario]_[expected]`
- **Example**: `combat_damage_test.gd` → `test_base_attack_returns_expected_damage()`

## Story Type → Test Evidence

| Story Type | Required Evidence | Location |
|---|---|---|
| Logic | Automated unit test — must pass | `tests/unit/[system]/` |
| Integration | Integration test OR playtest doc | `tests/integration/[system]/` |
| Visual/Feel | Screenshot + lead sign-off | `tests/evidence/` |
| UI | Manual walkthrough OR interaction test | `tests/evidence/` |
| Config/Data | Smoke check pass | `production/qa/smoke-*.md` |

## CI

Tests run automatically on every push to `main` and on every pull request.
A failed test suite blocks merging.
```
```

### 引擎特定文件

#### Godot 4（`Engine: Godot`）

创建 `tests/gdunit4_runner.gd`：

```gdscript
# GdUnit4 test runner — invoked by CI and /smoke-check
# Usage: godot --headless --script tests/gdunit4_runner.gd
extends SceneTree

func _init() -> void:
    var runner := load("res://addons/gdunit4/GdUnitRunner.gd")
    if runner == null:
        push_error("GdUnit4 not found. Install via AssetLib or addons/.")
        quit(1)
        return
    var instance = runner.new()
    instance.run_tests()
    quit(0)
```

创建 `tests/unit/.gdignore_placeholder`，内容为：
`# Unit tests go here — one subdirectory per system (e.g., tests/unit/combat/)`

创建 `tests/integration/.gdignore_placeholder`，内容为：
`# Integration tests go here — one subdirectory per system`

在 README 中注明：**安装 GdUnit4**
```
1. Open Godot → AssetLib → search "GdUnit4" → Download & Install
2. Enable the plugin: Project → Project Settings → Plugins → GdUnit4 ✓
3. Restart the editor
4. Verify: res://addons/gdunit4/ exists
```

#### Unity（`Engine: Unity`）

创建 `tests/EditMode/` 占位文件 `tests/EditMode/README.md`：
```markdown
# Edit Mode Tests
Unit tests that run without entering Play Mode.
Use for pure logic: formulas, state machines, data validation.
Assembly definition required: `tests/EditMode/EditModeTests.asmdef`
```

创建 `tests/PlayMode/README.md`：
```markdown
# Play Mode Tests
Integration tests that run in a real game scene.
Use for cross-system interactions, physics, and coroutines.
Assembly definition required: `tests/PlayMode/PlayModeTests.asmdef`
```

在 README 中注明：**启用 Unity Test Framework**
```
Window → General → Test Runner
(Unity Test Framework is included by default in Unity 2019+)
```

#### Unreal Engine（`Engine: Unreal` 或 `Engine: UE5`）

创建 `Source/Tests/README.md`：
```markdown
# Unreal Automation Tests
Tests use the UE Automation Testing Framework.
Run via: Session Frontend → Automation → select "MyGame." tests
Or headlessly: UnrealEditor -nullrhi -ExecCmds="Automation RunTests MyGame.; Quit"

Test class naming: F[SystemName]Test
Test category naming: "MyGame.[System].[Feature]"
```

---

## 阶段 4：创建 CI/CD 工作流

### Godot 4

创建 `.github/workflows/tests.yml`：

```yaml
name: Automated Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    name: Run GdUnit4 Tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          lfs: true

      - name: Run GdUnit4 Tests
        uses: MikeSchulze/gdUnit4-action@v1
        with:
          godot-version: '[VERSION FROM docs/engine-reference/godot/VERSION.md]'
          paths: |
            tests/unit
            tests/integration
          report-name: test-results

      - name: Upload Test Results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: reports/
```

### Unity

创建 `.github/workflows/tests.yml`：

```yaml
name: Automated Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    name: Run Unity Tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          lfs: true

      - name: Run Edit Mode Tests
        uses: game-ci/unity-test-runner@v4
        env:
          UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }}
        with:
          testMode: editmode
          artifactsPath: test-results/editmode

      - name: Run Play Mode Tests
        uses: game-ci/unity-test-runner@v4
        env:
          UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }}
        with:
          testMode: playmode
          artifactsPath: test-results/playmode

      - name: Upload Test Results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: test-results/
```

注意：Unity CI 需要一个 `UNITY_LICENSE` secret。在首次 CI 运行之前添加到 GitHub 仓库 secrets。

### Unreal Engine

创建 `.github/workflows/tests.yml`：

```yaml
name: Automated Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    name: Run UE Automation Tests
    runs-on: self-hosted  # UE requires a local runner with the editor installed

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          lfs: true

      - name: Run Automation Tests
        run: |
          "$UE_EDITOR_PATH" "${{ github.workspace }}/[ProjectName].uproject" \
            -nullrhi -nosound \
            -ExecCmds="Automation RunTests MyGame.; Quit" \
            -log -unattended
        shell: bash

      - name: Upload Logs
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-logs
          path: Saved/Logs/
```

注意：UE CI 需要一个安装了 Unreal Editor 的自托管运行器。在运行器上设置 `UE_EDITOR_PATH` 环境变量。

---

## 阶段 5：创建烟雾测试种子

创建 `tests/smoke/critical-paths.md`：

```markdown
# Smoke Test: Critical Paths

**Purpose**: Run these 10-15 checks in under 15 minutes before any QA hand-off.
**Run via**: `/smoke-check` (which reads this file)
**Update**: Add new entries when new core systems are implemented.

## Core Stability (always run)

1. Game launches to main menu without crash
2. New game / session can be started from the main menu
3. Main menu responds to all inputs without freezing

## Core Mechanic (update per sprint)

<!-- Add the primary mechanic for each sprint here as it is implemented -->
<!-- Example: "Player can move, jump, and the camera follows correctly" -->
4. [Primary mechanic — update when first core system is implemented]

## Data Integrity

5. Save game completes without error (once save system is implemented)
6. Load game restores correct state (once load system is implemented)

## Performance

7. No visible frame rate drops on target hardware (60fps target)
8. No memory growth over 5 minutes of play (once core loop is implemented)
```

---

## 阶段 6：搭建后总结

写入所有文件后，报告：

```
Test infrastructure created for [engine].

Files created:
- tests/README.md
- tests/unit/ (directory)
- tests/integration/ (directory)
- tests/smoke/critical-paths.md
- tests/evidence/ (directory)
[engine-specific files]
- .github/workflows/tests.yml

Next steps:
1. [Engine-specific install step, e.g., "Install GdUnit4 via AssetLib"]
2. Write your first test: create tests/unit/[first-system]/[system]_test.[ext]
3. Run `/qa-plan sprint` before your first sprint to classify stories and set
   test evidence requirements
4. `/smoke-check` before every QA hand-off

Gate note: /gate-check Technical Setup → Pre-Production now requires:
- tests/ directory with unit/ and integration/ subdirectories
- .github/workflows/tests.yml
- At least one example test file
Run /test-setup and write one example test before advancing.

Verdict: **COMPLETE** — test framework scaffolded and CI/CD wired up.
```

---

## 协作协议

- **永远不要覆盖现有测试文件** — 只创建缺失的文件。如果测试运行器文件存在，保持原样。
- **创建文件前始终询问** — 阶段 2 需要明确批准。
- **引擎检测是不可协商的** — 如果引擎未配置，停止并重定向到 `/setup-engine`。不要猜测。
- **`force` 标志跳过"已存在"早期退出但永远不覆盖。** 它的意思是"即使目录已存在也创建任何缺失的文件。"
- 对于 Unity CI，注意 `UNITY_LICENSE` secret 必须手动配置。不要尝试自动化许可证管理。
