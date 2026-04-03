<h1 align="center">Power Query CLI</h1>

<p align="center">
  A command-line interface for executing Power Query M expressions via the Microsoft Fabric API; preview semantic model partitions, test transformations, inspect steps
</p>

<p align="center">
  <img src="https://img.shields.io/badge/version-0.1.0-blue" alt="Version">
  <img src="https://img.shields.io/badge/Power_BI-F2C811?logo=powerbi&logoColor=000" alt="Power BI">
  <img src="https://img.shields.io/badge/Microsoft_Fabric-008272" alt="Microsoft Fabric">
  <img src="https://img.shields.io/badge/license-GPL--3.0-green" alt="License">
</p>

> **Early preview.** Commands and flags may change between versions.

> [!IMPORTANT]
> This CLI requires Microsoft Fabric, because the dependent APIs use a Dataflow Gen2 runner to execute Power Query. You can't (yet?) create a Gen2 dataflow in Power BI Pro, PPU, or Premium capacities.

## Install

```bash
cargo install pq-fabric
```

Or download a prebuilt binary from [Releases](https://github.com/data-goblin/pq-cli/releases).

## Prerequisites

- Azure CLI (`az`) installed and authenticated (`az login`)
- Access to a Microsoft Fabric workspace (Fabric capacity required)
- `fab` CLI installed (for partition preview; `cargo install fab-fabric`)

## Quick Start

```bash
# Set up a runner dataflow (once per workspace)
pq setup -w <workspace-id> --bind "myserver.database.windows.net;MyDatabase"
# Prints: export PQ_WORKSPACE=... export PQ_DATAFLOW=...

# Execute inline M
pq exec code "let x = 1 + 1 in x"

# Execute a table expression
pq exec code 'let t = #table({"Name","Value"}, {{"Widget",42000},{"Gadget",18500}}) in t'

# Preview a semantic model partition (first 100 rows)
pq exec partition "MyWorkspace.Workspace/MyModel.SemanticModel/Budget"

# Preview a specific step in the transformation chain
pq exec partition "MyWorkspace.Workspace/MyModel.SemanticModel/Orders" --step 1 --limit 5

# Run an existing dataflow query
pq exec dataflow --dataflow <df-id> -n "MyQuery"

# Pipe M from stdin
echo 'let x = List.Sum({1,2,3}) in x' | pq exec code -

# Output as CSV
pq exec code --csv 'let t = #table({"a","b"}, {{1,2},{3,4}}) in t'
```

## Command Reference

### Setup

```
pq setup -w <workspace-id>           Create a runner dataflow
  --name <name>                        Runner name (default: PQRunner)
  --bind <server;database>             Bind a SQL connection
  --cluster-id <id>                    ClusterId (auto-discovered if omitted)
```

The runner dataflow is a reusable execution context. Create once, reuse indefinitely. It sits idle when not in use (zero capacity cost).

### Execute

```
pq exec code <expression>            Execute inline M code
  -w, --workspace <id>                 Workspace ID [env: PQ_WORKSPACE]
  -d, --dataflow <id>                  Runner dataflow ID [env: PQ_DATAFLOW]
  -l, --limit <n>                      Max rows to display
  --csv                                Output as CSV
  --arrow-out <path>                   Save raw Arrow IPC to file
  -n, --query-name <name>              Query name (default: Result)

pq exec partition <path>             Preview a semantic model partition
  <path>                               Workspace.Workspace/Model.SemanticModel/Table
  -w, --workspace <id>                 Runner workspace [env: PQ_WORKSPACE]
  -d, --dataflow <id>                  Runner dataflow [env: PQ_DATAFLOW]
  -s, --step <n>                       Preview up to step index (0-based)
  -l, --limit <n>                      Max rows (default: 100)
  --csv                                Output as CSV

pq exec dataflow                     Run a named query from a dataflow
  -w, --workspace <id>                 Dataflow workspace [env: PQ_WORKSPACE]
  --dataflow <id>                      Target dataflow ID
  -n, --query-name <name>              Query name to execute
  -l, --limit <n>                      Max rows to display
  --csv                                Output as CSV
```

### List

```
pq list connections                  List tenant connections
  -f, --filter <text>                  Filter by path substring
  -t, --kind <type>                    Filter by type (SQL, Lakehouse, Web)

pq list runners -w <workspace-id>    List dataflows in a workspace
```

## How It Works

The CLI uses the Fabric [executeQuery API](https://learn.microsoft.com/en-us/rest/api/fabric/dataflow/query-execution/execute-query) to send M expressions to a Dataflow Gen2 item for evaluation. Results are returned as Apache Arrow streams and rendered as tables.

For partition preview, `pq` calls the `fab` CLI to extract the M expression and shared parameters from the semantic model's TMDL definition, inlines the parameter values, and optionally truncates to a specific step.

### Connection Binding

Data source access (e.g. `Sql.Database`) requires a connection bound to the runner dataflow. The `pq setup --bind` command handles this by:

1. Finding the connection in your tenant (by path match)
2. Discovering the ClusterId from an existing dataflow in the workspace
3. Pushing a definition with the connection binding via `updateDefinition`

> [!NOTE]
> If you don't have a connection yet you might need to create it with the APIs or if you prefer manually in the Fabric user interface.

## Environment Variables

| Variable | Description |
|----------|-------------|
| `PQ_WORKSPACE` | Default workspace ID (set by `pq setup`) |
| `PQ_DATAFLOW` | Default runner dataflow ID (set by `pq setup`) |
| `PQ_DEBUG` | Set to any value to print the generated mashup document |

## Limitations

- 90-second execution timeout (Fabric API limit)
- Requires a Fabric capacity (Dataflow Gen2 is not available on Pro/PPU/Premium)
- Data source access requires connections bound to the runner
- Partition preview requires `fab` CLI for TMDL extraction
- Preview API; may change

## Use or Re-use

You do not have the license to copy and incorporate this into your own products, trainings, courses, or tools. If you copy this project or use an agent to rewrite it, you must include attribution and a link to the original project.

<br>

---

<p align="center">
  <em>Built with assistance from <a href="https://claude.ai/claude-code">Claude Code</a>. AI-generated code has been reviewed but may contain errors. Use at your own risk.</em>
</p>

---

<p align="center">
  <a href="https://github.com/data-goblin">Kurt Buhler</a> · <a href="https://data-goblins.com">Data Goblins</a> · part of <a href="https://tabulareditor.com">Tabular Editor</a>
</p>
