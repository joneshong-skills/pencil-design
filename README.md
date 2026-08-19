<h1 align="center">Pencil Design</h1>

<p align="center">
  <a href="./README.md"><b>English</b></a> |
  <a href="./README.zh.md">繁體中文</a>
</p>

<p align="center">
  <a href="https://github.com/joneshong-skills/pencil-design/stargazers"><img src="https://img.shields.io/github/stars/joneshong-skills/pencil-design?style=flat-square" alt="GitHub Stars"></a>
  <a href="https://github.com/joneshong-skills/pencil-design/blob/main/LICENSE"><img src="https://img.shields.io/github/license/joneshong-skills/pencil-design?style=flat-square" alt="License"></a>
  <img src="https://img.shields.io/badge/version-0.2.0-blue?style=flat-square" alt="Version">
</p>

<p align="center">
  Expert design assistant for Pencil.dev -- AI-native vector design in your IDE via MCP, from ideation to design-to-code export.
</p>

---

## Features

- **IDE-Native Design** -- create and edit .pen vector files directly from Claude Code via Pencil MCP tools
- **Component Reuse** -- discover existing reusable components and design tokens before building, ensuring consistency
- **Section-by-Section Building** -- iterative build-verify-fix workflow with visual screenshot verification after each section
- **Multi-Screen Orchestration** -- parallelize multi-screen designs with sub-agents working on independent canvas zones
- **Design-to-Code Export** -- convert .pen designs to Tailwind CSS + shadcn/ui components with semantic class names
- **4 Built-in Design Systems** -- Shadcn UI, Halo, Lunaris, and Nitro style systems for rapid prototyping

## Usage

### Trigger Phrases

> "design in pencil", "create .pen", "pencil design", "edit .pen file", "design a page", "vibe coding", "design to code"

### Examples

```
# Load Pencil MCP tools (once per session)
mcp__mcpproxy__retrieve_tools(server_name: "pencil")

# Check current editor state
get_editor_state

# Discover reusable components
batch_get(reusable: true)

# Read design tokens
get_variables

# Build a section
batch_design([...operations...])

# Verify visually
get_screenshot
snapshot_layout(problemsOnly: true)
```

## Workflow

```
SELECT_MODE --> SCAFFOLD --> DESIGN --> REFINE --> EXPORT
```

### Standard 10-Step Flow

1. **mcpproxy** -- load Pencil MCP tools
2. **get_editor_state** -- inspect current file and selection
3. **batch_get** -- discover existing reusable components
4. **get_variables** -- read design tokens
5. **get_guidelines** -- load rules for the task type (web-app, mobile-app, landing-page, etc.)
6. **get_style_guide** -- choose visual direction (optional)
7. **find_empty_space_on_canvas** -- locate placement area
8. **batch_design** -- build one section at a time (max 25 ops per call)
9. **get_screenshot** -- visually verify the section
10. **snapshot_layout** -- detect overlaps and clipping issues

Repeat steps 8-10 for each section. Never build an entire page in a single batch_design call.

### batch_design Operations

| Op | Purpose |
|----|---------|
| **Insert** `I()` | Create new node |
| **Copy** `C()` | Clone existing node |
| **Update** `U()` | Modify properties |
| **Replace** `R()` | Swap component instance |
| **Move** `M()` | Reposition in tree |
| **Delete** `D()` | Remove node |
| **Image** `G()` | Fill with AI-generated or stock image |

## Integration

| Skill / Tool | Relationship |
|--------------|-------------|
| **frontend-design** | Load for unique visual identity (avoid generic AI aesthetic) |
| **brand-guidelines** | Apply brand tokens to .pen designs |
| **canvas-design** | Different tool: PNG/PDF art output, not .pen files |

## Installation

1. Install and run Pencil Desktop App or VS Code/Cursor extension
2. Ensure the Pencil MCP server is available before starting Claude Code
3. Place `SKILL.md` in `~/.claude/skills/pencil-design/`

## License

[MIT](https://opensource.org/licenses/MIT)
