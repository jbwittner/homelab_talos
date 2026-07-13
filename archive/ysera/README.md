# Cluster ysera

Cluster Kubernetes Talos Linux mono-nœud, VPS hébergé chez OVH.

## Informations

| Champ | Valeur |
|-------|--------|
| Nom | ysera |
| Endpoint | https://51.255.205.150:6443 |
| Talos | v1.13.5 |
| Kubernetes | v1.36.1 |
| Nœud | vps-7a5b6065 — control plane |
| IP | 51.255.205.150/24 |
| Interface | eno1 — gateway 51.255.205.254 |

## Paramètres

Valeurs à substituer dans la [procédure de bootstrap](../docs/BOOTSTRAP.md) et les [commandes courantes](../docs/COMMANDES.md) :

| Placeholder | Valeur |
|-------------|--------|
| `<IP>` | 51.255.205.150 |
| `<cluster>` | ysera |
| `<node-file>` | ysera-vps-7a5b6065.yaml |
| `<context>` | admin@ysera |
