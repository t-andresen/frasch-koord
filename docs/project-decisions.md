# frasch-maps: Decisions

A Google-Maps-like web map with all labels and UI in **North Frisian**, based on the
**OpenMapTiles** vector tile schema. This file holds the decisions behind it that the code does
not show by itself.

## How this file is kept

- It states the **current** decisions, ordered by topic. It is not a log.
- An entry belongs here only if the code does not show it and someone could undo it by accident:
  domain, linguistic and legal choices, the owner's choices, rejected alternatives, special cases
  in the data.
- One bullet per decision: the decision, its reason, the rejected alternative, the issue number.
- A changed decision replaces its entry. Nothing is appended, and nothing is marked "superseded".
- How the code works belongs in the code and the READMEs; what a change did, in its pull request;
  open questions and planned work, in GitHub issues.
- History: the issues named in the entries, and git. The full log up to 2026-11-15 is
  [this file at `30d632d`](https://github.com/t-andresen/frasch-koord/blob/30d632d/docs/project-decisions.md).

## Scope

- **Phase 1 covers North Frisia; later come country names and larger cities worldwide, with
  detailed Frisian names only for Germany, Denmark and the Netherlands.** Name coverage and tile
  extent are independent: tiles may cover more than the name list and then fall back to German or
  local names.
- **One full planet build in the OpenMapTiles schema (~100 GB PMTiles) is the single style
  source.** The map must never be blank anywhere. The planet does not fit on the dev machine, so
  development uses the Schleswig-Holstein extract (plus Denmark for name matching) and the
  planet is built once on a rented VM. Rejected: "world to z7 plus a detail region", because it
  is blank outside the region beyond z7 or shows duplicate labels. (#200)

## Languages & dialects

- **Names, tiles and UI are dialect-aware from the start, although Mooring is the initial
  dialect.** Other dialects (Fering, Öömrang, Sölring, ...) are added later, with per-dialect
  name fields and per-dialect UI translation files. Tags are `frr-x-<subtag>`: ISO 639-3 `frr`
  plus a private-use subtag (at most 8 characters), because BCP 47 has no registered dialect
  subtags.
- **`names/dialects.csv` is the only list of dialects.** The place-list columns, the injected
  `name:*` tags, Planetiler `--languages`, the search index and the frontend selector all derive
  from it, so adding a dialect is one line plus a column and no code change. Only rows with
  `view=yes` are selectable, and only a Frisian dialect can be a map view.
- **The registry also holds the other local varieties: `nds` (Low Saxon, "Plattdeutsch") and
  `jut` (Synnejysk, "Süderjütisch").** A `language` column (ISO 639) tells them apart. A tag is
  `<language>` or `<language>-x-<subtag>`; a Frisian row needs a subtag, because a bare `frr` is
  OSM's generic `name:frr`. "Frisian" means `language == frr`. (#83)
- **`names/places.csv` has one row per place and one column per dialect, plus a `local`
  column.** North Frisian is a dozen varieties, so each is first-class. `local` holds the form
  the people of the place use where it differs from the area's dialect (sub-dialects such as
  Fahretoft); empty means the area's dialect, and a bracket remark names the variety, e.g.
  `Brouersweerw (Foortuftinge)`. There is no catch-all `other` column.
- **Low Saxon is the local majority language in northern Germany.** Südergoesharde is extinct
  (1981), but its historic names still label the local view. Nordstrand, Pellworm and Eiderstedt
  get no Frisian area (Low Saxon there). Names of the old sheet marked `(Gooshiirdinge)` sit in
  the Goesharde the place lies in. There is no `older` column: island and Hallig names live in
  their dialect's column, mainland names in the Harde's column where OSM's `name:frr` confirms
  the form. These are the owner's decisions.
- **The `nds` and `jut` columns follow `de`; they are no Frisian names.** A row is never named
  by them as a Frisian name (`any_name`), the matcher does not search them, and a row whose only
  names are `nds`/`jut` stays off the map. For single-name fallbacks they rank after German and
  Danish but are not left out. (#83, #154)
- **Low Saxon is one name: the list's, else OSM's `name:nds`.** It is decided in `resolve`
  alone, and the tiles keep OSM's where the list has none, as with `name:de`. Rejected: a
  separate search field `name_nds` and card line `card.lowSaxon`. (#83, #61)

## Name list, licence & legal

- **The name list is the owner's own work, kept in git as the hand-edited `names/places.csv`
  and merged into the tiles at build time.** The original Google Sheet is no longer consulted;
  its export is archived in `names/bootstrap/`, and only map-relevant columns were imported
  (inhabitant, nds and source columns dropped on purpose). Names are edited as CSV through pull
  requests. Rejected: upstreaming to OSM `name:frr` as the main strategy, because OSM has one
  `frr` field without dialects, so a custom mechanism is needed anyway (contributing to OSM can
  still happen separately). Rejected: swapping labels in the browser, because big lookup
  expressions are slow and OSM IDs are not reliably present in OpenMapTiles output.
- **The published name list "Frasche stääsnoome" (`names/places.csv` and the other hand-edited
  files in `names/`) is licensed under ODbL 1.0, contents under DbCL 1.0, full text in
  `names/LICENSE`.** Attribution string: "Frasche stääsnoome, Thore Andresen, ODbL 1.0". The
  tiles are ODbL anyway (OSM-derived), so a permissive licence gains nothing; ODbL lets anyone
  push names into OSM without a waiver, and share-alike returns extended lists to the public.
  Accepted cost: third parties cannot carry the names into Wikidata (CC0); the author can, as
  rights holder. This works because the sources are many cited publications (Hoekstra 2015,
  Nordfriisk Instituut, Löfstedt 1931, literary texts) and `places.csv` holds only OSM
  references, not OSM names or coordinates. Rejected: CC BY-SA, because OSM cannot import it.
- **The map always shows "© OpenStreetMap contributors" and the OpenMapTiles credit.** It is a
  fixed licence notice in every UI language, because ODbL applies to published databases
  derived from OSM. (#22)
- **The code is MIT (root `LICENSE`); the style fork keeps OSM Bright's BSD / CC BY notice in
  `web/src/map/style/LICENSE-osm-bright.md`.** (#22)

## Dialect areas

- **The dialect of an object is taken by point-in-polygon against
  `names/dialect_areas.geojson`, smallest area wins.** `names/dialect_areas.csv` names the OSM
  municipalities and island polygons of each dialect and `build_dialect_areas.py` builds the
  committed geojson. Assignment is by geometry, never by name; "smallest wins" resolves nesting
  (Hamburger Hallig lies inside Reußenköge).
- **Mainland areas are a researched draft, assigned to the six Harden by historic Harde
  membership from German Wikipedia.** Rows whose `note` says `medium` or `low` need the owner's
  check, above all the Gotteskoog border villages Holm, Uphusum and Galmsbüll and the assignment
  of Stedesand to the Karrharde. (#2)
- **Bohmstedt, Drelsdorf and Ahrenshöft follow the documented Mittelgoesharder dialect, not
  their historic Harde membership.** They form one parish cluster and are the core
  Mittelgoesharder-Friesisch villages, although Ahrenshöft belonged to the Nordergoesharde. (#2)
- **Synnejysk is the default local language of Schleswig, Low Saxon the default south of it.**
  `jut` rows cover parts of Schleswig-Flensburg, Flensburg and the Nordfriesland mainland that no Frisian
  row claims, each marked "low - default from #83, unverified". A place in no area is Low Saxon:
  Eiderstedt, Nordstrand, Elisabeth-Sophien-Koog, Pellworm, Friedrichstadt, the Nordsee-Treene
  villages south and east of Husum, Stapelholm, the city of Schleswig and Rendsburg-Eckernförde's
  part of Schleswig. These are the owner's decisions; the `jut` rows are a starting point to
  review. North of the border is #15. (#83)
- **`names/dialect_areas_parts.geojson` has one Feature per municipality and is never read by
  the pipeline.** It exists for review, because the dissolved file shows neither municipality
  name nor note, so a wrong assignment is invisible. It includes Kreis Nordfriesland
  municipalities no row claims (`assigned: false`), found by `admin_level=8` relations with a
  `de:regionalschluessel` starting `01054` (rejected: a bounding box, which catches neighbouring
  Kreise). Unassigned features carry no `dialect`, so `AreaIndex.from_geojson` refuses the file;
  its unit is the municipality, so reading it would silently change every dialect lookup.
  Simplification is 0.0001° (~11 m), since 0.0005° leaves cracks between neighbours. It is
  committed because regenerating needs the gitignored 158 MB extract. (#2)
- **The dev-only review view `?areas` is read-only and does not model confidence.** It colours
  one fill per dialect and shows the `note` verbatim on selection. Rejected: a patch file as in
  `?curate`, because the CSV is small and hand-edited and a patch file would add a second way to
  change the data; a `confidence` column or note parsing, because the research prose ("best guess
  only") must not ship to the public site. (#2)

## Matching

- **The matcher compares names exactly after normalisation; there is no fuzzy matching.** A
  match is the OSM reference `type/id` (several for a split river or dyke) plus the Wikidata item
  where the object has one. Countries are matched through Wikidata alone.
- **The matcher only rewrites rows with `status=auto` or with empty `osm` and `wikidata`.**
  Hand-filled rows are never touched, so `git diff` after a run shows exactly what it proposed. An
  owner reviews a proposal by writing the `osm` reference, optionally `status=ok`, or `status=skip`.
- **`frasch update` runs the matcher every time, whatever the stamps say.** The matcher changes
  `places.csv` and depends on Wikidata's answers, which no stamp covers. Only the scan of an
  extract is skipped when its output is not stale. Rejected: skipping the matcher by stamp. (#95)
- **A row holds the objects of its `osm` cell and its Wikidata item, unless it is `skip`.** It
  holds them also while it is `not_a_place` or has no Frisian name yet; it is on the map only if
  it holds something, is a place and has a Frisian name. (#94)
- **Two rows claiming one object or item is an error of the list itself.** `check-inputs` lists
  it and reading the list refuses it, so every command stops with file and line. It was reported
  three times with three severities before. Rejected: merging the claims (first non-empty value
  wins). (#94, #93)
- **Nothing writes a double claim.** `frasch curate apply` refuses a decision for an object or
  item another row holds. The matcher counts what it gives out in a run as claimed: of two rows
  for one place the first gets the object, the second stays unmatched with "... is taken by row
  `<id>`" (an id, as a line number goes stale). An object whose item another row holds is
  matched without the item (OSM tags an island and its village with one item); a country, found
  by its item alone, is then not found. (#94, #98)
- **A row the matcher matched holds the whole feature against the rows after it, as a human
  decision does.** Otherwise the second of two rows for one feature got a sibling object (the
  river's ways when the first got the relation). The exception is a row that finds the object only
  by a name it merely also goes by (`alt_name`, `old_name`): the dyke Deezbüll Deich carries
  `alt_name=Deezbülleck`, and the hamlet Deezbülleck keeps its node. Rank cannot tell this case,
  as an annotated `short_name` ranks like an `alt_name`. (#98, #94)
- **A peninsula's outline gets the island bonus.** Nordstrand's outline passed as the canonical
  object of an island but scored 4 instead of 30. (#97)

## Curation & local places

- **Map changes are made at three levels, without a Planetiler fork.** (1) `names/curation.csv`
  `set_tags` fix tags per OSM id in the injector; (2) the tile attributes `frasch:kind`,
  `frasch:minzoom`, `frasch:maxzoom`; (3) style layers per kind in `frasch-bright.json`. OMT drops
  `place=islet` (Habel, Norderoog, Südfall) and Nordstrand has no island polygon in OSM.
  Rejected: forking Planetiler.
- **Label priority is the physical order of the style layers, low priority first.** MapLibre
  resolves label collisions in reverse layer order (the last symbol layer wins), and
  `symbol-sort-key` only orders within a layer.
- **OpenMapTiles sets a label's first zoom by its `place` value; `frasch:minzoom` can only push
  later.** `hamlet` shows from z11, `isolated_dwelling` and `neighbourhood` from z14. So a
  hand-placed warft or settlement is a `hamlet` (as OSM's own Hallig Warften are), and
  `island`/`hallig` get `place=island`. Rejected: `isolated_dwelling` for a warft.
- **In `set_tags` an empty value removes the tag.** `place=` takes the `place` tag off an object,
  a square or a local node, so OpenMapTiles does not label it; a local node without `place` stops
  the build. Rejected: a `hide` column, `maxzoom=0`. (#174)
- **Nordstrand's parts Odebel, Sööderhuuwen, Norderhuuwen, Ängloun and Weesten are `hamlet`
  in curation, as Norder-/Süder-Waygaard are.** OSM has `neighbourhood`, which shows only from z14.
  Sööden, Naidik and Ååsterdik stay neighbourhoods. Mursemkuuch shows once, on its hamlet node,
  so its neighbourhood node loses `place`. (#174)
- **OSM lacks many listed places, so they are local places.** Harden do not exist in OSM, most
  Köge exist only as street or Sielzug names (Gotteskoog has no polder object), about 100 Warften
  only as building or street names, and the vanished Halligen (Bohnen-, Hasen-, Pohns-,
  Schweine-, Gross-, Christianshallig) not at all. Nothing is uploaded to OSM.
- **A place OSM does not have is `local/<slug>`, alone in the `osm` cell, positioned by its
  curation row.** The slug is lowercase ASCII, digits and hyphens (`local/westerheide-amrum`);
  `wikidata` stays empty. The mandatory `lat`/`lon` of the curation row place a new node, or only
  a synthetic square if the row has `polygon_km2`. The build stops on a local reference without
  a curation row, a local curation row without `lat`/`lon`, or a reference used twice. Curation is
  keyed by OSM reference, so a coordinate in the list could take no `set_tags`, zoom window or
  `polygon_km2` (Köge and Harden are areas). Rejected: `lat`/`lon` columns in `places.csv`
  and a coordinate pseudo key, which write the position twice. Cost: two rows per place.
- **A synthetic square belongs to the row's own object.** It is centred on the object if that is
  a node, else on the curation row's `lat`/`lon`, and replaces the object's own place label. The
  row's position (search entry, dialect lookup, `check-tiles`) is the square's centre, so the
  search entry lies where the label is. Rejected: changing OSM, as the foreshore is land. (#168)
- **Oland's square (2.1 km², the coastline polygon's area) is centred on the Sommerdeich ring.**
  The coastline way covers the foreshore along the rail dam, so its interior point lay outside
  the Sommerdeich. (#168)
- **Hamborjer Håli (Hamburger Hallig) is a 0.84 km² square with `frasch:kind=hallig`, from z12,
  centred on 54.5987/8.8165.** The `admin_level=10` boundary includes a ~30 m strip along the dam
  road, which put its interior point 1.3 km east of the Hallig. The centre lies halfway between
  the strip-less centroid, which the salt marsh pulls to the north-eastern edge, and the
  south-western shore. Rejected: changing OSM, as the strip is plausible as a cadastral
  boundary. (#182, #185)
- **`places.csv` stays the single hand-edited source; the dev-only curation view (`?curate`)
  only appends decisions to `names/work/curate-patch.jsonl`.** `curate apply` writes them into
  `places.csv` and, for a local reference, a position row into `curation.csv`. The matcher's
  ownership rule is thus enforced in one place, and a decision against a stale reference is
  refused. Rows are addressed by their stable id, not their names. Rejected: editing the CSV in
  the browser, deciding from `REPORT.md` by hand. (#1, #23)
- **A reference is written whole.** A decision without a Wikidata item clears the row's item, and
  `skip` clears `osm` and `wikidata`. Otherwise an unreviewed auto match stayed beside `skip` and
  read as hand-filled once `skip` was removed. (#97)

## Labels & naming

- **A Frisian dialect view labels with the first present of these tile keys.** In a Frisian
  area: `name:<dialect>`, `frasch:local`, the other Frisian dialects in registry order,
  `name:frr`, `name:nds`, `name:de`, `name:latin`, `name`. Elsewhere `frasch:local` moves behind
  `name:frr`. A Synnejysk or Low Saxon local name thus labels a Frisian view only where no Frisian
  name is known. Map, card and search follow this one chain (`labelChain`). Rejected: `name:frr`
  ahead of the dialect tag, because OSM's ~510 `name:frr` objects mix dialects (Kiel has
  *Kil*). (#83)
- **The local view labels with `frasch:local`, `name:nds`, `name:latin`, `name`, `name:de`.**
  There is deliberately no `name:frr`, which outside a Frisian area is an exonym (Pinneberg is
  *Pinebärj*) or another dialect's name. `name:de` is last, so a feature with only a German name
  stays labelled. Low Saxon is the local majority language everywhere in the tiles. (#32)
- **A place the list has takes its local name from the list alone.** For an object a row claims
  (by reference, by QID, or as member way of its waterway relation) it is the list's `local`, else
  the list's name in the variety of the area. OSM's `name:frr`/`name:nds` is no fallback; it
  showed for listed places (Husum gave *Hüsem*). Listed places without a `local` have no local
  name until one is added, and the local view then shows Low Saxon or German. (#161, #196)
- **An object no row claims takes OSM's own name in the area's language as `frasch:local`.**
  That is `name:frr` in a Frisian area, `name:nds` in a Low Saxon one and none in a Synnejysk
  one: OSM has no `name:jut`, and its Frisian exonym must never become the local name of a
  Synnejysk place. It applies to every object (streets, stations, offices), so a new `name:frr`
  reaches the map with the next tile build. Outside an area `name:frr` is an exonym and not
  used. Rejected: a `within` branch in the style (needs the dialect areas in the browser);
  recording these objects in `osm_objects.json`, which stays the objects of the list. (#81, #83,
  #161)
- **One rule, `placenames.resolve`, gives names and tile keys to the injector and the search
  export.** A label and its search entry thus cannot disagree. `TILE_KEY` is the only place
  that defines the keys the injector writes, and `check-tiles` compares each of its fields and
  the name in every dialect. (#93)
- **The list's German name is the tiles' `name:de`.** The injector writes the first variant of a
  row's `de` on every object the row tags; where the list has none, OSM's stays. Map, card and
  search agree, and the map shows *Norddorf* and *Glücksburg*, not OSM's *Norddorf auf Amrum*
  and *Glücksburg (Ostsee)*. (#61)
- **A tile's `name_de` is not a German name.** OpenMapTiles writes it on every feature, with the
  plain `name` where OSM has no `name:de`, so the card labelled Danish places (Kalleshave)
  "Deutsch". Rejected: reading `name_de` as German. (#104)
- **A generic name in no known language gets no language label.** On the headline it has none,
  elsewhere it is not shown; it is named only where it equals the place's `name:de` or
  `name:da`. Accepted cost: 153 features with `name:nds` and no `name:de` show only the Low Saxon
  name. (#104)
- **For a clicked label, the local name and dialect area come from its own tile feature.** The
  list entry's serve a card opened from search or a link. An entry lies where its first object
  lies, so a second object of the row (Nordwarft: the way on the Hallig, the node in the
  Nordergoesharde) can lie in another area with another local name. (#89, #104)
- **The card of a listed place has no "Frisian (OSM)" line; its headline follows the label.**
  The line showed where OSM's `name:frr` differed from every list name (Bandixwarf). The tiles
  keep `name:frr`. Places the list does not have keep the line. Rejected: ignoring the tile's
  `name:frr` altogether. Known edge: outside Frisian areas `frasch:local` follows `name:frr`, so
  a listed place with only a `local` Frisian name would show OSM's; no row is so (#197). (#161)
- **The card shows every known Low Saxon and Synnejysk name, also when it equals German or the
  headline (Husum, Treen).** The reader sees that the name is the same, not that it is unknown.
  Only the headline's own name gets no line. Rejected: hiding names identical to German. (#83)
- **Where exactly one name is needed, the chain is the Frisian columns, `de`, `da`, then the
  other local varieties (`nds`, `jut`).** It names a created row's id and the curation view's
  label, so a place with only an `nds` or `jut` name is not nameless. Every language counts, but
  Low Saxon and Synnejysk come after German and Danish. `any_name` stays Frisian-only. (#154)
- **Nordstrand is labelled once, as the island, at every zoom.** The island square carries the
  row's names in every dialect; the village node `node/85929111` loses `place` (no settlement is
  called Nordstrand), and its parts show as hamlets beneath, as on Pellworm. A village label from
  z12 had only a Mooring name, so the label switched dialect. Rejected: giving the village node
  the row's other names. (#174, #168)

## Tiles & build

- **The map uses vector tiles in the OpenMapTiles schema, in one PMTiles file, drawn with
  MapLibre GL JS.** Vector tiles let the browser switch dialect without separate tile sets; the
  schema keeps existing OMT styles usable; PMTiles has low lock-in (`pmtiles convert`, `martin`);
  MapLibre gives smooth zoom and rotation. The tile URL is the frontend's only dependency on
  hosting.
- **Planetiler is the stock jar, never a fork; names reach it through the OSM extract.** A
  pre-processing step adds `name:frr-x-mooring` and the other dialect tags to the extract. This
  keeps the name list independent of the tile tool. Rejected: forking Planetiler.
- **Planetiler's name tag lists (`--languages`, `--extra_name_tags`) come from the commands
  `frasch dialects --tags` and `frasch tile-keys`.** A test fails when the injector writes a key
  Planetiler is not told to carry, because such a key would silently miss the tiles.
  (#93)
- **Every build input is pinned by sha256: the Planetiler jar, Natural Earth, the water polygons,
  the lake centerlines.** Two builds of one commit must not differ in their inputs. Natural Earth
  and the water polygons have no versioned upstream URL, so they are mirrored as assets of a
  release of this repository. Rejected: pinning the upstream URLs by hash, because it breaks with
  every upstream update. (#28)
- **Geofabrik extracts are verified by their `.md5`; `SNAPSHOT=yymmdd` builds from a dated
  extract, while the name pipeline stays on `-latest`.** The `-latest` extract changes daily, so
  CI cannot rebuild extract-derived files and only checks their stamps. (#24)
- **`names.json` and the tiles carry a `built_from` stamp (git blob hashes of their inputs), and
  CI checks it.** A committed `names.json` was once built from an uncommitted `places.csv`, and a
  fresh clone could not rebuild it. (#24)
- **Each finished archive is the asset of its own release of this repository, and `web/tiles.lock`
  pins the one the site uses (URL and sha256).** A frontend contributor or CI cannot get a map
  without running the tile pipeline, and a build must record which archive it shipped. The site
  build ships the pinned archive or none and stops at its start when archive or glyphs are
  missing. `public/tiles/` is for the dev server only. Rejected: a warning instead of stopping
  (owner's choice). (#28)
- **`just tiles` links its archive for the dev server only when the file name is the one
  `web/tiles.lock` pins.** The dev server serves only the pinned name; another region's build
  links nothing. `just setup` and `npm run fetch-assets` link the pinned archive again. Rejected:
  a `--reset` flag for `just setup` alone (owner's choice). (#169)
- **A differing `built_from` between the pinned archive and `names.json` only warns, never fails
  the build.** Archives are published on their own, so the pinned one can legitimately be older
  than the names. Rejected: failing the build; a runtime check in the browser. (#108)
- **Every build check runs in one Vite plugin, so a plain `vite build` runs all of them; only
  `tsc -b` stays outside.** The same rule ("what the lock pins is what is on disk") was written
  in TypeScript and bash, and a plain `vite build` skipped the output check. One module reads
  the lock pins, and assets are fetched in Node. Rejected: spawning `tsc -b` inside the plugin
  (owner's choice); a hand-written zip reader or yauzl (`fflate` needs no dependencies). (#108)
- **The site ships only the glyph ranges its labels need, listed in the committed
  `web/fonts.lock`.** The full set is 102 MB per deploy, almost all for scripts no label uses.
  The list is the Latin blocks and punctuation plus what a survey of the archive found. Ä ö ü å
  (Mooring) and đ ā (Sölring) must stay covered (owner's requirement). Rejected: deriving the
  ranges at every build, because a committed list is reviewed in git. (#29)
- **The build requires a glyph range for every code point of every label and stops when the
  pinned archive needs one the list lacks.** MapLibre draws some characters itself without asking
  for their range, but the check does not rely on that internal. With `VITE_TILES_URL` the
  fetched archive is checked if present, else the build warns. (#29)
- **A symbol layer that labels a feature by name must carry `"metadata": {"frasch:label": true}`
  and `["get", "name"]` as its text-field.** `labelLayers` rewrites only marked layers, so an
  unmarked name layer keeps its upstream text-field. The style file names no URL, since
  `styleFor` sets its own; a test enforces this. (#112)
- **Style layers that match nothing in Schleswig-Holstein stay (`highway-shield-us-*`,
  `landcover-glacier`, `landcover-ice-shelf`, `place-continent`, `place-country-*`).** They match
  on the planet build that launch needs. Rejected: removing them as dead code. (#112)
- **The tiles omit the `housenumber` and `mountain_peak` layers (`--exclude_layers`).** No style
  layer reads them (the archive held 279,451 house numbers). A style layer that reads one needs
  the flag removed. (#112)
- **Every square carries the tile attribute `frasch:square=yes`, and `frasch check-tiles` finds
  squares by it.** The square's way id belongs to the injector, so nothing else ties its label to
  its row. Rejected: a reserved id range (Planetiler's node map jumps in ids); reading
  `curation.csv` in check-tiles (a heuristic). (#168)

## Web app

- **The frontend is plain `maplibre-gl` with the `pmtiles` protocol, MiniSearch and i18next with
  one UI file per dialect; there is no search backend.** An in-browser index over a JSON file of
  names is enough for v1. Photon or Pelias come in when the area grows or address search is
  needed. Rejected: `react-map-gl`.
- **UI strings are German placeholders until the owner writes the Mooring ones; tooling never
  invents Mooring strings.** Missing Mooring strings fall back to German. (#5, #7)
- **One dropdown selects the view, and only registry entries with `view=yes` are offered.** A
  dialect without a UI translation cannot be offered. The local view uses the Mooring UI,
  configured in one place. The registry keeps `view=yes|no`, which means exactly "offered as a
  view", and "view" is the one word for the selector concept. (#105)
- **The German dialect names live only in the registry `label`, not in `de.json`.** One source
  for the names. Mooring keeps its own names; a dialect new to the registry needs a name in every
  UI. (#105)
- **The UI language follows the view in a `useLayoutEffect`.** A page opened in a non-default view
  never shows the default's language first. Dev views take the view from the link and write
  neither `?view=` nor `?place=`. (#103)
- **The geolocate button is MapLibre's `GeolocateControl`, with labels from the locale files.**
  MapLibre reads its `locale` option once and cannot change it, and no view switches the UI
  language at runtime, so the button keeps the language the map was created in. Owner's choice:
  no live relabelling. Rejected: special cases for the phone sheet covering the dot or a position
  outside the map's area. (#5, #162)
- **The link is the initial pick: the selection starts with the `?place=` link read at mount,
  before the map writes its own hash.** No card shows until the name list has the entry. A pick by
  hand replaces the pending link for good, and `close` closes a card only. `?place=` is the
  entry's id, or the pick's own reference while the list is unavailable. Owner's decision.
  Rejected: one `useOpenLinked` for the public app and the dev panels. (#103)
- **Old share links keep working: `entryLookup` finds an entry by any of its OSM references, and
  ignores a trailing `#<line>`.** (#104, #112)
- **The browser reads an `osm` cell in one module, and the Wikidata pattern `^Q[0-9]+$` comes
  from the JSON schemas.** Each pattern is typed once per contract; `\d` would accept non-ASCII
  digits, so Python uses `[0-9]+`. The schema's 5 kB in the bundle is accepted. Rejected: pattern
  copies in the code. (#102)

## Tooling & architecture

- **Every `places.csv` row has a stable `id`, and an id never changes.** It is a slug of the
  row's first name along the single-name chain, with `-2`, `-3` on repeats. Matches, curation,
  the search index, `frasch:ref` in tiles and `?place=` links all use it. Line numbers appear only
  in human messages: inserting a row renumbered all below it, which broke links and made
  curation decisions fail after hand edits. Old links and tiles still resolve through the entry's
  `osm` field. `local/` slugs use the same fold (ü becomes u), and the browser folds no name;
  the matcher's transliteration of umlauts is for matching only and is no slug rule. Rejected:
  identification by line number or by kind plus names; a second folding rule. (#23, #102, #154)
- **A position is worked out once, in `frasch.locate`, and committed as `names/osm_objects.json`.**
  The injector and the search export both read it and resolve the dialect the same way. Two separate
  lookups disagreed on Sylt, Amrum, Oland and more. A dialect is assigned per object, not per
  row; an administrative area above municipality level gets none. A row on the map whose object
  the file does not know stops the export and the injector. (#24)
- **The pipeline is defined once, in `frasch/pipeline.py`.** One table gives each generated file
  its inputs, how it is built and when it is stale; `update`, `check-outputs` and `build <name>`
  all use it, so a new generated file is one entry. Recipe, `update` and check each had their
  own answer before and drifted apart. (#95)
- **`just update` runs all post-edit steps in order and skips slow ones whose inputs are
  unchanged.** A pure `just` recipe has no timestamps, so it would rescan the extracts (~8 min)
  every time and could not be tested. It does not build tiles (Java, minutes, not committed) and
  commits nothing. A failing step stops the run, but a decision `curate.py apply` refuses stays
  in the patch and the run goes on (exit 1), so one stale browser decision blocks nothing. (#57)
- **`REPORT.md` stays committed, with a stamp on its last line that CI checks.** The matcher needs
  the git-ignored candidates, so CI cannot rebuild it. After an edit to `places.csv` or
  `dialects.csv`, CI fails until `just update` (or `frasch match`) has rewritten it. (#95)
- **There is one entry point: `uv run just`, and one command, `uv run frasch <command>`, with no
  launcher scripts.** A flag names the same file in every command, a command accepts only the
  flags of the files it works on, and abbreviated options are refused, so a path a command would
  ignore is an error. `just` is a dev dependency because a global one is not pinned by
  `uv.lock`. (#91, #74)
- **`check` runs every check, lists the failures at the end, and CI calls the same composites**
  (`just check-python`, `npm run check`, `npm run smoke`), so local and CI cannot drift. Price:
  CI shows one step per side, so a failure is found in the log. Rejected: stopping at the first
  failure. (#74)
- **`web/` needs no Python.** Its steps live in npm scripts; the `just` recipes only call them.
  (#28, #74)
- **Library code returns what it has to say; only a command's `main()` prints or exits.** Results
  are reports (`ApplyReport`, `ExportReport`, warnings), errors are `ValidationError`s listing
  every problem, and the command prints. Catching everything reported a programming error as
  "lookup failed", so only the expected exception types are caught. (#25, #98, #99, #100)
- **The four hand-edited tables are parsed only by `frasch/tables.py`, and every documented
  column is required.** Four hand-written readers had three behaviours on `;` separators, wrong
  cell counts and line numbers after blank lines. Own extra columns are fine. A problem has one
  shape, `errors.Problem`. (#92)
- **Stale or foreign file formats are refused, not tolerated.** A work file in an older format
  stops the run; no code reads a format that no current version writes. (#96)
- **Files that cross to the web app have a hand-written TypeScript type and a JSON Schema in
  `names/*.schema.json`.** Python tests validate the writer's output, a TypeScript test pins the
  type to the schema, and only `parseNames` validates at run time. The types had drifted
  (`km` nullable in Python only). Rejected: generated types, because of the extra dependency and
  generated file. (#96)
- **The fields of a curation action are stated once, in the patch schema.** One `if`/`then`
  branch per action names the fields it needs and may carry; no hand check repeats them. `allOf`
  rather than `oneOf`, so validators name the broken rule instead of "matches none". Python
  `decisions.ACTIONS` is the one table telling actions apart: a new action is one entry plus its
  schema branch. (#99)
- **The dev server validates patch entries against the same schema with
  `@hyperjump/json-schema`.** Hand checks let `polygon_km2: 0` through. Rejected: Ajv, because of
  its maintenance; `json-schema-library`, which got 1 of 11 probe entries wrong. (#99)
- **Every injector input is required; there are no `--no-areas` or `--no-curation` flags.** A
  build without dialect areas injected for minutes and then died in `frasch provenance`. A tile
  build without them is not wanted. `load_inputs` stops on a missing one before reading the
  extract. The search index still reads an absent `curation.csv` as "no local places". (#100)
- **Every part of the dialect areas, assigned or not, goes through the same polygon union.** The
  union decides where rings start and so which vertices simplifying keeps; skipping it for
  single-polygon parts changed 13 of 71 unassigned municipalities. The build has no tuning
  flags. (#101)
- **Every dev endpoint goes through `vite-plugins/devEndpoints.ts`, with the loopback and
  same-origin gates.** Under `vite --host` the parts file's research notes were readable from
  the local network. A failure answers one 500 pointing to the dev server log, never the error
  text, which may hold an absolute path. (#107)
- **The frontend table of tile keys is hand-written** as a copy of `placenames.TILE_KEY` plus
  OSM's own name keys. Nothing pins it to the Python table, so change both together. (#104)

## Testing

- **No text I/O relies on the locale.** Every text-mode open, `read_text`/`write_text` and
  `subprocess` call names its encoding. `check-python` sets `PYTHONWARNDEFAULTENCODING=1` and
  pytest turns `EncodingWarning` into an error, so CI fails on the next violation. (#111)
- **`areaLayers.test.ts` deliberately checks a registry fact:** it fails when a dialect is added
  without a colour in `DIALECT_COLORS`. It guards against a missing area colour. (#111)
- **The listed scenarios of the place card come from a generated
  `web/src/testing/generated/listedScenarios.json`, not from hand edits.** The fixture extract
  has a place for each; `tiles/tests/test_consistency.py` writes the file and fails when it is
  stale (`--write-scenarios` rewrites it). Rejected: an entry in the pipeline table, because
  scenarios are test data, not a pipeline output. (#109)
