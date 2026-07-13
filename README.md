# Talos Cluster Configuration Repository

Gestionnaire de configuration pour clusters Kubernetes Talos Linux, utilisant `talhelper` pour la gestion déclarative et SOPS pour le chiffrement des secrets.

## Vue d'ensemble

Ce dépôt centralise les configurations de plusieurs clusters Talos Linux, permettant une gestion reproductible, versionée et sécurisée. Chaque cluster est isolé dans son propre répertoire à la racine du dépôt et possède son propre README avec les IPs, le stockage et la procédure de bootstrap.

Tous les clusters sont actuellement **mono-nœud** (le control plane accueille aussi les workloads).

## Clusters

| Cluster | Type | Hébergeur | Topologie | Endpoint | Documentation |
|---------|------|-----------|-----------|----------|---------------|
| [neltharion](neltharion/) | Bare metal | OVH | Mono-nœud | https://5.135.136.115:6443 | [README](neltharion/README.md) |
| [ysera](ysera/) | VPS | OVH | Mono-nœud | https://51.255.205.150:6443 | [README](ysera/README.md) |
| [onyxia](onyxia/) | Bare metal | Self-hosted (maison) | Mono-nœud | https://192.168.1.113:6443 | [README](onyxia/README.md) |
| [kalecgos](kalecgos/) | Bare metal | Self-hosted (maison) | Mono-nœud | https://192.168.1.11:6443 | [README](kalecgos/README.md) |

## Structure du répertoire

```
homelab_talos/
├── README.md                         # Ce fichier — présentation du projet
├── docs/
│   ├── INSTALLATION.md               # Prérequis et installation des outils
│   ├── BOOTSTRAP.md                  # Procédure générique d'installation d'un cluster
│   └── COMMANDES.md                  # Guide des commandes talhelper / talosctl / SOPS
├── .sops.yaml                        # Règles de chiffrement age (global)
└── <cluster>/                        # Un répertoire par cluster
    ├── README.md                     # Doc spécifique (IPs, stockage, bootstrap)
    ├── talconfig.yaml                # Configuration déclarative du cluster
    ├── talsecret.sops.yaml           # Secrets chiffrés avec SOPS
    └── clusterconfig/                # Généré par talhelper (gitignored)
        ├── talosconfig               # Configuration client Talos
        └── *.yaml                    # Configurations spécifiques aux nœuds
```

## Documentation

- **[docs/INSTALLATION.md](docs/INSTALLATION.md)** — Prérequis et installation des outils (talhelper, talosctl, sops, age) et des clés de chiffrement.
- **[docs/BOOTSTRAP.md](docs/BOOTSTRAP.md)** — Procédure générique d'installation d'un cluster depuis zéro.
- **[docs/COMMANDES.md](docs/COMMANDES.md)** — Guide des commandes courantes : génération de config, workflows, gestion des secrets SOPS, débogage et pièges.
- **README de chaque cluster** — données propres au cluster : IPs, disques, paramètres à substituer.

## Liens utiles

- [Documentation officielle Talos](https://www.talos.dev/)
- [talhelper GitHub](https://github.com/budimanjaya/talhelper)
- [SOPS GitHub](https://github.com/mozilla/sops)
