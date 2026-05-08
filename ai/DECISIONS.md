# Tervezési döntések — cic-network

A fogalmak kialakulásának history-ja.

---

## D-006 — yang_refs verziókezelési policy (2026-05-08)

**Döntés szükséges** (nyitott, BACKLOG B-001 + B-002)

**Helyzet:**
A `network-interface.yaml` `yang_refs` szekciója az összes cic-yang blokkot `cic-yang@v0.1.0`-ra hivatkozza,
de az ietf-lldp `yang/@v0.1.1`-ben jelent meg (v0.1.0-ban nem létezett).
Emellett a `switch-netconf-adapter.yaml` conformance_matrix-ában:
```yaml
yang_ref: cic-yang/schemas/ietf/ietf-lldp.yaml
```
Ez relatív path, amely `yang/main` HEAD-en nem létező fájlra mutat (csak a releases branch-en van).

**Döntés:**
- `yang_refs`-ben block-onkénti explicit verziót kell megadni (ne legyen egyetlen közös forrás)
- `yang_ref` conformance_matrix bejegyzésekben tag-ankolt formát kell használni:
  `cic-yang@v0.1.1/schemas/ietf/ietf-lldp.yaml`

**Azonnali javítás (B-001, B-002):** a cic-yang@v0.1.0 → v0.1.1 frissítés az lldp blokknál,
és a yang_ref path tag-ankolt formára javítása.

---

## D-005 — NetworkInterface v2: RFC 8343 alapú, cic-yang building blockok (2026-05-06)

**Döntés:** A cic-network v2 az egységes `NetworkInterface` DomainComposition-t
cic-yang building blockok referenciájával definiálja. A v1 `NetworkResource` és
`network-adapter.yaml` archivált.

**Scope (L-001 tanulság alapján):**
- ✓ L2 switching: fizikai switch port, OVS bridge, VLAN, VXLAN tunnel
- ✗ Cloud VPC/subnet — service réteg
- ✗ Routing protokollok (RFC 8349) — service réteg
- ✗ ACL/firewall (RFC 8519) — service réteg

**Adapter séma = cic-yang kompozíció:**
- `switch-netconf-adapter` = ietf-interfaces-physical + ietf-interfaces-vlan
- `ovs-adapter` = ietf-interfaces-logical + ietf-interfaces-vlan + ietf-interfaces-tunnel + ietf-ip-v4/v6

Ami nincs az adapter sémában → schema validation error (conformance by deletion, D-012).

**Backend értékek:** `switch` (fizikai, NETCONF) | `ovs` (szoftveres, OVS-VSCTL)

---

## D-001 — NetworkResource: egységes séma, adapter hordozza a paradigmát (2026-05-04)

**Döntés:** Switch port, Subnet, Route, SecurityGroup különálló DomainComposition-ök
helyett egyetlen `NetworkResource` séma van. A paradigma (layer2/layer3/cloud) az address
és adapter rétegben van, nem a config_surface-ben.

**Miért:** A compute D-010 tapasztalata — az egységes ComputeResource jobb mint a
VM/Physical/Cloud szétválasztás. A hálózatnál ugyanez igaz: egy switch port és egy cloud
subnet szemantikailag ugyanaz (NetworkResource), csak az adapter különböző.
Paradigma lock-in nélkül lehet overlay-ről cloud-ra migrálni.

**Capability mechanizmus:** A séma felsorolja az összes lehetséges state/ops/event mezőt
capability flaggel jelölve. Az adapter deklarálja a binding_surface-ben mit tud kezelni.

**Következmény:** Nem lesz `switch-port.yaml`, `subnet.yaml`, `security-group.yaml` külön.
Egyetlen forrás: `schemas/domain/network-resource.yaml`.

---

## D-002 — link_state enum (2026-05-04)

**Döntés:** A `link_state` enum: `up, down, degraded, testing, unknown`

**Miért:** Minden platform (switch, overlay, cloud) visszaad ilyen állapotot.
- `up`: teljes kapcsolat, forgalom folyik
- `down`: nincs kapcsolat
- `degraded`: részleges hiba (pl. half-duplex, CRC hibák felett küszöbön)
- `testing`: loopback teszt folyamatban
- `unknown`: adapter nem tudja meghatározni

Az adapter normalizálja a platform értékeket
(pl. JUNOS `present`+`up` → `up`, AWS `available` → `up`).

---

## D-004 — Egységes network-adapter contract (2026-05-04)

**Döntés:** Nem lesz 3 különálló adapter contract (switch/hypervisor-network/cloud).
Egyetlen `network-adapter.yaml` — a backend enum (`switch`, `overlay`, `cloud`) szétválasztja
a scope-ot, a capabilities lista szűri le amit az adott implementáció tud.

**Miért:** A contract csak azt mondja meg: observe → state_surface, apply → config_surface,
watch → event stream. Ez igaz mindhárom backendre. A protokoll különbség (NETCONF vs OVS vs
cloud API) az implementáció belügye, nem a contract-é. A binding_surface `known_adapters`
listája már elvégzi a szétválasztást.

**Következmény:** `schemas/adapters/network-adapter.yaml` — 1 fájl, minden backend lefedve.

---

## D-003 — address séma: backend/provider/location/id (2026-05-04)

**Döntés:** A NetworkResource address ugyanolyan struktúrájú mint a ComputeResource:
`{backend}/{provider}/{location}/{id}`

Példák:
- `switch/juniper-01/rack-a/ge-0/0/1`
- `overlay/proxmox-01/-/vmbr0`
- `cloud/aws/eu-central-1/sg-0abc123`

**Miért:** Konzisztens addressing modell a teljes CIC domain rétegben —
a Relay routing logikája nem domain-specifikus.

**Következmény:** A `provider` a routing kulcs (melyik adapter-példány kezeli).
A `location` opcionális — switch portoknál interface path, cloudnál region.
