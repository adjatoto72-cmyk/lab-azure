



lab azure qui permet de creer des utilisateur et de leur attribuer des licences avec powershell
# Étape 1 : Installer le module
Install-Module Microsoft.Graph -Scope CurrentUser -Force

# Étape 2 : Se connecter
Connect-MgGraph -Scopes "User.ReadWrite.All"

<img width="1059" height="174" alt="Capture d&#39;écran 2026-04-01 202414" src="https://github.com/user-attachments/assets/25aa2804-a1a9-432a-ab4e-e93446e84840" />


# Étape 3 : Récupérer le domaine Azure AD
$tenant = Get-MgOrganization
$domain = $tenant.VerifiedDomains[0].Name
Write-Host "Domaine Azure AD détecté : $domain" -ForegroundColor Green

<img width="782" height="97" alt="Capture d&#39;écran 2026-04-01 202356" src="https://github.com/user-attachments/assets/5397c670-ed6f-460e-9442-23f00066d988" />

# Étape 4 : Créer les 10 utilisateurs
$users = @(
    @{ FirstName = "Jean"; LastName = "Dupont"; Email = "jean.dupont" },
    @{ FirstName = "Marie"; LastName = "Martin"; Email = "marie.martin" },
    @{ FirstName = "Pierre"; LastName = "Bernard"; Email = "pierre.bernard" },
    @{ FirstName = "Luc"; LastName = "Petit"; Email = "luc.petit" },
    @{ FirstName = "Sophie"; LastName = "Roche"; Email = "sophie.roche" },
    @{ FirstName = "Antoine"; LastName = "Lefevre"; Email = "antoine.lefevre" },
    @{ FirstName = "Isabelle"; LastName = "Moreau"; Email = "isabelle.moreau" },
    @{ FirstName = "Thomas"; LastName = "Girard"; Email = "thomas.girard" },
    @{ FirstName = "Nathalie"; LastName = "Laurent"; Email = "nathalie.laurent" },
    @{ FirstName = "Marc"; LastName = "Renault"; Email = "marc.renault" }
)


$createdCount = 0
$errorCount = 0

foreach ($user in $users) {
    try {
        $displayName = "$($user.FirstName) $($user.LastName)"
        $userPrincipalName = "$($user.Email)@$domain"
        $mailNickname = $user.Email.Replace(".", "")
        
        # Générer un mot de passe sécurisé
        $password = -join ((65..90) + (97..122) + (48..57) | Get-Random -Count 16 | ForEach-Object {[char]$_})
        
        Write-Host "Création de l'utilisateur : $displayName..." -ForegroundColor Cyan
        
        # Créer l'utilisateur avec les paramètres nommés
        New-MgUser -DisplayName $displayName -MailNickname $mailNickname -UserPrincipalName $userPrincipalName -AccountEnabled:$true -PasswordProfile @{ ForceChangePasswordNextSignIn = $true; Password = $password }
        
        Write-Host "✓ $displayName créé avec succès (UPN: $userPrincipalName)" -ForegroundColor Green
        $createdCount++
    }
    catch {
        Write-Host "✗ Erreur lors de la création de $displayName : $_" -ForegroundColor Red
        $errorCount++
    }
}
<img width="958" height="302" alt="image" src="https://github.com/user-attachments/assets/86ae8585-0701-462f-913e-aa1710fa2821" /># lab-azure
# Étape 5 : Résumé
Write-Host "`n========== RÉSUMÉ ==========" -ForegroundColor Yellow
Write-Host "Utilisateurs créés : $createdCount" -ForegroundColor Green
Write-Host "Erreurs : $errorCount" -ForegroundColor $(if($errorCount -gt 0) { "Red" } else { "Green" })
Write-Host "=========================" -ForegroundColor Yellow

<img width="1000" height="105" alt="Capture d&#39;écran 2026-04-01 203524" src="https://github.com/user-attachments/assets/234e9433-774b-4a50-9f9b-16c50df21efb" />

# Étape 5 : Créer les 5 groupes
Write-Host "`n========== CRÉATION DES GROUPES ==========" -ForegroundColor Yellow

$groupes = @("Comptabilité", "DSI", "RH", "Finance", "Consultant")
$groupesCrees = @{}

foreach ($groupe in $groupes) {
    try {
        Write-Host "Création du groupe : $groupe..." -ForegroundColor Cyan
        
        $newGroup = New-MgGroup -DisplayName $groupe -MailNickname ($groupe.Replace(" ", "").Replace("é", "e")) -GroupTypes "Unified" -SecurityEnabled:$false
        
        Write-Host "✓ Groupe $groupe créé avec succès" -ForegroundColor Green
        
        $groupesCrees[$groupe] = $newGroup.Id
    }
    catch {
        Write-Host "✗ Erreur lors de la création du groupe $groupe : $_" -ForegroundColor Red
    }
}
# Étape 6 : Ajouter les utilisateurs aux groupes
Write-Host "`n========== AJOUT DES UTILISATEURS AUX GROUPES ==========" -ForegroundColor Yellow

foreach ($user in $createdUsers) {
    try {
        $groupe = $user.Groupe
        $groupId = $groupesCrees[$groupe]
        
        if ($groupId) {
            Write-Host "Ajout de $($user.DisplayName) au groupe $groupe..." -ForegroundColor Cyan
            
            New-MgGroupMember -GroupId $groupId -DirectoryObjectId $user.Id
            
            Write-Host "✓ $($user.DisplayName) ajouté au groupe $groupe" -ForegroundColor Green
        }
    }
    catch {
        Write-Host "✗ Erreur lors de l'ajout de $($user.DisplayName) au groupe $groupe : $_" -ForegroundColor Red
    }
}

# Étape 7 : Résumé
Write-Host "`n========== RÉSUMÉ ==========" -ForegroundColor Yellow
Write-Host "Utilisateurs créés : $createdCount" -ForegroundColor Green
Write-Host "Groupes créés : $($groupesCrees.Count)" -ForegroundColor Green
Write-Host "Erreurs : $errorCount" -ForegroundColor $(if($errorCount -gt 0) { "Red" } else { "Green" })
Write-Host "=========================" -ForegroundColor Yellow

<img width="1108" height="157" alt="Capture d&#39;écran 2026-04-01 210410" src="https://github.com/user-attachments/assets/a44face2-bd2e-4876-8274-cd669aca0bf8" />

Ce script fait :

    ✅ Crée 10 utilisateurs
    ✅ Crée 5 groupes (Comptabilité, DSI, RH, Finance, Consultant)
    ✅ Ajoute automatiquement chaque utilisateur au groupe correspondant
    ✅ Affiche un résumé final

Distribution des utilisateurs :

    Comptabilité : Jean Dupont, Marie Martin
    DSI : Pierre Bernard, Luc Petit
    RH : Sophie Roche, Marc Renault
    Finance : Antoine Lefevre, Isabelle Moreau
    Consultant : Thomas Girard, Nathalie Laurent

