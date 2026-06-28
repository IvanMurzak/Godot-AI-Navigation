# CLAUDE.md — Godot-AI-Navigation

A **Godot-MCP extension**: an MCP tool family for Godot's built-in navigation nodes
(`NavigationRegion2D`/`3D`, `NavigationAgent2D`/`3D`, `NavigationLink2D`/`3D`), shipped as a
**source-only NuGet package** (`com.IvanMurzak.Godot.MCP.Navigation`) that compiles inside a consumer's
Godot project against the consumer's own GodotSharp. Created from
[`Godot-AI-Tools-Template`](https://github.com/IvanMurzak/Godot-AI-Tools-Template). The packaging recipe
is the load-bearing detail — read `docs/source-only-nuget-recipe.md`.

## Layout

- `src/Godot-AI-Navigation/` — the source-only package (`Godot.NET.Sdk`).
  - `Runtime/Tools/Tool_Navigation.cs` — the `[AiToolType]` family (one partial class).
  - `Runtime/Tools/Tool_Navigation.Ids.cs` — all tool-id consts (pure-managed; pinned by tests).
  - `Runtime/Navigation/` — pure-managed support types: `NavigationDimension` (2D/3D), `NavigationNodeInfo`,
    and the agent-config clamp/validation rules (all unit-tested, no Godot native API).
  - `Editor/Tools/Tool_Navigation.{Editor,RegionCreate,RegionSetMesh,AgentCreate,AgentConfigure,LinkCreate,Get}.cs`
    — editor tools behind `#if TOOLS` (touch `EditorInterface`/live nodes; main-thread-marshalled; E2E-verified).
  - `build/com.IvanMurzak.Godot.MCP.Navigation.props` — the source-injection props (auto-imported by NuGet
    in the consumer; MUST stay named `<PackageId>.props`).
- `tests/Godot-AI-Navigation.Tests/` — xUnit specs for the pure-managed sources only (no Godot binary).
- `testbed/Navigation-Testbed.csproj` — a consumer `Godot.NET.Sdk` project that restores the local-packed
  package; `dotnet build` of it is the source-injection proof.

## Tools

| Tool | Kind | File |
| --- | --- | --- |
| `navigation-region-create` | editor | `Editor/Tools/Tool_Navigation.RegionCreate.cs` |
| `navigation-region-set-mesh` | editor | `Editor/Tools/Tool_Navigation.RegionSetMesh.cs` |
| `navigation-agent-create` | editor | `Editor/Tools/Tool_Navigation.AgentCreate.cs` |
| `navigation-agent-configure` | editor | `Editor/Tools/Tool_Navigation.AgentConfigure.cs` |
| `navigation-link-create` | editor | `Editor/Tools/Tool_Navigation.LinkCreate.cs` |
| `navigation-get` | editor | `Editor/Tools/Tool_Navigation.Get.cs` |

The tools wrap Godot's built-in navigation: regions (`NavigationRegion2D`/`3D` + their
`NavigationPolygon`/`NavigationMesh`), agents (`NavigationAgent2D`/`3D`), and links
(`NavigationLink2D`/`3D`). Pure-managed cores (dimension handling, agent-config clamping, node-info
shaping) live under `Runtime/` and are the unit-tested surface.

## Build / test (no Godot binary)

```bash
dotnet build src/Godot-AI-Navigation/Godot-AI-Navigation.csproj   # source-only package compiles tools
dotnet test  tests/Godot-AI-Navigation.Tests/Godot-AI-Navigation.Tests.csproj
dotnet pack  src/Godot-AI-Navigation/Godot-AI-Navigation.csproj -p:Version=0.0.0-ci -o local-nuget
dotnet build testbed/Navigation-Testbed.csproj                  # consumes the local package (injection proof)
```

`Godot.NET.Sdk` supplies GodotSharp from NuGet, so no Godot install is needed to build/test/pack or to
prove the source-injection recipe (the testbed build is a faithful proxy for `godot --build-solutions`).
The recipe is verified to compile into the consumer across the CI's multi-Godot matrix (4.3 / 4.4 / 4.5).
When proving locally, note `dotnet pack` re-uses the **global NuGet cache** for an already-cached version:
if you re-pack the same `Version`, clear `~/.nuget/packages/com.ivanmurzak.godot.mcp.navigation/<ver>` (or
pack a unique version) before re-restoring the testbed, or you'll silently build the stale cached source.

## Conventions

- Root namespace `com.IvanMurzak.Godot.MCP.Navigation`. Every `.cs` starts with the Apache-2.0 header.
- Pure-managed tools/cores (no Godot native API) → `Runtime/` (outside `#if TOOLS`, unit-testable);
  editor-driving tools → `Editor/` (behind `#if TOOLS`, every Godot call via `MainThread.Instance.Run(...)`,
  E2E-verified). Godot types that wrap a native object (`Node`, `Resource`, `NodePath`, `SceneTree`) crash
  a no-Godot xUnit host on construction — keep them out of `Runtime/`.
- The package declares ONLY the `com.IvanMurzak.McpPlugin` / `com.IvanMurzak.ReflectorNet` min-version
  deps; **GodotSharp must never become a package dependency** (CI asserts the nuspec). Keep the MCP pins in
  lockstep with the core Godot-MCP addon; bump with `commands/update-core.ps1`.
- One `[AiToolType] partial class Tool_Navigation`; one `[AiTool]` method per partial-class file. New
  pure-managed sources must be added to the test csproj `<Compile Include>` list to be unit-tested.

## Find detail in

- `docs/source-only-nuget-recipe.md` — the packaging recipe (the centerpiece) + the consumer story.
- `docs/ci.md` — workflows, the version gate, multi-Godot matrix, required secrets.
