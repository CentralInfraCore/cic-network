# LLM Rules — cic-network

- Minden állításhoz jelöld meg: **defined** / **draft** / **concept**
- Ha a státuszt nem tudod fájl-szinten alátámasztani, ne mondd ki tényként
- `schemas/atomic/` és `schemas/aggregate/` upstream — ne módosítsd
- Csak `schemas/domain/` és `schemas/adapters/` az írható terület
- Minden döntést rögzíts `ai/DECISIONS.md`-ben D-NNN formátumban
- `make validate` — ha nem zöld, javíts, ne kerüld meg
- Az adapter contract NEM implementáció — interface contract
