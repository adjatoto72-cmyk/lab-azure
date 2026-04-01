<img width="958" height="302" alt="image" src="https://github.com/user-attachments/assets/86ae8585-0701-462f-913e-aa1710fa2821" /># lab-azure



lab azure qui permet de creer des utilisateur et de leur attribuer des licences avec powershell
# Étape 1 : Installer le module
Install-Module Microsoft.Graph -Scope CurrentUser -Force

# Étape 2 : Se connecter
Connect-MgGraph -Scopes "User.ReadWrite.All"

# Étape 3 : Récupérer le domaine Azure AD
$tenant = Get-MgOrganization
$domain = $tenant.VerifiedDomains[0].Name
Write-Host "Domaine Azure AD détecté : $domain" -ForegroundColor Green 
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

# Étape 5 : Résumé
Write-Host "`n========== RÉSUMÉ ==========" -ForegroundColor Yellow
Write-Host "Utilisateurs créés : $createdCount" -ForegroundColor Green
Write-Host "Erreurs : $errorCount" -ForegroundColor $(if($errorCount -gt 0) { "Red" } else { "Green" })
Write-Host "=========================" -ForegroundColor Yellow
