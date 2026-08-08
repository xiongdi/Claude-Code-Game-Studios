# Hook: pre-commit-code-quality

## Trigger（触发）

在任何修改 `src/` 中文件的提交之前运行。

## Purpose（目的）

在代码进入版本控制之前强制执行编码标准。捕获风格违规、缺失的文档注释、过于复杂的方法，以及本应为数据驱动的硬编码值。

## Implementation（实现）

```bash
#!/bin/bash
# Pre-commit hook: Code quality checks
# Adapt the specific checks to your language and tooling

CODE_FILES=$(git diff --cached --name-only --diff-filter=ACM | grep -E '^src/')

EXIT_CODE=0

if [ -n "$CODE_FILES" ]; then
    for file in $CODE_FILES; do
        # Check for hardcoded magic numbers in gameplay code
        if [[ "$file" == src/gameplay/* ]]; then
            # Look for numeric literals that are likely balance values
            # Adjust the pattern for your language
            if grep -nE '(damage|health|speed|rate|chance|cost|duration)[[:space:]]*[:=][[:space:]]*[0-9]+' "$file"; then
                echo "WARNING: $file may contain hardcoded gameplay values. Use data files."
                # Warning only, not blocking
            fi
        fi

        # Check for TODO/FIXME without assignee
        if grep -nE '(TODO|FIXME|HACK)[^(]' "$file"; then
            echo "WARNING: $file has TODO/FIXME without owner tag. Use TODO(name) format."
        fi

        # Run language-specific linter (uncomment appropriate line)
        # For GDScript: gdlint "$file" || EXIT_CODE=1
        # For C#: dotnet format --check "$file" || EXIT_CODE=1
        # For C++: clang-format --dry-run -Werror "$file" || EXIT_CODE=1
    done

    # Run unit tests for modified systems
    # Uncomment and adapt for your test framework
    # python -m pytest tests/unit/ -x --quiet || EXIT_CODE=1
fi

exit $EXIT_CODE
```

## Agent Integration（Agent 集成）

当此 hook 失败时：
1. 风格违规：用你的格式化工具自动修复，或调用 `lead-programmer`
2. 硬编码值：调用 `gameplay-programmer` 将值外部化
3. 测试失败：调用 `qa-tester` 诊断，调用 `gameplay-programmer` 修复
