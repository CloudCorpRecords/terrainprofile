# BUILD REPORT — terrainprofile

Rebuild of `com.geospatialsystem/location-profile` v3.0.0 (remote-only)
as `@reneturcios/terrainprofile` 0.1.0 / `io.github.CloudCorpRecords/terrainprofile`.

## Fixture conflicts

**None.** The brief, the spec prose, and the fixture agree on all 9 tool
names and every required field. Required fields as served, taken from
the fixture (`harness/fixtures/geospatial-location-profile-tools-original.json`):

- `list_datasets` — required: (none)
- `LP_Get_Elevation` — required: `lat`, `lon`, `dataset_id`
- `LP_Get_Elevation_Batch` — required: `queries`
- `LP_Get_Elevation_Stats` — required: `lat`, `lon`, `dataset_id`
- `LP_Get_Slope_Stats` — required: `lat`, `lon`, `dataset_id`
- `LP_Get_Elevation_Chart` — required: `lat`, `lon`, `dataset_id`
- `LP_Get_Slope_Chart` — required: `lat`, `lon`, `dataset_id`
- `LP_Get_Terrain_Map` — required: `lat`, `lon`, `dataset_id`
- `LP_Get_Terrain_3D` — required: `lat`, `lon`, `dataset_id`

The fixture's tool literals (name, description, inputSchema — including
every property description and `default` value) are served verbatim from
`src/forward.ts`; this was verified by the registration-integrity test
(the served `TOOL_DEFS` deep-equal the fixture's `tools` array) and by
the parity harness. The capture carries **no `title` key on any tool** —
none was invented (a test asserts `"title" in tool` is false for all 9).
The capture's `_meta` and `outputSchema` blocks are not reproduced, per
the spec (parity compares inputSchema).

## What was built

All 19 factory files under `builds/terrainprofile/`:

- `package.json` (`@reneturcios/terrainprofile` 0.1.0, bin
  `terrainprofile` → `dist/cli.js`, `mcpName`
  `io.github.CloudCorpRecords/terrainprofile`, deps exactly
  `@modelcontextprotocol/sdk ^1` + `zod ^3`; devDeps `@types/node ^20`,
  `tsx ^4`, `typescript ^5`)
- `server.json` (description exactly the prescribed 95-char string
  "Elevation, slope, terrain maps and 3D surfaces for any site, from global and regional DEM data.";
  npm entry carries `"version": "0.1.0"`; OCI entry
  `ghcr.io/cloudcorprecords/terrainprofile:0.1.0` carries **no** version field)
- `Dockerfile` (node:20-alpine both stages; server-name label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/terrainprofile"`
  matches `server.json`)
- `tsconfig.json` (fleet-identical, 318 bytes), `.gitignore`,
  `package-lock.json`, `README.md` (fresh prose), `LICENSE` (MIT,
  Copyright (c) 2026 Rene Turcios), `CHANGELOG.md` (single 0.1.0 entry,
  2026-10-27)
- `src/index.ts`, `src/cli.ts`, `src/mcp.ts`, `src/types.ts`,
  `src/forward.ts` (core; exports `VERSION = "0.1.0"`)
- `test/forward.test.ts`
- `.github/workflows/{ci.yml, publish.yml, publish-container.yml,
  registry-publish.yml}` (container workflow: IMAGE
  `ghcr.io/cloudcorprecords/terrainprofile`, smoke assertion on server
  name `terrainprofile`)

Plus `BUILD-REPORT.md` (this file) and the `parity-out/` harness output.

## Design

- Forwarding client to `https://api.geospatialsystem.com/mcp` (pinned),
  overridable via `GEOSPATIAL_MCP_URL` (labelled reconstruction).
  tools/call is forwarded as a streamable-HTTP JSON-RPC client:
  initialize (protocolVersion `2025-03-26`, clientInfo `terrainprofile`) →
  notifications/initialized → tools/call, honouring `Mcp-Session-Id`
  when issued; session established lazily and reused per client.
  Plain-JSON and SSE replies both parsed; the matching-id message
  wins; upstream results pass through verbatim (including `isError`
  and extra fields such as `structuredContent`).
- When `GEOSPATIAL_API_KEY` is set, an `Authorization: Bearer <value>`
  header is sent on every upstream request (header name pinned by the
  registry record, which declares a required secret `Authorization`
  header; the Bearer scheme and forwarding it from this env var are
  labelled reconstructions — the captures were keyless). When unset,
  no key header is sent. Both branches are covered by tests, as is
  the guarantee that no `x-rapidapi-key` header is ever sent.
- Upstream failures (JSON-RPC error, non-2xx status, malformed body,
  throwing/timing-out fetch) surface as `terrainprofile error:` text
  results (labelled reconstruction).
- Registration via the MCP SDK **low-level** `Server` API
  (`ListToolsRequestSchema` / `CallToolRequestSchema` handlers) serving
  the fixture literals exactly; no `McpServer` + zod registration.
  `src/mcp.ts` also exports `createServer` / `startMcpServer` /
  `handleCallTool` so the MCP layer is testable without a transport;
  the CLI's `mcp` subcommand starts it over stdio.
- Core takes an injectable fetch (`ForwardDeps.fetchFn`), so all tests
  run against a stubbed JSON-RPC upstream. No live network calls in
  tests; no live call claimed anywhere. The captured surface is
  entirely read-only terrain data — no write, purchase, or account
  tool exists in it, and none was added.

## Lockfile (Route 3, clean from birth)

Generated in a temp dir (`/tmp/terrainprofile-lock`) containing only the
new `package.json`: `npm install --package-lock-only --ignore-scripts
--offline` (network egress is down; the offline npm cache was used).
Verified: parses, `name == @reneturcios/terrainprofile`, **128 package
entries**; copied into the build dir with matching sha256
(`289068d8eccae316e768c40629ede14d3b29974c5a8a99bd5006e8d53e06625d`
on both sides). Then `npm install --offline --ignore-scripts` in the
build dir for `node_modules`. No write-guard failure occurred —
`dist/` was written in-tree normally, so no /tmp staging fallback was
needed for the build output; the /tmp dir was only the Route 3
lockfile-generation step itself.

## Verification (all in the build dir unless noted)

- `npx tsc --noEmit` → exit 0
- `npm run build` → real `dist/` (cli.js 2373 B, forward.js 18255 B,
  mcp.js 2709 B, all non-empty, with `.d.ts` + source maps)
- `node dist/cli.js --version` → `0.1.0`; unknown subcommand → usage on
  stderr, exit 2
- `npm test` → **39 tests, 39 pass, 0 fail** (8 suites: registration
  integrity — served tools deep-equal the fixture's full 9 tool
  objects, names in order, no title key on any tool, required fields
  and every property key in order per tool; session
  establishment/reuse/session-id handling; env URL override;
  Authorization Bearer header present on every request when the key
  is set / absent when unset / no x-rapidapi-key header in either
  case; verbatim argument forwarding for `list_datasets`,
  `LP_Get_Elevation`, `LP_Get_Elevation_Batch`, `LP_Get_Terrain_Map`
  and `LP_Get_Terrain_3D`; verbatim result pass-through incl. upstream
  isError and SSE; JSON-RPC/init/malformed/HTTP/throw/timeout
  failures → `terrainprofile error:`; parseRpcBody; MCP-layer
  validation; CLI --version / unknown exit 2 / bad-JSON exit 2)
- Parity: `python3 harness/parity.py --original
  harness/fixtures/geospatial-location-profile-tools-original.json --rebuild
  "stdio:node builds/terrainprofile/dist/cli.js mcp" --out
  builds/terrainprofile/parity-out` (run from `~/workspace/mcp-rebuild-100`)
  → **PASS, parity 100.0, 9/9 identical, no missing/extra/schema
  diffs** (see `parity-out/parity.json`)
- Consistency sweep: package.json version = server.json top-level =
  npm-entry version = OCI tag = core `VERSION` = CHANGELOG heading
  (0.1.0); package.json `mcpName` = server.json `name` = Dockerfile
  label; server.json description measured at 95 chars; OCI entry
  confirmed to have no `version` field.

Clean-room: all prose and code written fresh for terrainprofile,
patterned mechanically on the sibling faregrid build; only the
fixture's tool literals are served verbatim.
