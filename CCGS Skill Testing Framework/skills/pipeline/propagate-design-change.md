# Skill Test Spec: /propagate-design-change

## Skill Summary

`/propagate-design-change` 处理 GDD 修订级联。当 GDD 更新时，该 skill 追踪所有引用它的下游产物：ADR、TR-registry 条目、story 和 epic。它生成一份结构化的影响报告，展示需要更改什么以及为什么。该 skill 不会自动应用更改 —— 它为每个受影响的产物提出编辑建议，并在进行任何修改之前按产物询问 "May I write"。

该 skill 在分析期间是只读的，在更新阶段按产物进行写入门控。它没有 director gate —— 分析本身是机械追踪，不是创意审查。

---

## Static Assertions (Structural)

Verified automatically by `/skill-test static` — no fixture needed.

- [ ] Has required frontmatter fields: `name`, `description`, `argument-hint`, `user-invocable`, `allowed-tools`
- [ ] Has ≥2 phase headings
- [ ] Contains verdict keywords: COMPLETE, BLOCKED, NO IMPACT
- [ ] Contains "May I write" collaborative protocol language (per-artifact approval)
- [ ] Has a next-step handoff at the end
- [ ] Documents that changes are proposed, not applied automatically

---

## Director Gate Checks

No director gates — this skill spawns no director gate agents during analysis.
The impact report is a mechanical tracing operation; no creative or technical
director review is required at the analysis stage.

---

## Test Cases

### Case 1: Happy Path — GDD revision affects 2 stories and 1 epic

**Fixture:**
- `design/gdd/[system].md` exists and has been recently revised (git diff shows changes)
- `production/epics/[layer]/EPIC-[system].md` references this GDD
- 2 story files reference TR-IDs from this GDD
- The changed GDD section affects the acceptance criteria of both stories

**Input:** `/propagate-design-change design/gdd/[system].md`

**Expected behavior:**
1. Skill reads the revised GDD and identifies what changed (git diff or content comparison)
2. Skill scans ADRs, TR-registry, epics, and stories for references to this GDD
3. Skill produces an impact report: 1 epic affected, 2 stories affected
4. Skill shows the proposed change for each artifact
5. For each artifact: asks "May I update [filepath]?" separately
6. Applies changes only after per-artifact approval

**Assertions:**
- [ ] Impact report identifies all 3 affected artifacts (1 epic + 2 stories)
- [ ] Each affected artifact's proposed change is shown before asking to write
- [ ] "May I write" is asked per artifact (not once for all artifacts)
- [ ] Skill does NOT apply any changes without per-artifact approval
- [ ] Verdict is COMPLETE after all approved changes are applied

---

### Case 2: No Impact — Changed GDD has no downstream references

**Fixture:**
- `design/gdd/[system].md` exists and has been revised
- No ADRs, stories, or epics reference this GDD's TR-IDs or GDD path

**Input:** `/propagate-design-change design/gdd/[system].md`

**Expected behavior:**
1. Skill reads the revised GDD
2. Skill scans all ADRs, stories, and epics for references
3. No references found
4. Skill outputs: "No downstream impact found for [system].md — no artifacts reference this GDD."
5. No write operations are performed

**Assertions:**
- [ ] Skill outputs the "No downstream impact found" message
- [ ] Verdict is NO IMPACT
- [ ] No "May I write" asks are issued (nothing to update)
- [ ] Skill does NOT error or crash when no references are found

---

### Case 3: In-Progress Story Warning — Referenced story is currently being developed

**Fixture:**
- A story referencing this GDD has `Status: In Progress`
- The developer has already started implementing this story

**Input:** `/propagate-design-change design/gdd/[system].md`

**Expected behavior:**
1. Skill identifies the In Progress story as an affected artifact
2. Skill outputs an elevated warning: "CAUTION: [story-file] is currently In Progress — a developer may be working on this. Coordinate before updating."
3. The warning appears in the impact report before the "May I write" ask for that story
4. User can still approve or skip the update for that story

**Assertions:**
- [ ] In Progress story is flagged with an elevated warning (distinct from regular affected-artifact entries)
- [ ] Warning appears before the "May I write" ask for that story
- [ ] Skill still offers to update the story — the warning does not block the option
- [ ] Other (non-In-Progress) artifacts are not affected by this warning

---

### Case 4: Edge Case — No argument provided

**Fixture:**
- Multiple GDDs exist in `design/gdd/`

**Input:** `/propagate-design-change` (no argument)

**Expected behavior:**
1. Skill detects no argument is provided
2. Skill outputs a usage error: "No GDD specified. Usage: /propagate-design-change design/gdd/[system].md"
3. Skill lists recently modified GDDs as suggestions (git log)
4. No analysis is performed

**Assertions:**
- [ ] Skill outputs a usage error when no argument is given
- [ ] Usage example is shown with the correct path format
- [ ] No impact analysis is performed without a target GDD
- [ ] Skill does NOT silently pick a GDD without user input

---

### Case 5: Director Gate — No gate spawned regardless of review mode

**Fixture:**
- A GDD has been revised with downstream references
- `production/session-state/review-mode.txt` exists with `full`

**Input:** `/propagate-design-change design/gdd/[system].md`

**Expected behavior:**
1. Skill reads the GDD and traces downstream references
2. Skill does NOT read `production/session-state/review-mode.txt`
3. No director gate agents are spawned at any point
4. Impact report is produced and per-artifact approval proceeds normally

**Assertions:**
- [ ] No director gate agents are spawned (no CD-, TD-, PR-, AD- prefixed gates)
- [ ] Skill does NOT read `production/session-state/review-mode.txt`
- [ ] Output contains no "Gate: [GATE-ID]" or gate-skipped entries
- [ ] Review mode has no effect on this skill's behavior

---

## Protocol Compliance

- [ ] Reads revised GDD and all potentially affected artifacts before producing impact report
- [ ] Impact report shown in full before any "May I write" ask
- [ ] "May I write" asked per artifact — never for the entire set at once
- [ ] In Progress stories flagged with elevated warning before their approval ask
- [ ] No director gates — no review-mode.txt read
- [ ] Ends with next-step handoff appropriate to verdict (COMPLETE or NO IMPACT)

---

## Coverage Notes

- ADR impact (when a GDD change requires an ADR update or new ADR) follows the
  same per-artifact approval pattern as story/epic updates — not independently
  fixture-tested.
- TR-registry impact (when changed GDD requires new or updated TR-IDs) is part
  of the analysis phase but not independently fixture-tested.
- The git diff comparison method (detecting what changed in the GDD) is a runtime
  concern — fixtures use pre-arranged content differences.
