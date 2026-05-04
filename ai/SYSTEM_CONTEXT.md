# System Context — cic-network (AI számára)

Olvasd el mielőtt bármit módosítasz.

---

## Mi ez a rendszer?

A `cic-network` a CentralInfraCore **network domain schema repo**.

A cic-primitives leszármazottja — egységes `NetworkResource` ManagedEntity specializáció,
amely platform-agnosztikusan leírja a hálózati erőforrásokat.

**A paradigma (layer2 switch port / layer3 subnet / cloud VPC) az address és adapter rétegben van,
nem a config_surface-ben.** Ez ugyanaz az elvontság mint a ComputeResource-nál (D-010).

---

## Kompozíciós lánc

```
base-repo
  └─[remote: base]─► cic-primitives (@v0.1.2)
                          └─[remote: base]─► cic-network (ez a repo)
```

Git remote = öröklődési lánc. A fájlstruktúra IS az interface contract.

---

## NetworkResource — tervezési alap

**Egységes séma, capability mechanizmussal** (D-001 döntés alapján):

```
NetworkResource
  identity:             kind=NetworkResource, namespace=cic:network
  config_surface:       mtu, vlan_id, cidr, gateway, dhcp_enabled, security_rules, tags
  state_surface:        link_state, assigned_addresses, packet_stats, error_counters
  operation_surface:    enable, disable, flush_mac_table, apply_acl
  notification_surface: link-state-changed, address-assigned, acl-violation
  binding_surface:      {backend}/{provider}/{location}/{id}
```

**backend értékek:**
- `switch` — managed switch port, VLAN, trunk
- `overlay` — OVS/Linux bridge/VXLAN tunnel
- `cloud` — cloud VPC, subnet, security group, route table

**Capability példák:**
- `vlan_tagging` — trunk port, VLAN ID kezelés
- `dhcp` — DHCP server/relay funkció
- `acl` — access control list (ingress/egress rules)
- `nat` — NAT/masquerade (cloud és overlay)
- `bgp` — BGP routing (layer3 switch, cloud TGW)
- `flow_telemetry` — sFlow/NetFlow/IPFIX

---

## Adapter réteg

| Adapter | Backend | Provider példák |
|---|---|---|
| `switch-adapter` | switch | juniper-01, cisco-core, aruba-access |
| `hypervisor-network-adapter` | overlay | ovs-br0, linux-bridge, proxmox-vmbr |
| `cloud-network-adapter` | cloud | aws, gcp, azure, hcloud, do, ovh, oci |

---

## Kapcsolat cic-compute-val

A `ComputeResource.network_interfaces` listában `network` mező hivatkozik egy
`NetworkResource` address-re. A cic-network az első természetes függőség a compute után.

---

## Jelenlegi állapot (2026-05-04)

| Elem | Státusz |
|---|---|
| git bootstrap + primitives/@v0.1.2 merge | **defined** |
| project.yaml + dependency.yaml | **defined** |
| `schemas/domain/network-resource.yaml` | **draft** — séma írás folyamatban |
| adapter contracts | **concept** |
| `make validate` zöld | **pending** — séma befejezése után |
| első signed release | **concept** |
