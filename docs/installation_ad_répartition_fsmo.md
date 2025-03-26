# Installation et configuration Active Directory avec deux Contrôleurs de Domaine (DC)

## 1. Préparation du projet

- **Nom du domaine** : mbits.lan
- **Serveurs** :
    - **WIN-AD-GUI** (avec interface graphique)
    - **WIN-AD-CORE** (sans interface graphique)
- **Objectif** : Assurer la tolérance aux pannes en configurant deux contrôleurs de domaine avec une répartition optimisée des rôles FSMO.

---

## 2. Configuration réseau (avant installation)

1. **Attribuer des adresses IP fixes** aux deux serveurs.
    
    - Exemple :
        - WIN-AD-GUI : 172.168.1.10
        - WIN-AD-CORE : 172.168.1.20
2. **Configurer le DNS** :
    
    - Utiliser le DNS interne déjà configuré sur le contrôleur de domaine principal.
    - Assurez-vous que les serveurs puissent résoudre les noms :
        
        ```cmd
        nslookup win-ad-core.mbits.lan
        nslookup win-ad-gui.mbits.lan
        ```
        

---

## 3. Installation du rôle AD DS (Active Directory Domain Services)

### **Sur WIN-AD-GUI**

1. Ouvrir le **Gestionnaire de serveur**.
2. Cliquer sur **Ajouter des rôles et fonctionnalités**.
3. Sélectionner le rôle **Services de domaine Active Directory (AD DS)**.
4. Valider et installer le rôle.
5. Promouvoir le serveur en contrôleur de domaine :
    - Choisir **Ajouter une nouvelle forêt**.
    - Nommer le domaine **mbits.lan**.
    - Définir le mot de passe du mode de récupération.
    - Valider l'installation et redémarrer le serveur.

### **Sur WIN-AD-CORE**

6. Se connecter en administrateur local.
7. Ajouter le serveur au domaine Active Directory existant :
    
    ```powershell
    Add-Computer -DomainName "mbits.lan" -Restart
    ```
    
8. Installer le rôle AD DS :
    
    ```powershell
    Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
    ```
    
9. Promouvoir le serveur en contrôleur de domaine secondaire :
    
    ```powershell
    Install-ADDSDomainController -DomainName "mbits.lan"
    ```
    
10. Suivre les étapes de confirmation et redémarrer le serveur.

---

## 4. Vérification de la réplication AD

11. Utiliser la commande suivante pour vérifier la réplication entre les DC :
    
    ```cmd
    repadmin /replsummary
    ```
    
12. Assurer qu'il n'y a pas d'échecs de réplication.

---

## 5. Répartition des rôles FSMO

### **Rôles FSMO et recommandations**

- **Sur WIN-AD-GUI** :
    
    - Maître de schéma (_Schema Master_)
    - Maître des noms de domaine (_Domain Naming Master_)
- **Sur WIN-AD-CORE** :
    
    - Contrôleur de domaine principal (_PDC Emulator_)
    - Gestionnaire de pool RID (_RID Master_)
    - Maître d'infrastructure (_Infrastructure Master_)

### **Transfert des rôles FSMO via PowerShell**

Sur **WIN-AD-CORE**, exécuter la commande suivante :

```powershell
Move-ADDirectoryServerOperationMasterRole -Identity "WIN-AD-CORE" -OperationMasterRole PDCEmulator, RIDMaster, InfrastructureMaster
```

Pour vérifier les rôles :

```powershell
netdom query fsmo
```

![sortie](/captures/fsmo.png)
