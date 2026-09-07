# CIC Network — Claude kontextus

## Branch szabály — KÖTELEZŐ

**Érdemi fejlesztés kizárólag a `devel` ágon történhet.**

- `main` — csak merge fogad (devel → main), közvetlen commit tilos
- `network/releases/v*` — kizárólag release tag célra
- `devel` — ez az aktív fejlesztési ág

Ha nem `devel`-en vagyunk: figyelmeztetés, és átváltás `devel`-re mielőtt bármilyen
schema, kód vagy dokumentáció változtatás történik.

## Mi ez a rendszer

A `cic-network` egy **domain-repó** — a `cic-primitives` meta-séma rétegére épülve
hálózati objektumokat (interfészek, VLAN-ok) ír le, schema-szinten.

A `schemas/atomic/`+`schemas/aggregate/` alatti fájlok **öröklöttek** a
`cic-primitives`-ból (a `base` remote-on át) — ez a repó nem definiálja őket,
csak felhasználja. A saját munka a domain-kompozíció:
`schemas/examples/network-interface.yaml`.

A primitívek azok az **irreducibilis szemantikai atomok és kompozícióik**, amelyekből
bármilyen menedzselt objektum strukturált, validálható, verziózott YAML sémává fordítható
— ezt a réteget a `cic-primitives` adja, nem ez a repó.

Részletes architektúra: `ai/SYSTEM_CONTEXT.md`
Következő konkrét feladatok: `ai/PROMPTMAP.yaml`
Tervezési döntések háttere: `ai/DECISIONS.md`

---

## Boot sequence — minden session elején

Mielőtt szakmai kérdésre válaszolsz, végezd el ezt a sorrendet:

1. `mcp__cic-graph__kb_status` — tudásbázis elérhető és friss?
2. Olvasd el: `ai/SYSTEM_CONTEXT.md`
3. Státusz térkép: mi **defined**, mi **draft**, mi **concept**
4. Bridge térkép: hol nincs még séma-szintű megfelelő a fogalomnak

Amíg ez a négy pont nincs meg, ne tegyél tényállításokat a primitive modell állapotáról.

---

## Háromszintű státusz — minden állításhoz kötelező

| Státusz | Jelentés |
|---|---|
| **defined** | YAML séma létezik, `make validate` zöld |
| **draft** | Design megvan írásban, séma még nincs |
| **concept** | Megbeszélt, de formálisan még nincs rögzítve |

## Scaffold térkép (aktuális)

| Elem | Státusz | Megjegyzés |
|---|---|---|
| git repo bootstrap | **defined** | `git merge base@0.5.0` a `cic-primitives`-on át (nem közvetlen) |
| `dependency.yaml` | **defined** | `base@0.5.0` composition lock (a `cic-primitives`-tól örökölt) |
| `project.yaml` | **defined** | `x-cic.repo_type: domain` |
| `schemas/` struktúra | **defined** | atomic/ + aggregate/ (örökölt) + examples/network-interface.yaml (saját) |
| atomic/aggregate réteg | **öröklött** | Shape, Role, Behavior, Contract, Address, Identity, Event, Access + ConfigSurface/StateSurface/OperationSurface/PolicySurface/ManagedEntity |
| `NetworkInterface` domain composition | **defined** | teljes surface-készlet, YANG+RESTCONF+NACM levezetéssel |
| `make validate` zöld | **defined** | Docker-alapú tooling, Vault nélkül is fut |
| signed release | **defined** | `network/@v0.4.1` kiadva, GHCR-en publikálva |

---

## A két szint

```
atomic primitive   = irreducibilis szemantikai atom
                     Shape · Role · Behavior · Contract · Address · Identity · Event · Access
                   → ezekből schema fragment generálható

aggregate primitive = szemantikai kompozíció sealed/defaulted/required slot-okkal
                   → ezek adják a használható tervezési egységeket
                   → aggregate-ből indulunk, nem atomból
```

Az objektum mindig következmény, soha nem kiindulópont.

---

## A kompozíciós mechanizmus

**Git remote = öröklődési lánc.** Nem YAML override rules.

```
base-repo (upstream sablon)
    │  remote: base → git merge base@0.5.0
    └──► cic-primitives
              │  remote: base → git merge base@0.5.0
              └──► cic-network  (ez a repo)  ·  cic-yang, cic-storage, stb. (testvér domain repók)
```

A fájlstruktúra IS az interface contract. A merge konfliktus = séma sértés.

---

## Bridge térkép — hol szakad meg a lánc

```
concept/Shape atom        ──?──  schemas/atomic/shape.yaml
concept/ManagedEntity     ──?──  schemas/aggregate/managed-entity.yaml
concept/git-composition   ──?──  git remote + base@0.5.0 merge
design/project.yaml       ──?──  compiler tooling (repo_type döntés)
```

Ha egy kérdés ilyen pontra mutat: ne mondd, hogy "nincs" — mondd, hogy
**"a fogalom documented, de a séma-szintű megfelelője még nem létezik"**.

---

## Graph-first reasoning (MCP)

MCP kérdéseknél ne `search_query → snippet → válasz` sorrendben dolgozz.

Helyette:
1. Fogalom azonosítás → induló node-ok (`search_nodes`, `find_nodes`)
2. 1–2 hop szomszédok (`neighbors`, `guided_path`)
3. Státusz ellenőrzés (defined/draft/concept)
4. Bridge ellenőrzés (van-e séma-fájl megfelelő)
5. Csak ebből válasz

---

## Reasoning mód

Válasz előtt azonosítsd:

- **immersion**: fogalmak, relációk, a primitive modell logikájának befogadása — ne javasolj implementációt
- **design**: séma struktúra, slot definíciók, kompozíciós szabályok tervezése
- **implementation**: konkrét YAML, séma fájl, Makefile változás

Immersion módban tilos hiányt feltételezni ott, ahol scaffold szándékos.

---

## Kapcsolódó repók

| Repo | Remote | Mit ad |
|---|---|---|
| `cic-primitives` | `base` | atomic/aggregate primitívák, tooling, signing hook, CI, Makefile, mk/infra.mk |
| `base-repo` | közvetett (a `cic-primitives` saját `base` remote-ja) | eredeti tooling-sablon |
| `CIC-Relay` | — | a runtime, ami a `network-interface.yaml` kompozíciót futtatja |

---

## Mérce

```bash
make validate          # séma validáció — ha ez nem zöld, semmi sem kész
make release VERSION=  # signed artifact
```

Lezárási kritérium minden primitive-re:
1. Ebből hogyan lesz séma (YAML)?
2. Ebből hogyan lesz API (RESTCONF / OpenAPI)?
3. Ebből hogyan lesz runtime viselkedés?

Ha mind a három megválaszolható → lezárt.
