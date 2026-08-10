# Talos Cluster Configuration Repository

Gestionnaire de configuration pour les clusters Kubernetes Talos Linux **bleu-arcanagos** et **bleu-kalecgos**, utilisant `talhelper` pour la gestion déclarative et SOPS pour le chiffrement des secrets.

## Vue d'ensemble

Ce dépôt centralise la configuration des clusters Talos Linux de façon reproductible, versionée et sécurisée. Chaque cluster vit dans son propre répertoire à la racine et possède son README avec les IPs, le stockage et la procédure de bootstrap.

Les clusters sont **mono-nœud** (le control plane accueille aussi les workloads). Les anciens clusters sont conservés dans [`archive/`](archive/).

## Clusters

| Cluster | Type | Hébergeur | Topologie | Endpoint | Documentation |
|---------|------|-----------|-----------|----------|---------------|
| [bleu-arcanagos](bleu-arcanagos/) | Bare metal | Self-hosted (maison) | Mono-nœud | https://192.168.1.12:6443 | [README](bleu-arcanagos/README.md) |
| [bleu-kalecgos](bleu-kalecgos/) | Bare metal | Self-hosted (maison) | Mono-nœud | https://192.168.1.11:6443 | [README](bleu-kalecgos/README.md) |

## Structure du répertoire

```
homelab_talos/
├── README.md                         # Ce fichier — présentation du projet
├── docs/
│   ├── INSTALLATION.md               # Prérequis et installation des outils
│   ├── BOOTSTRAP.md                  # Procédure générique d'installation d'un cluster
│   └── COMMANDES.md                  # Guide des commandes talhelper / talosctl / SOPS
├── .sops.yaml                        # Règles de chiffrement age (global)
├── archive/                          # Anciens clusters (neltharion, ysera, onyxia)
├── bleu-arcanagos/                   # Cluster actif
│   ├── README.md                     # Doc spécifique (IPs, stockage, bootstrap)
│   ├── talconfig.yaml                # Configuration déclarative du cluster
│   ├── talsecret.sops.yaml           # Secrets chiffrés avec SOPS
│   └── clusterconfig/                # Généré par talhelper (gitignored)
│       ├── talosconfig               # Configuration client Talos
│       └── *.yaml                    # Configurations spécifiques aux nœuds
└── bleu-kalecgos/                    # Cluster actif
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
- **[README bleu-arcanagos](bleu-arcanagos/README.md)** — données propres au cluster : IPs, disques, paramètres à substituer.
- **[README bleu-kalecgos](bleu-kalecgos/README.md)** — données propres au cluster : IPs, disques, paramètres à substituer.

## Liens utiles

- [Documentation officielle Talos](https://www.talos.dev/)
- [talhelper GitHub](https://github.com/budimanjaya/talhelper)
- [SOPS GitHub](https://github.com/mozilla/sops)
