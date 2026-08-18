---
name: pencil-design
description: "design, pencil, pen, create, edit, page, screen, Pencil 設計, 建立 .pen 檔案, 編輯 .pen, 設計畫面"
version: 0.1.0
tools: ToolSearch, mcp__pencil__get_editor_state, mcp__pencil__batch_get, mcp__pencil__batch_design, mcp__pencil__get_screenshot, mcp__pencil__snapshot_layout, mcp__pencil__get_guidelines, mcp__pencil__get_style_guide, mcp__pencil__get_style_guide_tags, mcp__pencil__get_variables, mcp__pencil__set_variables, mcp__pencil__find_empty_space_on_canvas, mcp__pencil__open_document, mcp__pencil__search_all_unique_properties, mcp__pencil__replace_all_matching_properties
argument-hint: "Describe the design you want to create or edit"
---

# Pencil Design

Expert design assistant for .pen files via Pencil MCP tools. Handles the full
workflow: ideation, style guide, component reuse, section-by-section building,
visual verification, and design-to-code export.

## Prerequisites

- Pencil Desktop App or VS Code/Cursor extension must be running BEFORE Claude Code starts
- MCP tools are deferred — load via `ToolSearch(query: "+pencil")` before first use
- .pen files are JSON-based, Git-friendly, **no auto-save** (remind user: Cmd+S)

## Agent Delegation

This skill runs in **main context** — MCP tool calls require ToolSearch which only
main context and `general-purpose` agents have.

### Multi-Screen Orchestration (Auto-Trigger)

**When to trigger**: Design involves 2+ screens/sections, OR user says "parallel" / "多工" / "同時".

**User provides only**: A design description (e.g., "Design a SaaS dashboard app with dark theme").
The skill autonomously decides: how many screens, what each section contains, how to parallelize.

**File strategy** (aligned with Figma community best practices):
- **Default: Single .pen file** — one file per module/project, organize by canvas sections
- **Split to multi-file only when**: pages/sections exceed ~10, OR performance degrades noticeably
- Rationale: single-file keeps components/tokens consistent, avoids cross-file sync overhead

#### Mode A: Single-File Sub-Agent (Default)

Multiple agents work on different **canvas sections** within ONE .pen file:

1. **Analyze** — Decompose description into N independent sections (screens/pages)
2. **Prepare shared context** (main context):
   - `get_editor_state` + `batch_get(reusable: true)` + `get_variables`
   - `find_empty_space_on_canvas` → plan N non-overlapping placement zones
   - Collect: design tokens, component refs, style guide choice
3. **Spawn N parallel `general-purpose` agents**, each with:
   - `ToolSearch(query: "+pencil")` to load MCP tools
   - Target section coordinates (x, y, width, height from step 2)
   - Full shared context: tokens, component refs, style guide name
   - Explicit batch_design instructions for their section only
   - Output constraint: "Return node IDs created and any issues encountered."
4. **Verify** (main context): `get_screenshot` + `snapshot_layout(problemsOnly: true)` per section

#### Mode B: Multi-File CLI Parallel (Overflow)

Only when sections > 10 or user explicitly requests separate files.
Reference `references/config.template.json` for entry schema and model selection guide.

1. **Analyze** — Decompose into N screens, assign model per complexity
2. **Setup**: `mkdir -p ~/workshop/outputs/pencil-parallel/screens`
3. **Pre-create**: `for s in <names>; do echo '{}' > .../screens/${s}.pen; done`
4. **Generate** `config.json` — N entries, each with: file path, detailed prompt, model
5. **Launch**: `cd ~/workshop/outputs/pencil-parallel && pencil --agent-config config.json`
6. **Verify** — Open each .pen, `get_screenshot` to check

## Design Workflow

### Standard 10-Step Flow

```
1. ToolSearch("+pencil")           ← Load MCP tools (once per session)
2. get_editor_state                ← Current file, selection, context
3. batch_get(reusable: true)       ← Discover existing components
4. get_variables                   ← Read design tokens
5. get_guidelines(topic)           ← Load rules for task type
6. get_style_guide_tags            ← Browse style options (optional)
   get_style_guide(tags)           ← Get visual direction (optional)
7. find_empty_space_on_canvas      ← Find placement space
8. batch_design                    ← Build section by section
9. get_screenshot                  ← Visual verify EACH section
10. snapshot_layout(problemsOnly)  ← Detect overlaps/clipping
```

Repeat steps 8-10 for each section. Never build an entire page in one batch_design call.

### Topic Selection for get_guidelines

| Task | Topic |
|------|-------|
| Landing page / marketing | `landing-page` |
| Web application / SaaS | `web-app` |
| Mobile app screen | `mobile-app` |
| Presentation slides | `slides` |
| Data table / dashboard | `table` |
| Using a design system | `design-system` |
| Generating code from design | `code` |
| Tailwind CSS implementation | `tailwind` |

Load multiple topics when relevant (e.g., `web-app` + `design-system`).

## batch_design Operations

