# Joindre un PC Windows 10 au Domaine mbits.lan

### 1. Prérequis

- Avoir un compte administrateur du domaine (ex: `MBITS\Administrateur`).
- Le PC doit être sur le même réseau que le contrôleur de domaine.
- Le serveur DNS du PC doit pointer vers le contrôleur de domaine.

### 2. Vérification de la Connectivité

Avant de joindre le PC au domaine, vérifier :

```powershell
ping mbits.lan
nslookup mbits.lan
ipconfig /all  # Vérifier le DNS
```

Si la résolution DNS échoue, modifier le DNS dans `ncpa.cpl` (Panneau de configuration > Réseau > IPv4).

---

### 3. Joindre le Domaine via Interface Graphique

1. **Ouvrir les Paramètres Système** : `Win + I` > **Système** > **Informations système**.
2. Cliquer sur **Modifier les paramètres**.
3. Dans l’onglet **Nom de l’ordinateur**, cliquer sur **Modifier**.
4. Sélectionner **Domaine**, entrer `mbits.lan`, puis **OK**.
5. Entrer les identifiants administrateur du domaine.
6. Redémarrer le PC.
7. Se connecter avec un compte du domaine (`MBITS\Utilisateur` ou `Utilisateur@mbits.lan`).

---

### 4. Joindre le Domaine via PowerShell

8. **Ouvrir PowerShell en administrateur**.
9. Vérifier la connexion avec le domaine :
    
    ```powershell
    Test-ComputerSecureChannel -Server "mbits.lan"
    ```
    
10. Joindre le domaine avec :
    
    ```powershell
    Add-Computer -DomainName "mbits.lan" -Credential "MBITS\Administrateur" -Restart
    ```
    
11. Après redémarrage, se connecter avec un compte du domaine.

---

### 5. Vérification Après Adhésion

12. Vérifier que le PC est bien dans le domaine :
    
    ```powershell
    Get-ComputerInfo | Select CsDomain
    ```
    
 ![sortie](/captures/doc_ad_cliwin.png)
 
13. Vérifier que la machine apparaît dans l’Active Directory :
    
    ```powershell
    Get-ADComputer -Filter * | Select Name, DNSHostName
    ```
![sortie](/captures/doc_ad_cliwin2.png)

---
