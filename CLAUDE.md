# cic-network — Claude kontextus

## Nyelvi szabály — KÖTELEZŐ / Language policy — MANDATORY

| Terület / Scope | Szabály / Rule |
|---|---|
| Kommunikáció (AI ↔ fejlesztő) | Magyar / Hungarian |
| Commit üzenetek / Commit messages | **Angol / English** |
| Kód kommentek, docstringek / Code comments, docstrings | **Angol / English** |
| Schema YAML `description` mezők / fields | **Angol / English** |
| Dokumentáció (README, ai/*.md) / Documentation | **Kétnyelvű HU+EN / Bilingual HU+EN** |
| docs/hu/ | Magyar forrás / Hungarian source |
| docs/en/ | Angol fordítás / English translation |

---

## Branch szabály — KÖTELEZŐ

**Érdemi fejlesztés kizárólag a `network/devel` ágon történhet.**

- `network/main` — csak merge fogad (network/devel → network/main), közvetlen commit tilos
- `network/releases/v*` — kizárólag release tag célra
- `network/devel` — ez az aktív fejlesztési ág

Ha nem `network/devel`-en vagyunk: figyelmeztetés, és átváltás `network/devel`-re mielőtt bármilyen
schema, kód vagy dokumentáció változtatás történik.

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
| `network-interface.yaml` | **defined** | RFC 8343, cic-yang building blockok |
| `switch-netconf-adapter` | **defined** | physical + vlan + lldp (NETCONF) |
| `ovs-adapter` | **defined** | logical + vlan + tunnel + ipv4 + ipv6 (OVS) |
| `make validate` zöld | **defined** | Docker-alapú tooling |
| Signed release | **defined** | network/@v0.3.0 — ECDSA + cic_countersign |

---

## Kritikus döntés (D-001)

**D-001 — unified NetworkResource**
Nincs Switch/Subnet/Route/SecurityGroup szétválasztás schema szinten.
A paradigma (layer2/layer3/cloud) az address és adapter rétegben van.
Address: `{backend}/{provider}/{location}/{id}`

---

## Kompozíciós lánc

```
base-repo
    └──► cic-primitives (primitives/@v0.1.5)
              └──► cic-yang (yang/@v0.1.3)
                        └──► cic-network (ez a repo)
```

---

## Mérce

```bash
make validate          # ha nem zöld, semmi sem kész
make release VERSION=  # signed artifact (Vault szükséges)
```
