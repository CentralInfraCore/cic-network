# LLM Szabályok / LLM Rules

## Magyar

- Minden állításhoz jelöld meg: **defined** / **draft** / **concept**
- Ha a státuszt nem tudod fájl-szinten alátámasztani, ne mondd ki tényként
- `schemas/atomic/` és `schemas/aggregate/` upstream (primitives) — ne módosítsd
- Csak `schemas/domain/` és `schemas/adapters/` az írható terület
- Minden döntést rögzíts `ai/DECISIONS.md`-ben D-NNN formátumban
- `make validate` — ha nem zöld, javíts, ne kerüld meg
- Az adapter contract NEM implementáció — interface contract
- A kompozíció mechanizmusa git — ne javasolj YAML override rules rendszert
- MCP kérdéseknél: graph-first, ne snippet-first

---

## English

- Tag every claim as: **defined** / **draft** / **concept**
- If you cannot back up the status at the file level, do not state it as fact
- `schemas/atomic/` and `schemas/aggregate/` are upstream (primitives) — do not modify
- Only `schemas/domain/` and `schemas/adapters/` are writable
- Record every decision in `ai/DECISIONS.md` as D-NNN
- `make validate` — if not green, fix it, do not work around it
- Adapter contract is NOT implementation — it is an interface contract
- The composition mechanism is git — do not propose a YAML override rules system
- For MCP queries: graph-first, not snippet-first
