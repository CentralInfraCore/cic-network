# cic-network — Claude kontextus

## Mi ez a rendszer

A `cic-network` a CentralInfraCore **network domain schema repo** — a cic-primitives leszármazottja.
Egységes `NetworkResource` séma leírja a hálózati erőforrásokat (switch port, VLAN, subnet, route,
security group, cloud VPC) — capability mechanizmussal, platform-agnosztikusan.

Nem futtat hálózatot. Hordozható domain- és adapter-sémákat definiál.
Implementációt a runtime repók (cic-relay és adapter implementációk) hordoznak.

Részletes architektúra: `ai/SYSTEM_CONTEXT.md`
Tervezési döntések: `ai/DECISIONS.md`
Kötelező szabályok: `ai/MAINTENANCE_CONTRACT.md`

---

## Boot sequence — minden session elején

1. `mcp__cic-graph__kb_status` — KB elérhető és friss?
2. `ai/DECISIONS.md` — D-001 ismerete kötelező mielőtt bármit mondasz
3. `ai/SYSTEM_CONTEXT.md` — teljes network domain kontextus
4. `ai/MAINTENANCE_CONTRACT.md` — mit szabad, mit nem

Amíg ez nincs meg, ne tegyél tényállításokat a network séma állapotáról.

---

## Háromszintű státusz — minden állításhoz kötelező

| Státusz | Jelentés |
|---|---|
| **defined** | YAML séma létezik, `make validate` zöld |
| **draft** | Design megvan, séma még nincs |
| **concept** | Megbeszélt, formálisan nem rögzítve |

---

## Aktuális séma állapot

| Elem | Státusz | Megjegyzés |
|---|---|---|
| `network-resource.yaml` | **draft** | unified, platform-agnosztikus — tervezés alatt |
| `hypervisor-network-adapter.yaml` | **concept** | OVS/Linux bridge contract |
| `cloud-network-adapter.yaml` | **concept** | cloud VPC/subnet/SG contract |
| `switch-adapter.yaml` | **concept** | managed switch (NETCONF/RESTCONF) contract |

---

## Kritikus döntés (D-001)

**D-001 — unified NetworkResource**
Nincs Switch/Subnet/Route/SecurityGroup szétválasztás schema szinten.
A paradigma (layer2/layer3/cloud) az address és adapter rétegben van.
Address: `{backend}/{provider}/{location}/{id}`

---

## Kompozíciós lánc

```
base-repo (upstream sablon)
    │  remote: base → git merge base@0.5.0 (via primitives)
    └──► cic-primitives (primitives/@v0.1.2)
              │  remote: base → git merge primitives/@v0.1.2
              └──► cic-network (ez a repo)
```

---

## Mérce

```bash
make validate          # ha nem zöld, semmi sem kész
make release VERSION=  # signed artifact (Vault szükséges)
```
