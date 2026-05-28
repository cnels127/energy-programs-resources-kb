# Energy Programs & Resources — Reference Data

Externalized reference data for the Energy Programs & Resources application (v1.2.5).

These JSON files contain large, relatively static datasets (glossaries, node/bus lists, market participants, etc.) that were previously embedded inline in the single-file HTML application. Externalizing them reduces the HTML file from **~2.0 MB → ~700 KB** (a 66% reduction).

Each `.json` file has a companion `.js` file (e.g. `miso-reference-data.js`) that wraps the same data as `window._refData_<rto> = {...};` for loading via synchronous `<script>` tags.

## Files

| File | Size (min) | Contents |
|------|-----------|----------|
| `miso-reference-data.json` | 186 KB | Nodes (2,596), acronyms (352), glossary (208), company registry (60), gen info, resources (13) |
| `spp-reference-data.json` | 475 KB | PNodes (1,583), buses (9,237), members (489), acronyms (181), glossary (413), gen info |
| `ercot-reference-data.json` | 587 KB | Settlement points (1,105), buses (19,339), members (2,239), acronyms (268), glossary (384), gen info |
| `pjm-reference-data.json` | 526 KB | Pricing nodes (14,386), glossary (164) |
| `isone-reference-data.json` | 546 KB | Pricing locations (1,211), members (582), glossary (1,050), acronyms (626), gen info |
| `caiso-reference-data.json` | 585 KB | Glossary (1,746), acronyms (623), members (376) |

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

**`nodes.data`** — `Array<[pnodeId: number, pnodeName: string, typeIndex: number, zone: string]>`

Type indices: `0` = AGGREGATE, `1` = EHV, `2` = EXT, `3` = GEN, `4` = HUB, `5` = INTERFACE, `6` = LOAD, `7` = RESIDUAL_METERED_EDC, `8` = ZONE

**`nodes.types`** — `string[]` ordered type labels matching the type indices

**`glossary.data`** — `Record<string, { definition: string, ref: string, term?: string }>`

### ISO-NE

**`nodes.data`** — `Array<[locationId: number, locationName: string, typeIndex: number]>`

Type indices: `0` = DRRAZ, `1` = EXT. NODE, `2` = HUB, `3` = HUB NODE, `4` = LOAD ZONE, `5` = NETWORK NODE

**`nodes.types`** — `string[]` ordered type labels matching the type indices

**`members.data`** — `Array<{ name, id, status, sector, type, classification, roles: string[], voting }>` (582 NEPOOL participants)

**`acronyms.data`** — `Record<string, string>` mapping acronym → full name (626 entries)

**`glossary.data`** — `Record<string, { definition: string, ref: string, term?: string }>` (1,050 entries; keyed by acronym when available, otherwise by title-cased term — matching SPP/ERCOT/PJM pattern)

**`genInfo`** — Nested structure object (region, pricing nodes section)

### CAISO

**`acronyms.data`** — `Record<string, string>` mapping acronym → full name (623 entries)

**`glossary.data`** — `Record<string, { definition: string, ref: string, term: string }>` (1,746 entries; keyed by acronym when available, otherwise by title-cased term)

**`members.data`** — `Array<{ name, code, roles: string[] }>` (376 entities). Roles: SC (CAISO Scheduling Coordinator), WEIM SC (Western EIM SC), CRR (CRR Holder), CB (Convergence Bidding), EDAM SC (EDAM SC), PTO (Participating Transmission Owner — 23 entities per CAISO PTO list), DRP (Demand Response Provider), LSE (Load Serving Entity), UDC (Utility Distribution Company)

## Loading in the HTML App

The HTML file loads data via synchronous XHR in a `<script>` tag in the `<head>`, which sets `window._refData_*` globals before the Babel/React app code executes. Each variable declaration reads from these globals inline:

```javascript
// In <head> — synchronous XHR loader
(function() {
  var base = 'https://raw.githubusercontent.com/cnels127/energy-programs-resources-kb/refs/heads/main/';
  var files = ['miso','spp','ercot','pjm','isone','caiso'];
  files.forEach(function(rto) {
    try {
      var xhr = new XMLHttpRequest();
      xhr.open('GET', base + rto + '-reference-data.json', false);
      xhr.overrideMimeType('application/json');
      xhr.send();
      if (xhr.status === 200) {
        window['_refData_' + rto] = JSON.parse(xhr.responseText);
      }
    } catch(e) { /* falls through to empty stubs */ }
  });
})();

// In the app code — inline assignment at declaration
var MISO_NODES = (window._refData_miso && window._refData_miso.nodes.data) || [];
var PJM_NODES = (window._refData_pjm && window._refData_pjm.nodes.data) || [];
var ISONE_NODES = (window._refData_isone && window._refData_isone.nodes.data) || [];
// ... etc
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
| `pjm.nodes.data` | `pjm_nodes` | `pnode_id` |
| `isone.nodes.data` | `isone_nodes` | `location_id` |
| `isone.members.data` | `isone_members` | `id` |
| `isone.acronyms.data` | `isone_acronyms` | `acronym` |
| `isone.glossary.data` | `isone_glossary` | `term` |
| `caiso.acronyms.data` | `caiso_acronyms` | `acronym` |
| `caiso.glossary.data` | `caiso_glossary` | `term` |
| `caiso.members.data` | `caiso_members` | `code` |

## Versioning

The `_meta.version` field tracks which app version the data was exported from. The `_meta.asOf` date indicates the data snapshot date. Update both when refreshing data.
