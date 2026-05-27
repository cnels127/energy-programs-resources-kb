# Energy Programs & Resources — Reference Data

Externalized reference data for the Energy Programs & Resources application (v1.2.4).

These JSON files contain large, relatively static datasets (glossaries, node/bus lists, market participants, etc.) that were previously embedded inline in the single-file HTML application. Externalizing them reduces the HTML file from **~2.0 MB → ~700 KB** (a 66% reduction).

## Files

| File | Size (min) | Contents |
|------|-----------|----------|
| `miso-reference-data.json` | 186 KB | Nodes (2,596), acronyms (352), glossary (208), company registry (60), gen info, resources (13) |
| `spp-reference-data.json` | 475 KB | PNodes (1,583), buses (9,237), members (489), acronyms (181), glossary (413), gen info |
| `ercot-reference-data.json` | 587 KB | Settlement points (1,105), buses (19,339), members (2,239), acronyms (268), glossary (384), gen info |
| `pjm-reference-data.json` | 37 KB | Glossary (164) |
| `isone-reference-data.json` | 7 KB | Gen info |

Pretty-printed versions are in `pretty/` for readability in PRs and diffs.

## Schema Reference

Each file has a `_meta` object with `rto`, `version`, `asOf`, and `description` fields. Data sections include a `_schema` string documenting the structure.

### MISO

**`nodes.data`** — `Array<[nodeName: string, nodeTypeIndex: number]>`

Type indices: `0` = CPNode, `1` = Aggregate, `2` = Interface, `3` = Load

**`acronyms.data`** — `Record<string, string>` mapping acronym → full name

**`glossary.data`** — `Record<string, { definition: string, ref: string }>` (Module A / BPM definitions)

**`companyRegistry.data`** — `Record<code, { name, cid, aliases, misoUrl, mapCenter, website, hifldName, ncrId, nercFunctions, ... }>`

**`genInfo`** — Nested structure object (hubs, zones, LBAs, etc.)

**`resources`** — `Array<{ full, badges, resQualKey, eligiblePrograms, ineligiblePrograms, ... }>`

### SPP

**`pnodes.data`** — `Array<[rtoZone: 0|1, pnodeName: string, busName: string]>`

Zone `0` = SPP East (SWPP), `1` = SPP West (WACM/WAUW)

**`buses.data`** — `Array<[rtoZone: 0|1, busName: string, area: string, station: string, kv: number|string, busType: string]>`

**`members.data`** — `Array<{ name, subcat, roles: string[], status, mpType, code }>`

**`acronyms.data`** — `Record<string, string>`

**`glossary.data`** — `Record<string, { definition: string, ref: string, term?: string }>`

### ERCOT

**`settlementPoints.data`** — `Array<[spName: string, spTypeCode: number]>`

Type codes: `1` = Hub, `2` = ResourceNode_Gen, `3` = ResourceNode_Load, `7` = PrivateUseNetwork_Gen, `8` = PrivateUseNetwork_Load, `9` = ResourceNode_Unit

**`buses.data`** — `Array<[busName: string, busNumber: number, station: string, area: string, zone: string, kv: number, owner: string]>`

**`members.data`** — `Array<{ name, short, subcat, roles: string[] }>`

**`acronyms.data`** — `Record<string, string>`

**`glossary.data`** — `Record<string, { definition: string, ref: string, term?: string }>`

### PJM

**`glossary.data`** — `Record<string, { definition: string, ref: string, term?: string }>`

### ISO-NE

**`genInfo`** — Nested structure object (hubs, zones, etc.)

## Loading in the HTML App

The HTML file loads these via `fetch()` at startup. Example integration pattern:

```javascript
// Configuration — set to your GitHub raw URL or CDN
var DATA_BASE_URL = 'https://raw.githubusercontent.com/your-org/repo/main/reference-data/';

// Async loader with fallback
async function loadReferenceData(rto) {
  try {
    const resp = await fetch(DATA_BASE_URL + rto + '-reference-data.json');
    if (!resp.ok) throw new Error(resp.status);
    return await resp.json();
  } catch (e) {
    console.warn('Failed to load ' + rto + ' data:', e);
    return null;
  }
}

// On app init, load all RTOs in parallel
Promise.all([
  loadReferenceData('miso'),
  loadReferenceData('spp'),
  loadReferenceData('ercot'),
  loadReferenceData('pjm'),
  loadReferenceData('isone')
]).then(function([miso, spp, ercot, pjm, isone]) {
  if (miso) {
    MISO_NODES = miso.nodes.data;
    MISO_ACRONYMS = miso.acronyms.data;
    ACRONYM_DEFS = miso.glossary.data;
    COMPANY_REG = miso.companyRegistry.data;
    MISO_GEN_INFO = miso.genInfo;
    MISO_RESOURCES = miso.resources;
  }
  // ... etc for other RTOs
});
```

## Database Migration

These JSON files are structured for straightforward database import. Each top-level key maps to a table or collection:

| JSON Key | Suggested Table | Primary Key |
|----------|----------------|-------------|
| `*.acronyms.data` | `acronyms` | `(rto, acronym)` |
| `*.glossary.data` | `glossary_terms` | `(rto, term)` |
| `miso.nodes.data` | `miso_nodes` | `node_name` |
| `spp.pnodes.data` | `spp_pnodes` | `pnode_name` |
| `spp.buses.data` | `spp_buses` | `bus_name` |
| `ercot.settlementPoints.data` | `ercot_settlement_points` | `sp_name` |
| `ercot.buses.data` | `ercot_buses` | `bus_name` |
| `ercot.members.data` | `ercot_members` | `(name, short)` |
| `spp.members.data` | `spp_members` | `name` |
| `miso.companyRegistry.data` | `miso_companies` | `code` |

## Versioning

The `_meta.version` field tracks which app version the data was exported from. The `_meta.asOf` date indicates the data snapshot date. Update both when refreshing data.
