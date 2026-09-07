# System Context (AI számára)

Olvasd el mielőtt bármit módosítasz.

---

## Mi ez a rendszer?

A `cic-network` egy **domain-repó** a `cic-primitives` (a CentralInfraCore
meta-séma rétege) fölött — hálózati objektumokat ír le, a `cic-primitives`
atomjaiból/aggregate-jeiből komponálva.

Ez a fájl az öröklött primitíva-modellt magyarázza (a `cic-primitives`-ból
származik), hogy a domain-kompozíció (`schemas/examples/network-interface.yaml`)
megírásához legyen kontextus — ez a repó maga nem a meta-séma réteg.

A primitívek két szinten léteznek:

**Atomic primitive** — irreducibilis szemantikai atom. Nem bontható tovább
anélkül, hogy menedzsment-szemantikát veszítene.

| Atom | Mit képvisel |
|---|---|
| Shape | Adat struktúrája — milyen mezők, milyen típusok |
| Role | Mit jelent menedzsment szempontból (config? state? kulcs? referencia?) |
| Behavior | Milyen műveletek hajthatók végre |
| Contract | Milyen feltételeknek kell teljesülnie (kényszer, validáció) |
| Address | Hogyan érhetjük el, hol lakik a rendszerben |
| Identity | Mi az (típusazonosság, nem példányazonosság) |
| Event | Milyen aszinkron jelzéseket képes kibocsátani |

**Aggregate primitive** — szemantikai kompozíció sealed/defaulted/required slot-okkal.
Nem csak atomok listája — kompozíciós objektum contracttal, override pontokkal.

```
ManagedEntity =
  Identity + ConfigSurface + StateSurface +
  OperationSurface + NotificationSurface +
  CapabilitySurface + LifecycleSurface + BindingSurface
```

---

## A kompozíciós mechanizmus

**Git remote = öröklődési lánc.** Ez nem metafora — ez a tényleges megvalósítás.

```
base-repo
  └─[remote: base]─► cic-primitives
                          └─[remote: base]─► cic-network (ez a repo), cic-yang, ...
```

A fájlstruktúra IS az interface contract. Ha a leszármazott repo eltér a
sablon struktúrájától, a `git merge base@0.5.0` konfliktusba megy.
Ez a kényszerítő mechanizmus — nem kell külön validátor.

Választott mechanizmus (döntés: 2026-04-30):
- ✅ Git remote + merge
- ❌ YAML override-rules.yaml (felesleges absztrakciós réteg)

---

## A séma infrastruktúra (örökölt a `cic-primitives`-on, végső soron a `base-repo`-n keresztül)

A `cic-primitives` `base` remote-jából merge-elve elérhetők:

```
tools/compiler.py     CLI: validate, release, get-name
tools/schemalib/      Schema pipeline
  ├── loader.py       YAML betöltés $ref feloldással
  ├── validator.py    Integritás ellenőrzés + jsonschema validálás
  └── artifact.py     Checksum, signing payload, artefaktum összeállítás
tools/releaselib/     Git/Vault service absztrakciók
mk/infra.mk           Makefile include
```

Signing mechanizmus (commit-msg hook):
1. `git write-tree` → staged tree snapshot
2. Determinisztikus tar stream → SHA256 digest
3. Vault Transit: ECDSA SHA256 aláírás
4. X.509 cert (CIC Root CA → fejlesztő)
5. `[signing-metadata]` → commit message

---

## Tervezett könyvtárstruktúra (concept)

```
schemas/
  atomic/
    shape.yaml
    role.yaml
    behavior.yaml
    contract.yaml
    address.yaml
    identity.yaml
    event.yaml
  aggregate/
    managed-entity.yaml
    config-surface.yaml
    state-surface.yaml
    operation-surface.yaml
    notification-surface.yaml
    capability-surface.yaml
    lifecycle-surface.yaml
    binding-surface.yaml
  index.yaml              ← meta-meta-séma (mint CIC-Schemas)
dependencies/             ← signed upstream sémák (template-schema)
source/                   ← ha generált tartalom kell
```

---

## Séma artifact struktúra (template-schema mintára)

Minden primitive YAML fájl:

```yaml
metadata:
  name: <primitive-name>
  version: v0.1.dev
  description: ...
  owner: Gabor Zoltan Sinko
  tags: [primitive, atomic|aggregate]
  validatedBy:
    name: template-schema
    version: v0.9.5_2025

spec:
  # JSON Schema a primitive struktúrájára
  type: object
  required: [...]
  properties:
    ...
```

---

## Kapcsolat más CIC repókkal

| Repo | Kapcsolat | Irány |
|---|---|---|
| `cic-primitives` | git remote `base` | upstream → cic-network (ez a repo) |
| `base-repo` | közvetett (a `cic-primitives` saját `base` remote-ja) | tooling eredete |
| `CIC-Schemas` | referencia minta | signing lánc minta |
| `CIC-Relay` | consumer | a `network-interface.yaml` kompozíció futtatása |

---

## Jelenlegi állapot

**A "Phase 1–7" történet fentebb (git bootstrap, atomic/aggregate réteg,
`make validate` zöld) a `cic-primitives` saját fejlesztési naplója — ez a
repó ezt öröklés útján kapta meg, nem maga hajtotta végre.** A `cic-network`
saját, valódi státusza (2026-09-07-ig): lásd a README "Aktuális állapot"
táblázatát — legfontosabb tétel a `NetworkInterface` domain composition
(`schemas/examples/network-interface.yaml`) és a `network/@v0.4.1` signed
release.

---

## Release folyamat (a `cic-primitives`-tól örökölt tanulságok, `network/*` névtérre igazítva)

```bash
# Előfeltételek
git checkout -b network/releases/vX.Y.Z
tools/vault-sign-agent.sh -k <developer.key> -c <developer.crt>

# Release
export VAULT_ADDR="https://127.0.0.1:18200"
export VAULT_TOKEN=$(cat $XDG_RUNTIME_DIR/vault/sign-token)
export VAULT_SKIP_VERIFY=1
make release

git add project.yaml
git tag -a "network/@vX.Y.Z" -m "release: X.Y.Z"
```

Dockerfile követelmény: `git`, `curl`, `jq`, `yq` + `safe.directory /app`.