Seven operations, max 25 per call. Operations execute sequentially; any failure rolls back all.

| Op | Syntax | Purpose |
|----|--------|---------|
| **Insert** | `id=I(parent, {type, ...})` | New node. Never set `id` manually. |
| **Copy** | `id=C(path, parent, {descendants, positionDirection, positionPadding})` | Clone node. Use `descendants` to modify children. |
| **Update** | `U(path, {props})` | Modify properties. Cannot change `id`, `type`, or `ref`. |
| **Replace** | `id=R(path, {nodeData})` | Replace node (component instance slot swaps). |
| **Move** | `M(nodeId, parent, index?)` | Reposition in tree. |
| **Delete** | `D(nodeId)` | Remove node. |
| **Image** | `G(nodeId, "ai"\|"stock", prompt)` | Fill frame/rectangle with generated or stock image. |

### Binding Rules

- Every I/C/R **must** have a binding name: `myCard=I(...)` not just `I(...)`
- `document` is the predefined root binding
- Bindings are valid only within a single batch_design call
- Use `+` for path navigation: `U(card+"/titleText", {content: "New"})`

### Copy Gotcha

Copy gives all descendants **new IDs**. Do NOT `U()` on copied children — use the
`descendants` parameter in the Copy call itself to modify children during copy.

### Move Gotcha

`M()` requires **real persisted node IDs**, not binding variables from `I()` in the same call.
If you need to reorder newly inserted nodes, do it in a **subsequent** batch_design call.

### flexWrap Gotcha

`wrap: true` / `flexWrap: "wrap"` on horizontal layouts does NOT reliably produce CSS-like
wrapping. Use **explicit row frames** instead (see `references/batch-design-examples.md`).

## Critical Rules

1. **Reuse components** — Always `batch_get(reusable: true)` first, insert via `ref`
2. **Use variables** — Reference token names, not hex values
3. **Prevent text overflow** — Text nodes need BOTH `width: "fill_container"` AND `textAutoResize: "height"`
4. **Verify every section** — `get_screenshot` + `snapshot_layout(problemsOnly: true)`
5. **No image node type** — Images are fills on frame/rectangle via `G()`
6. **Semantic naming** — `UserAvatarImage` not `Frame 42`
7. **25 ops max** per batch_design — split larger builds across calls
8. **Section-by-section** — Build iteratively: plan → build → verify → fix → next

## Design-to-Code Export

1. `get_guidelines("code")` + `get_guidelines("tailwind")`
2. `get_variables` → map tokens to Tailwind `@theme` directives
3. `batch_get` → read full design tree
4. Map Pencil components → shadcn/ui (or target library)
5. Generate semantic classes: `bg-primary` not `bg-[#3b82f6]`
6. Use CVA for variants, `cn()` for class merging, Lucide for icons

## Style Systems

4 built-in design systems: **Shadcn UI**, **Halo**, **Lunaris**, **Nitro**.
Select one early — reduces token cost by referencing known patterns.

## Quick Reference

### Keyboard Shortcuts

| Action | Shortcut |
|--------|----------|
| AI prompt panel | `Cmd+K` |
| Deep select element | `Cmd+Click` |
| Select parent | `Shift+Enter` |
| Make reusable component | `Cmd+Alt+K` |
| Group | `Cmd+G` |
| Create frame | `Cmd+Alt+G` |
| Duplicate | `Cmd+D` |
| Save | `Cmd+S` |
| Zoom to fit | `Cmd+0` |

### Figma Import

`Ctrl+C` in Figma → `Ctrl+V` in Pencil. Images don't transfer (re-add manually).
Complex designs: paste in sections to avoid clipboard limits.

### CLI (Desktop App Only)

```bash
pencil --agent-config config.json
```

Template: `references/config.template.json` — copy, modify entries, execute.
See **Agent Delegation → Multi-Screen Orchestration** for the full automated procedure.

**Constraints**:
- Each entry = independent Pencil window + independent Claude API call
- Max tested concurrency: 6 windows (bottleneck: RAM + API rate limit)
- `.pen` files must be pre-created (`echo '{}' > screen.pen`)
- Model selection: `claude-opus-4-6` / `claude-4.5-sonnet` for complex, `claude-4.5-haiku` for simple

### Token Optimization

- Select a design system upfront (AI references known patterns)
- Hide/delete irrelevant canvas frames before starting
- Use low `readDepth` / `searchDepth` values in batch_get
- Build section by section, not entire page at once

## Integration

- **frontend-design** — Load for unique visual identity (avoid generic AI aesthetic)
- **brand-guidelines** — Apply brand tokens to .pen designs
- **canvas-design** — Different tool: PNG/PDF art, not .pen files

## Additional Resources

### Reference Files
- **`references/config.template.json`** — Multi-agent CLI config template (copy + modify per task)
- **`references/batch-design-examples.md`** — Common batch_design patterns and recipes
- **`references/troubleshooting.md`** — Known issues, gotchas, and workarounds
