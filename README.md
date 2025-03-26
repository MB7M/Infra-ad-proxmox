# Infrastructure Active Directory - Proxmox VE

Infrastructure AD complète sur cloud privé avec réplication multi-DC, automatisation du provisioning et intégration multi-OS (Win10, Debian12).

---

## Schéma d’architecture

![Schéma réseau - Proxmox](/captures/schema-ad-proxmox.png)

---

## Architecture

- **Hyperviseur** : Proxmox VE 
- **Réseau** : 172.168.1.0/24 (bridge)
- **Domaine** : MBITS.LAN
- **Machines virtuelles** :
  - 2 DC Windows Server (2022 GUI + Core)
  - 1 client Windows 10
  - 1 serveur Debian 12

---

## Services déployés

- AD DS
- DNS / DHCP
- Kerberos, SSSD (Linux)
- Provisioning PowerShell (avec CSV structuré)

---

## Documentation technique

Accessible dans [`/docs`](./docs) :

- [Installation & Répartition des rôles FSMO](./docs/installation_ad_répartition_fsmo.md)
- [Jointure client Windows au domaine](./docs/Intégration_client_windows10.md)
- [Intégration Linux via SSSD/Kerberos](./docs/intégration_debian.md)

---

## Automatisation du Provisioning

Le script PowerShell permet la création :
- D’unités organisationnelles
- De groupes de sécurité
- De comptes utilisateurs depuis un CSV

→ [Script complet](./scripts/provisioning.md)  
→ [Fichier CSV](./scripts/Users_CSV.xlsx)

---

## Structure du dépôt

```bash
Infra-ad-proxmox/
├── docs/
│   ├── installation_ad_répartition_fsmo.md
│   ├── intégration_client_windows10.md
│   └── intégration_debian.md
├── scripts/
│   ├── provisioning.md
│   └── Users_CSV.xlsx
├── captures/
│   └── schema-reseau-ad.png
├── LICENSE
└── README.md

---

## Objectifs atteints

- Provisioning automatisé AD
- DC redondants avec rôles FSMO distribués
- Services réseau opérationnels (DNS/DHCP)
- Intégration sécurisée des systèmes.

