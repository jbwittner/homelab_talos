# Cluster kalecgos

Cluster Kubernetes Talos Linux mono-nœud, homelab (réseau local).

## Informations

| Champ | Valeur |
|-------|--------|
| Nom | kalecgos (clusterName `bleu-kalecgos`) |
| Endpoint | https://192.168.1.11:6443 |
| Talos | v1.13.4 |
| Kubernetes | v1.36.2 |
| Nœud | vert-eranikus — control plane |
| IP | 192.168.1.11 |

> Mono-nœud : `allowSchedulingOnControlPlanes: true` — workloads autorisés sur le control plane.

## Stockage

| Disque | Série | Rôle |
|--------|-------|------|
| nvme0n1 | 202512100355 | Système (installDisk) |

## Paramètres

Valeurs à substituer dans la [procédure de bootstrap](../docs/BOOTSTRAP.md) et les [commandes courantes](../docs/COMMANDES.md) :

| Placeholder | Valeur |
|-------------|--------|
| `<IP>` | 192.168.1.11 |
| `<cluster>` | kalecgos |
| `<node-file>` | bleu-kalecgos-vert-eranikus.yaml |
| `<context>` | admin@bleu-kalecgos |
