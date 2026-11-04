# Changelog

## 0.1.0 (2026-10-27)

- First release: a forwarding client for the hosted Geospatial System location-profile MCP server, serving all 9 captured tools locally for tools/list and forwarding every tools/call upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when issued).
- Tools: `list_datasets`, `LP_Get_Elevation`, `LP_Get_Elevation_Batch`, `LP_Get_Elevation_Stats`, `LP_Get_Slope_Stats`, `LP_Get_Elevation_Chart`, `LP_Get_Slope_Chart`, `LP_Get_Terrain_Map`, `LP_Get_Terrain_3D`.
- Upstream results pass through verbatim; upstream failures surface as `terrainprofile error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). Upstream endpoint overridable via `GEOSPATIAL_MCP_URL`; when `GEOSPATIAL_API_KEY` is set it is sent as an `Authorization: Bearer <value>` header on every upstream request.
