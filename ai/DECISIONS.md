# Tervezési döntések — cic-network

A fogalmak kialakulásának history-ja.

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
