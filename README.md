# Energy Programs & Resources — Reference Data

Externalized reference data for the Energy Programs & Resources application (v1.2.5).

These JSON files contain large, relatively static datasets (glossaries, node/bus lists, market participants, etc.) that were previously embedded inline in the single-file HTML application. Externalizing them reduces the HTML file from **~2.0 MB → ~821 KB**.

Each `.json` file has a companion `.js` file (e.g. `miso-reference-data.js`) that wraps the same data as `window._refData_<rto> = {...};` for loading via synchronous `<script>` tags.

## Files

| File | Size (min) | Contents |
|------|-----------|----------|
| `miso-reference-data.json` | 186 KB | Nodes (2,596), acronyms (352), glossary (208), company registry (60), gen info, resources (13) |
| `spp-reference-data.json` | 475 KB | PNodes (1,583), buses (9,237), members (489), acronyms (181), glossary (413), gen info |
| `ercot-reference-data.json` | 587 KB | Settlement points (1,105), buses (19,339), members (2,239), acronyms (268), glossary (384), gen info |
| `pjm-reference-data.json` | 535 KB | Pricing nodes (14,386), glossary (164), members (1,131) |
| `isone-reference-data.json` | 346 KB | Pricing locations (1,211), acronyms (626), glossary (1,050), members (582), gen info |
| `caiso-reference-data.json` | 585 KB | Acronyms (623), glossary (1,746), members (376) |
| `nyiso-reference-data.json` | 79 KB | Generator nodes (747), transmission nodes (144), load zones (11), acronyms (379), glossary (379), market participants (440) |

## Loading

The HTML application loads all files synchronously at startup via XHR from `raw.githubusercontent.com`:

```javascript
var base = 'https://raw.githubusercontent.com/cnels127/energy-programs-resources-kb/refs/heads/main/';
var files = ['miso','spp','ercot','pjm','isone','caiso','nyiso'];
files.forEach(function(rto) {
  var xhr = new XMLHttpRequest();
  xhr.open('GET', base + rto + '-reference-data.json', false);
  xhr.overrideMimeType('application/json');
  xhr.send();
  if (xhr.status === 200) window['_refData_' + rto] = JSON.parse(xhr.responseText);
});
```

Each file is assigned to `window._refData_{rto}` and accessed via deep property chains with null guards.

## Schema Reference — NYISO

### `nyiso-reference-data.json`

```
{
  "genNodes": [[name, ptid], ...],                        // 747 generator nodes
  "transNodes": [[name, ptid, to, zone, subzone], ...],   // 144 transmission nodes
  "loadZones": [[name, ptid], ...],                       // 11 load zone PTIDs
  "acronyms": { "data": { "ACR": "Full Name", ... } },   // 379 acronyms
  "glossary": { "data": { "ACR": { "definition": "Full Name", "ref": "" }, ... } },  // 379 entries
  "asOf": "05/28/2026",
  "sources": { ... }
}
```

**Generator Nodes** (`genNodes`): `[0]` = node name, `[1]` = PTID

**Transmission Nodes** (`transNodes`): `[0]` = node name, `[1]` = PTID, `[2]` = TO, `[3]` = zone letter (A–K), `[4]` = subzone name

**Load Zones** (`loadZones`): `[0]` = zone abbreviation (e.g., "CAPITL"), `[1]` = PTID

**Acronyms** (`acronyms.data`): `Record<string, string>` — acronym → full name (379 entries merged from NYISO Acronyms PDF + 2003 PRL Evaluation glossary)

**Glossary** (`glossary.data`): `Record<string, {definition, ref}>` — keyed by acronym, matching rendering pattern of other RTOs

**Zone mapping**: A=West, B=Genesee, C=Central, D=North, E=Mohawk Valley, F=Capital, G=Hudson Valley, H=Millwood, I=Dunwoodie, J=New York City, K=Long Island

### Database Migration Mapping

| JSON Path | Table | PK | Notes |
|-----------|-------|----|-------|
| `nyiso.genNodes` | `nyiso_gen_nodes` | `ptid` | 747 generator nodes |
| `nyiso.transNodes` | `nyiso_trans_nodes` | `ptid + name` | 144 transmission nodes |
| `nyiso.loadZones` | `nyiso_load_zones` | `ptid` | 11 load zones |
| `nyiso.acronyms.data` | `nyiso_acronyms` | `acronym` | 379 acronym mappings |
| `nyiso.glossary.data` | `nyiso_glossary` | `term` | 379 glossary entries |

## Version History

- **v1.2.5** — Added `nyiso-reference-data.json`: generator nodes (747), transmission nodes (144), load zones (11), acronyms (379), glossary (379). Glossary merged from NYISO Acronyms PDF (338 entries) + 2003 PRL Evaluation (41 unique additions)
- **v1.2.5** — Added `caiso-reference-data.json` (585 KB): glossary, acronyms, members. Updated `isone-reference-data.json` with glossary (1,050), acronyms (626), members (582), pricing locations (1,211)
- **v1.2.4** — Initial externalization: miso, spp, ercot, pjm, isone reference data files
