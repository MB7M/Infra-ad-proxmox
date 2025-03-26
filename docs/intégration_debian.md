# Intégration d'un serveur Debian dans un domaine Active Directory

## **Objectif**

- Joindre un serveur Debian à un domaine **Active Directory (AD)**.
- Authentifier les utilisateurs AD sur Debian.
- Utiliser **Kerberos** pour sécuriser l’authentification.
- Configurer **SSSD** pour gérer les utilisateurs AD.

---

## **Packages nécessaires**

```bash
apt install -y realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin krb5-user
```

### **Explication des packages :**

- **realmd** → Permet de découvrir et joindre un domaine AD.
- **sssd** → Gère l’authentification des utilisateurs via AD.
- **sssd-tools** → Outils de gestion pour SSSD.
- **libnss-sss & libpam-sss** → Intègrent AD dans les mécanismes de login Unix.
- **adcli** → Outil pour joindre un domaine AD via Kerberos.
- **samba-common-bin** → Contient des outils pour interagir avec AD.
- **krb5-user** → Gestion de l’authentification Kerberos.

---

## **Étape 1 : Vérification des prérequis**

### **Vérifier le DNS**

Le serveur Debian **doit pouvoir résoudre le contrôleur de domaine (DC)** :

```bash
nslookup mbits.lan
nslookup <nom_du_DC>.mbits.lan
```

Si ce n’est pas le cas, configurer **`/etc/resolv.conf`** :

```bash
nano /etc/resolv.conf
```

Ajouter :

```
nameserver <IP_DC>
search mbits.lan
```

Teste avec :

```bash
ping <nom_du_DC>.mbits.lan
```

⚠️ **Erreur fréquente** : Si le DNS est mal configuré, **Debian ne pourra pas rejoindre le domaine**.

---

## **Étape 2 : Découverte et intégration au domaine**

### **Vérifier que le domaine est détecté**

```bash
realm discover mbits.lan
```

On doit voir une sortie confirmant que **le domaine est actif et supporte l’intégration**.

### **Joindre le domaine**

Exécuter cette commande en remplaçant **Administrateur** par un compte ayant les droits de jonction :

```bash
realm join --client-software=sssd --user=Administrateur mbits.lan
```

Saisir le mot de passe administrateur.

⚠️ **Erreur fréquente** : Si le DNS ou **Kerberos** est mal configuré, **realm va échouer**.

### **Vérifier que le serveur Debian est bien ajouté dans Active Directory**

Dans une session **PowerShell sur un DC** :

```powershell
Get-ADComputer -Filter {Name -eq "NomDebian"}
```

 ![sortie](/captures/doc_ad.png)
Si le serveur Debian **apparaît dans l’OU `Computers`**, c’est bon !

---

## **Étape 3 : Configuration de SSSD**

### **Modifier `sssd.conf`**

```bash
nano /etc/sssd/sssd.conf
```

Ajoute **cette configuration correcte** :

```ini
[sssd]
domains = mbits.lan
config_file_version = 2
services = nss, pam, ssh, sudo

[domain/mbits.lan]
id_provider = ad
auth_provider = ad
access_provider = ad
cache_credentials = true
krb5_realm = MBITS.LAN
ad_domain = mbits.lan
use_fully_qualified_names = False
fallback_homedir = /home/%u
default_shell = /bin/bash
enumerate = True
ldap_id_mapping = True
ad_gpo_access_control = disabled
```

### **Explication des paramètres clés :**

- **`id_provider = ad`** → Indique que l'authentification des utilisateurs provient d'Active Directory.
- **`auth_provider = ad`** → Permet d'utiliser AD comme source d'authentification.
- **`access_provider = ad`** → Gère les permissions d'accès basées sur AD.
- **`cache_credentials = true`** → Permet de stocker les identifiants en cache pour les connexions hors ligne.
- **`krb5_realm = MBITS.LAN`** → Définit le royaume Kerberos correspondant au domaine AD.
- **`ad_gpo_access_control = disabled`** → Désactive l’application des GPOs, qui peuvent poser problème sur Debian.
- **`fallback_homedir = /home/%u`** → Évite que Debian crée des home avec des noms non standards (`/home/utilisateur@mbits.lan`).
- **`use_fully_qualified_names = False`** → Permet de se connecter avec `utilisateur` au lieu de `utilisateur@mbits.lan`.

### **Appliquer les permissions correctes**

```bash
chmod 600 /etc/sssd/sssd.conf
chown root:root /etc/sssd/sssd.conf
```

### **Redémarrer SSSD**

```bash
systemctl restart sssd
systemctl enable sssd
systemctl restart realmd
```

### **Vérifier que les utilisateurs AD sont détectés**

```bash
id administrateur@mbits.lan
getent passwd administrateur
```
 ![sortie](/captures/doc_ad_deb.png)
Si l’utilisateur est bien détecté, c’est bon !

---
