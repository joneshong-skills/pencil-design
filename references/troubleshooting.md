# Troubleshooting

Common issues and workarounds for Pencil MCP tool usage.

## MCP Connection

### "Tool not found" or MCP tools unavailable
- **Cause**: Pencil app not running, or started after Claude Code
- **Fix**: Open Pencil Desktop/Extension FIRST, then restart Claude Code
- Also run `mcp__mcpproxy__retrieve_tools(server_name: "pencil")` to load deferred tools

### Authentication errors
- Ensure `claude` CLI is logged in (`claude login`)
- Remove `ANTHROPIC_API_KEY` from env if set (conflicts with MCP auth)
- Restart IDE after re-authenticating

## batch_design Failures

### "Operation failed, rolling back"
- Check node IDs exist (use `batch_get` to verify)
- Verify parent accepts children of that type
- Ensure binding names are unique within the call
- Check you're not exceeding 25 operations

### "Cannot update id/type/ref"
- U() cannot change `id`, `type`, or `ref` properties
- To change type: Delete + Insert a new node
- To change ref: Use R() (Replace) instead

### Updating copied node children fails
- Copy (C) generates new IDs for all descendants
- You cannot predict or reference these new IDs
- Use `descendants` param in the Copy call to modify children during copy

### Move (M) cannot use bindings from same call
- `M()` requires actual persisted node IDs, not binding variables from I() in the same batch_design call
- Wrong: `foo=I(parent, {...})` then `M(foo, otherParent, 0)` — foo is a binding, not a real ID
- Fix: Use M() on existing (already-persisted) node IDs, or do it in a subsequent batch_design call

### Deleting from reusable component cascades to all instances
- `D()` on a child of a `reusable: true` component removes it from ALL instances across all screens
- No need to delete from each screen's instance separately
- But updating/adding children to a reusable does NOT auto-propagate — instances need individual U() calls for overrides

## Visual Issues

### Text overflow / clipping
- `snapshot_layout(problemsOnly: true)` to detect
- Fix requires **both** properties on text nodes:
  1. `width: "fill_container"` — text node fills parent width
  2. `textAutoResize: "height"` — node grows vertically to fit wrapped text
- Setting only one of these is insufficient — text will still clip
- Parent frame must also have `layout: "vertical"` and constrained width
- Example: `U("textNodeId", {width: "fill_container", textAutoResize: "height"})`

### flexWrap does not work reliably
- Setting `wrap: true` or `flexWrap: "wrap"` on horizontal layouts does NOT produce
  CSS-like flex wrapping behavior — cards stay in a single row and overflow
- **Workaround**: Create explicit row frames instead of relying on wrap
  ```
  grid=I(parent, {type: "frame", name: "Grid", layout: "vertical", gap: 16})
  row1=I(grid, {type: "frame", name: "Row1", layout: "horizontal", gap: 16})
  c1=I(row1, {type: "ref", ref: "CardId", width: "fill_container"})
  c2=I(row1, {type: "ref", ref: "CardId", width: "fill_container"})
  c3=I(row1, {type: "ref", ref: "CardId", width: "fill_container"})
  row2=I(grid, {type: "frame", name: "Row2", layout: "horizontal", gap: 16})
  c4=I(row2, {type: "ref", ref: "CardId", width: "fill_container"})
  c5=I(row2, {type: "ref", ref: "CardId", width: "fill_container"})
  ```

### Elements overlapping
- Check `layout` property on parent frame (should be "vertical" or "horizontal")
- Verify `gap` is set between children
- Use `snapshot_layout` to identify exact overlap coordinates

### Blurry rendering
- Avoid fractional pixel values (10.43px → 10px)
- Snap to 8px grid
- Use integer values for x, y, width, height

## File Issues

### Changes not saving
- .pen has NO auto-save — must `Cmd+S` manually
- Git commit frequently as backup
- If file corrupted: `git checkout -- file.pen` to restore

### Large file performance
- Reduce `readDepth` / `searchDepth` in batch_get calls
- Hide unused frames on canvas
- Split large designs into multiple .pen files

## Image Issues

### "Cannot create image node"
- No `image` node type exists in .pen schema
- Create a `frame` or `rectangle`, then use `G()` to fill it

### Image not appearing after G()
- Verify the target node is a frame or rectangle
- Check the prompt is descriptive enough
- Try `"stock"` instead of `"ai"` for real photos

## Figma Import

### Missing images after paste
- Images don't transfer via clipboard
- Re-add images manually (drag-drop on macOS, or G() for AI fill)

### Broken layout after paste
- Flatten complex Figma groups before copying
- Paste in smaller sections
- Check auto-layout properties transferred correctly

## Codex CLI Conflict

### config.toml modified unexpectedly
- Pencil may modify or copy Codex CLI config
- Back up `~/.codex/config.toml` before first Pencil use
- If corrupted: restore from backup
