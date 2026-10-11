<h1 align="center">Godot AI Navigation</h1>

<p align="center">
  AI <b>MCP tools</b> for Godot <b>Navigation</b> (2D &amp; 3D) — an extension for
  <a href="https://github.com/IvanMurzak/Godot-MCP">Godot-MCP / AI Game Developer</a>.
</p>

`Godot-AI-Navigation` adds a focused MCP tool family for Godot's built-in navigation nodes —
`NavigationRegion2D`/`NavigationRegion3D`, `NavigationAgent2D`/`NavigationAgent3D`, and
`NavigationLink2D`/`NavigationLink3D` — authored in C# with `[AiToolType]` / `[AiTool]` (the same model
as Unity-MCP and the core Godot-MCP addon), and shipped as a **source-only NuGet package** that compiles
inside any consumer's Godot project against the consumer's own GodotSharp — no bundled Godot, no version
lock. Created from
[`Godot-AI-Tools-Template`](https://github.com/IvanMurzak/Godot-AI-Tools-Template).

## Tools

| Tool | Kind | Description |
| --- | --- | --- |
| `navigation-region-create` | editor (`#if TOOLS`) | Create a `NavigationRegion2D`/`NavigationRegion3D` node in the edited scene; optional parent, name, `dimension` (2D/3D). |
| `navigation-region-set-mesh` | editor (`#if TOOLS`) | Assign a `NavigationPolygon` (2D) / `NavigationMesh` (3D) resource to a region by `res://` path. |
| `navigation-agent-create` | editor (`#if TOOLS`) | Create a `NavigationAgent2D`/`NavigationAgent3D` under a parent node. |
| `navigation-agent-configure` | editor (`#if TOOLS`) | Update an agent's scalar properties — `radius`, `path_desired_distance`, `target_desired_distance`, `max_speed`, `avoidance_enabled`; each clamped/validated. |
| `navigation-link-create` | editor (`#if TOOLS`) | Create a `NavigationLink2D`/`NavigationLink3D` with start/end positions. |
| `navigation-get` | editor (`#if TOOLS`) | Read a navigation node's scalar config (read-only). |

The pure-managed cores behind these tools (dimension handling, the agent-config clamp/validation rules,
node-info shaping) live under `src/Godot-AI-Navigation/Runtime/` and are CI-unit-tested; the editor-driving
tools live under `Editor/` behind `#if TOOLS` and marshal every Godot call onto the editor main thread via
`MainThread.Instance.Run(...)`.

## Install (in a consumer Godot project)

Requires the core [`godot_mcp`](https://github.com/IvanMurzak/Godot-MCP) addon. Then either:

- **Extensions dock** — pick it inside the Godot editor (Install → adds the `<PackageReference>` → rebuild).
- **CLI** — `godot-cli install-extension com.IvanMurzak.Godot.MCP.Navigation`.
- **By hand** — add `<PackageReference Include="com.IvanMurzak.Godot.MCP.Navigation" Version="x.y.z" />`
  to the consumer `.csproj` and rebuild.

After a rebuild the `[AiToolType]` tool family is auto-discovered — no registry edit.

## Build & test (no Godot binary needed)

`Godot.NET.Sdk` pulls GodotSharp from NuGet, so the package builds and unit-tests headless:

```bash
dotnet build src/Godot-AI-Navigation/Godot-AI-Navigation.csproj            # compiles tools (Godot API resolves)
dotnet test  tests/Godot-AI-Navigation.Tests/Godot-AI-Navigation.Tests.csproj   # pure-managed unit tests
dotnet pack  src/Godot-AI-Navigation/Godot-AI-Navigation.csproj -p:Version=0.0.0-ci -o local-nuget
dotnet build testbed/Navigation-Testbed.csproj                            # consumer build = source-injection proof
```

The testbed build proves the source-injection recipe: the package's `.cs` are injected as `<Compile>`
items into the consumer and compile against the consumer's own GodotSharp. CI runs this across a
multi-Godot-version matrix; an end-to-end leg additionally boots real headless Godot, installs the core
addon, and (once a local MCP server is wired) calls each tool via `godot-cli run-tool`.

## Docs

- `docs/source-only-nuget-recipe.md` — the packaging recipe (the centerpiece).
- `docs/ci.md` — workflows, the version gate, the multi-Godot matrix, required secrets.
- `AGENTS.md` — agent / maintainer notes.

## Publish

Source-only, version-gated release (see `docs/ci.md`): publishing uses NuGet Trusted Publishing (OIDC,
no stored API key), bump `<Version>` (`commands/bump-version.ps1 -NewVersion x.y.z`), merge to `main`;
`release.yml` runs the full matrix, publishes the package to NuGet, and cuts an atomic GitHub Release.

License: **Apache-2.0**.
