# terrainprofile

terrainprofile is a small local bridge to the hosted
Geospatial System location-profile MCP server, which
reads elevation data from global and regional DEM
(digital elevation model) datasets. Give it a latitude
and longitude plus a dataset id, and it answers with the
elevation at that point, elevation and slope statistics
over small neighbourhood windows, range-box charts,
a composite terrain map, or a 3-D perspective surface
of the site.

The bridge keeps no terrain data of its own. It serves
the upstream server's tool list locally, exactly as
captured from the live endpoint, and forwards each tool
call upstream, returning the result untouched. The
captured surface is read-only terrain data: there is no
write, purchase, or account tool in it.

## Tools

- `list_datasets` — list all available elevation
  datasets on the server. Takes no arguments; call it
  first to get a `dataset_id`.
- `LP_Get_Elevation` — elevation in metres at one
  coordinate. Required: `lat`, `lon`, `dataset_id`.
  Available on all tiers including Free.
- `LP_Get_Elevation_Batch` — elevation for up to 100
  points in one call. Required: `queries`, a list of
  objects with `lat`, `lon`, and `dataset_id`.
  Available on all tiers including Free.
- `LP_Get_Elevation_Stats` — elevation statistics
  (min, average, max) for 3×3, 5×5 and 7×7 pixel windows
  centred on the coordinates. Required: `lat`, `lon`,
  `dataset_id`. Requires Pro or Enterprise key.
- `LP_Get_Slope_Stats` — terrain slope statistics
  (min, average, max) in degrees for the same window
  sizes, WGS-84 corrected. Required: `lat`, `lon`,
  `dataset_id`. Requires Pro key.
- `LP_Get_Elevation_Chart` — a range-box chart of
  elevation min/avg/max across the three window sizes,
  returned as a URL to the chart image. Required:
  `lat`, `lon`, `dataset_id`. Optional: `fmt`
  (`"png"` by default, also `"svg"` or `"pdf"`).
  Requires Pro key.
- `LP_Get_Slope_Chart` — the same range-box chart for
  terrain slope. Required: `lat`, `lon`, `dataset_id`.
  Optional: `fmt` (default `"png"`). Requires Pro key.
- `LP_Get_Terrain_Map` — a composite terrain map:
  satellite imagery plus smoothed contours with labels,
  flow-direction arrows, and a location pin, all in
  EPSG:3857. Required: `lat`, `lon`, `dataset_id`.
  Optional: `offset` (half-window pixels, default 20),
  `n_arrows` (flow arrow grid density, default 11),
  and `fmt` (default `"png"`). Requires Pro key.
- `LP_Get_Terrain_3D` — a 3-D perspective terrain
  surface coloured by elevation, with a red spike pin
  marking the query location. Required: `lat`, `lon`,
  `dataset_id`. Optional: `offset` (default 20),
  `elev_angle` (camera elevation, default 45),
  `azim_angle` (camera azimuth, default 135), and
  `fmt` (`"png"` by default, or `"html"`).
  Requires Pro key.

Coordinates are decimal degrees in WGS-84.

## Configuration

- `GEOSPATIAL_MCP_URL` (optional) — override the
  upstream endpoint. Defaults to
  `https://api.geospatialsystem.com/mcp`. (The override
  hook is this rebuild's own labelled reconstruction.)
- `GEOSPATIAL_API_KEY` (optional) — when set, every
  upstream request carries an
  `Authorization: Bearer <key>` header, for the caller's
  own Geospatial System key. The registry record for
  the original server declares a required secret
  `Authorization` header; the Bearer scheme and the
  forwarding from this env var are labelled
  reconstructions, because the original surface was
  captured keyless. When it is not set, no key header
  is sent.

### Forwarding mechanics

On the first tool call the client opens a streamable
HTTP session with the upstream: a JSON-RPC `initialize`
request (protocolVersion `2025-03-26`, clientInfo name
`terrainprofile`), then a `notifications/initialized`
notification, then the `tools/call` request carrying
the caller's arguments unchanged. When the upstream
answers `initialize` with an `Mcp-Session-Id` header,
that value rides on every later request of the session,
which is reused for the life of the process. Replies
may arrive as plain JSON or as server-sent events;
both are read.

Successful upstream results pass through verbatim,
including any `isError` flag the upstream sets.
Failures — a JSON-RPC error, a non-2xx status, an
unreadable response, or a request that cannot be
completed — come back as an `isError` result whose
text starts with `terrainprofile error:` (this rendering
is this rebuild's own labelled reconstruction).

## CLI

Each tool is also a subcommand taking one JSON argument
string:

```bash
npx @reneturcios/terrainprofile list_datasets
npx @reneturcios/terrainprofile LP_Get_Elevation '{"lat":37.7749,"lon":-122.4194,"dataset_id":"<id from list_datasets>"}'
npx @reneturcios/terrainprofile LP_Get_Elevation_Batch '{"queries":[{"lat":37.7749,"lon":-122.4194,"dataset_id":"<id>"}]}'
npx @reneturcios/terrainprofile LP_Get_Terrain_3D '{"lat":37.7749,"lon":-122.4194,"dataset_id":"<id>","fmt":"html"}'
npx @reneturcios/terrainprofile --version
```

Exit codes: `0` on success, `1` on an `isError`
result, `2` on a usage error.

## MCP server

```bash
npx -y @reneturcios/terrainprofile mcp
```

MCP client config:

```json
{
  "mcpServers": {
    "terrainprofile": {
      "command": "npx",
      "args": ["-y", "@reneturcios/terrainprofile", "mcp"]
    }
  }
}
```

## Docker

```bash
docker run -i ghcr.io/cloudcorprecords/terrainprofile
docker run --rm ghcr.io/cloudcorprecords/terrainprofile --version
```

The container's default command is `mcp`, so the first
form is a stdio MCP server; any other arguments reach
the CLI normally.

## Development

```bash
npm install
npm run build
npm test
npm run dev
```

The test suite runs entirely against a stubbed
JSON-RPC upstream — no live network calls.

## License

MIT — see `LICENSE`.
