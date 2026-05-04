# Onboarding (AI) — cic-network

## 1 perc alatt

- **Mi ez:** cic-network — network domain schema repo, cic-primitives leszármazottja
- **Fő séma:** `NetworkResource` — egységes, platform-agnosztikus
- **Paradigma:** layer2/layer3/cloud az adapter+address rétegben, nem a sémában
- **Mérce:** `make validate` — ha nem zöld, semmi sem kész

## Mielőtt bármit írsz

1. `mcp__cic-graph__kb_status` — KB elérhető?
2. Olvasd: `ai/SYSTEM_CONTEXT.md`
3. Nézd: `ai/PROMPTMAP.yaml` — mi a következő konkrét lépés
4. Nézd: `ai/DECISIONS.md` — D-001/D-002/D-003 ismerete kötelező

## A primitive réteg (örökölt, csak olvasható)

| Atom | Kérdés amire válaszol |
|---|---|
| Shape | Milyen mezők, milyen típusok? |
| Role | Config? State? Kulcs? Referencia? |
| Behavior | Milyen műveletek hajthatók végre? |
| Contract | Milyen feltételeknek kell teljesülnie? |
| Address | Hogyan érhető el? |
| Identity | Mi az (típus szinten)? |
| Event | Milyen async jelzést bocsát ki? |

## Ha gond van

Javasolj design-diffet (`ai/DECISIONS.md`-hez), ne térj el csendben.
