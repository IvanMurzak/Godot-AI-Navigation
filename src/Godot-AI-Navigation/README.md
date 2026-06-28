# Navigation Tools

AI MCP tools for Godot **Navigation** (regions, agents & links, 2D & 3D), for [Godot-MCP / AI Game Developer](https://github.com/IvanMurzak/Godot-MCP).

A **source-only** MCP tool extension: the package ships C# source (no compiled DLL, no bundled Godot)
that compiles inside your Godot project against your own GodotSharp, so it never locks you to a Godot
version. Verified to compile into a consumer project across Godot 4.3 … 4.5.

## Install

Requires the core [`godot_mcp`](https://github.com/IvanMurzak/Godot-MCP) addon in your Godot C# project.

```bash
# via the godot-cli (resolves from the shared catalog, edits your .csproj, rebuilds)
godot-cli install-extension com.IvanMurzak.Godot.MCP.Navigation

# …or add the reference manually and rebuild:
#   <PackageReference Include="com.IvanMurzak.Godot.MCP.Navigation" Version="0.1.0" />
```

…or pick it from the **Extensions** dock inside the Godot editor.

After a rebuild, the extension's `[AiToolType]` tool family is auto-discovered — no registry edit.

## Tools

Wraps Godot's built-in navigation nodes: `NavigationRegion2D/3D` (navigable areas), `NavigationAgent2D/3D`
(pathfinding + RVO avoidance), and `NavigationLink2D/3D` (off-mesh connections).

| Tool | Kind | Description |
| --- | --- | --- |
| `navigation-defaults` | pure-managed | Return the recommended starter agent config (radius, desired distances, max speed) for a 2D/3D agent. |
| `navigation-region-create` | editor | Create a `NavigationRegion2D`/`NavigationRegion3D` (navigable area) in the edited scene. |
| `navigation-region-set-mesh` | editor | Assign a region's navigation resource (`NavigationPolygon` 2D / `NavigationMesh` 3D) by `res://` path, or attach a fresh empty one. |
| `navigation-agent-create` | editor | Create a `NavigationAgent2D`/`NavigationAgent3D` under a parent node. |
| `navigation-agent-configure` | editor | Update an agent's scalar properties (radius, path/target desired distance, max speed, avoidance) — each clamped to a valid range. |
| `navigation-link-create` | editor | Create a `NavigationLink2D`/`NavigationLink3D` with start/end positions. |
| `navigation-get` | editor | Read a navigation node's scalar config (read-only). |

All editor tools marshal every Godot call onto the editor main thread; agent values are clamped to valid
ranges before they touch a node.

License: Apache-2.0.
