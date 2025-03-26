```powershell
# Définir le chemin correct du fichier CSV
$CSVPath = "C:\Partage\billu.csv"

# Importer les données CSV
$Users = Import-Csv -Path $CSVPath -Encoding Default

# Définir le domaine Active Directory
$DomainDN = (Get-ADDomain).DistinguishedName

# 1 Créer les OUs basées sur le Département
$Users | ForEach-Object {
    $OUName = $_.Departement
    $OUPath = "OU=$OUName,$DomainDN"

    # Vérifier si l'OU existe, sinon la créer
    if (-not (Get-ADOrganizationalUnit -Filter {Name -eq $OUName})) {
        New-ADOrganizationalUnit -Name $OUName -Path $DomainDN -ProtectedFromAccidentalDeletion $false
        Write-Host "OU créée: $OUName"
    }
}

# 2️ Créer les Groupes basés sur le Service
$Users | ForEach-Object {
    $GroupName = $_.Service
    $OUName = $_.Departement
    $OUPath = "OU=$OUName,$DomainDN"

    # Vérifier si le groupe existe, sinon le créer
    if (-not (Get-ADGroup -Filter {Name -eq $GroupName})) {
        New-ADGroup -Name $GroupName -SamAccountName $GroupName -GroupCategory Security -GroupScope Global -Path $OUPath
        Write-Host "Groupe créé: $GroupName"
    }
}

# 3️ Créer les utilisateurs et les ajouter aux groupes
$Users | ForEach-Object {
    $FirstName = $_.Prenom
    $LastName = $_.Nom
    $UserName = ($FirstName + "." + $LastName).ToLower() # Format: prenom.nom
    $OUName = $_.Departement
    $OUPath = "OU=$OUName,$DomainDN"
    $UserPassword = "héhé" | ConvertTo-SecureString -AsPlainText -Force
    $FullName = "$FirstName $LastName"
    $GroupName = $_.Service

    # Vérifier si l'utilisateur existe déjà
    if (-not (Get-ADUser -Filter {SamAccountName -eq $UserName})) {
        # Création de l'utilisateur AD
        New-ADUser -SamAccountName $UserName -UserPrincipalName "$UserName@$(Get-ADDomain).DnsRoot" `
                   -Name $FullName -GivenName $FirstName -Surname $LastName `
                   -Path $OUPath -AccountPassword $UserPassword -Enabled $true `
                   -ChangePasswordAtLogon $true
        Write-Host "Utilisateur créé: $UserName"

        # Ajouter l'utilisateur au groupe
        Add-ADGroupMember -Identity $GroupName -Members $UserName
        Write-Host "Utilisateur $UserName ajouté au groupe $GroupName"
    }
}
