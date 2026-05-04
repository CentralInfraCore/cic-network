# AI Maintenance Contract — cic-network

---

## Mit szabad

- `schemas/domain/network-resource.yaml` módosítása, ha `make validate` zöld marad
- `schemas/adapters/` bővítése új adapter contract-tal
- `ai/DECISIONS.md` bővítése új döntéssel (D-NNN formátum, dátummal)
- `ai/PROMPTMAP.yaml` státusz frissítése (pending → in_progress → done)
- `dependency.yaml` frissítése ha upstream merge történt

## Mit nem szabad

- `schemas/atomic/` és `schemas/aggregate/` módosítása — upstream (cic-primitives)
- `schemas/index.yaml` módosítása — upstream (cic-primitives)
- `tools/`, `mk/`, `Makefile` módosítása — upstream (base-repo via primitives)
- `make validate` megkerülése — ha piros, javítani kell, nem kihagyni
- Döntés csendes megváltoztatása `ai/DECISIONS.md` bejegyzés nélkül

## Release folyamat

```bash
git checkout -b network/releases/vX.Y.Z
tools/vault-sign-agent.sh -k <developer.key> -c <developer.crt>
export VAULT_ADDR="https://127.0.0.1:18200"
export VAULT_TOKEN=$(cat $XDG_RUNTIME_DIR/vault/sign-token)
export VAULT_SKIP_VERIFY=1
make release
git tag -a "network/@vX.Y.Z" -m "release: X.Y.Z"
```

## Mérce

`make validate` — ha nem zöld, semmi sem kész.

Lezárási kritérium minden adapter contract-ra:
1. Az address mező egyezik a network-resource.yaml address kulcsával?
2. Az observe output mezők mappolnak a state_surface-re?
3. Az operations capability listája szinkronban van a binding_surface-szel?
