# cic-network

> Ez nem klasszikus repo. Ez AI-operált domain schema layer.
> Emberi belépő: ez a README. AI belépő: `ai/ONBOARDING.md`.

A `cic-network` a `cic-primitives` **meta-séma rétegére** épülő domain-repó —
hálózati objektumokat (interfészek, VLAN-ok) ír le, a `cic-primitives` atomic/
aggregate primitíváinak konkrét kompozíciójaként.

Nem a primitíva-réteg maga. A `schemas/atomic/`+`schemas/aggregate/` alatti
fájlok innen származnak (a `base` remote-on át öröklődnek) — ez a repó a
tényleges domain-kompozíciót (`schemas/examples/network-interface.yaml`)
adja hozzájuk.

---

## Két szint

| Szint | Mit képvisel | Hol van | Eredet |
|---|---|---|---|
| **atomic primitive** | 8 irreducibilis atom — Shape, Role, Behavior, Contract, Address, Identity, Event, Access | `schemas/atomic/` | öröklött a `cic-primitives`-ból |
| **aggregate primitive** | Kompozíció sealed/defaulted/required slot-okkal | `schemas/aggregate/` | öröklött a `cic-primitives`-ból |
| **domain composition** | konkrét hálózati objektum (pl. `NetworkInterface`) az aggregate-ekből | `schemas/examples/network-interface.yaml` | **ennek a repónak a saját munkája** |

A domain objektum mindig következmény, soha nem kiindulópont — ez itt a
`NetworkInterface`, ami a `ManagedEntity` aggregate-et alkalmazza a
`cic:network` domain-ra.

---

## Gyors start

```bash
make validate    # séma validáció — ha ez nem zöld, semmi sem kész
make release     # signed artifact (Vault szükséges)
```

---

## AI belépési pontok

| Fájl | Mire való |
|---|---|
| `ai/ONBOARDING.md` | Boot protokoll — minden session elején |
| `ai/MAINTENANCE_CONTRACT.md` | Mit szabad, mit nem, mikor kell döntés |
| `ai/SYSTEM_CONTEXT.md` | Teljes architekturális kontextus |
| `ai/PROMPTMAP.yaml` | Task queue — mi a következő konkrét lépés |
| `ai/DECISIONS.md` | Döntési history — miért úgy van ahogy van |

---

## Aktuális állapot

| Réteg | Státusz | Megjegyzés |
|---|---|---|
| Örökölt atomic/aggregate primitívák | **defined** | `schemas/atomic/`, `schemas/aggregate/` — a `cic-primitives`-ból, `base` remote-on át |
| `NetworkInterface` domain composition | **defined** | `schemas/examples/network-interface.yaml` — teljes surface-készlet (Config/State/Operation/Notification/Policy/Binding) |
| YANG-modul levezetés | **defined** | `derivation_chain.yang` a fenti fájlban — `cic-network-interface` modul |
| RESTCONF útvonalak | **defined** | `derivation_chain.restconf` — collection/instance/config/state/operations/events |
| NACM-analóg policy-leképezés | **defined** | `derivation_chain.nacm` |
| Signed release pipeline | **defined** | Vault Transit + ECDSA, `network/@v0.4.1` kiadva, GHCR-en publikálva |
| KubernetesPod sablon-példa | **öröklött, nem hálózat-specifikus** | `schemas/examples/kubernetes-pod.yaml` — a `cic-primitives` demója, nem ennek a repónak a munkája |
| Több hálózati objektum (route, VLAN mint önálló entitás, ACL) | **not implemented** | jelenleg egyetlen domain composition létezik |
| Production trust-chain | **not implemented** | CIC-Relay + CIC-Schemas feladata |

---

## Kapcsolódó repók

| Repo | Kapcsolat |
|---|---|
| `cic-primitives` | közvetlen upstream — atomic/aggregate primitívák + tooling, `git remote base` |
| `base-repo` | közvetett upstream (a `cic-primitives` saját `base@0.5.0` merge-én keresztül) |
| `CIC-Relay` | runtime — a `network-interface.yaml` kompozíciót futtatja (SNMP-adapter, reconcile loop) |

---

## Release artifact — GHCR

The schema release is available as an OCI artifact in GitHub Container Registry.

**With ORAS:**

```bash
oras pull ghcr.io/centralinfracore/schema/cic-network:v0.4.1-src2026
```

**With curl (no ORAS required):**

```bash
REPO="centralinfracore/schema/cic-network"; TAG="v0.4.1-src2026"; \
TOKEN=$(curl -fsSL "https://ghcr.io/token?scope=repository:${REPO}:pull" | jq -r .token); \
DIGEST=$(curl -fsSL \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Accept: application/vnd.oci.image.manifest.v1+json" \
  "https://ghcr.io/v2/${REPO}/manifests/${TAG}" | jq -r '.layers[0].digest'); \
curl -fL -H "Authorization: Bearer ${TOKEN}" \
  "https://ghcr.io/v2/${REPO}/blobs/${DIGEST}" \
  -o cic-network-v0.4.1.yaml
```
