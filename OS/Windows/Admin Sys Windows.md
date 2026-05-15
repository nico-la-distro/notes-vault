# Administration système Windows — Guide

> [!info] Prérequis La plupart des commandes PowerShell nécessitent les modules RSAT installés sur le poste d'administration.
> 
> ```powershell
> # Installer tous les outils RSAT (Windows 10/11)
> Get-WindowsCapability -Name RSAT* -Online | Add-WindowsCapability -Online
> 
> # Ou module par module
> Import-Module ActiveDirectory
> Import-Module GroupPolicy
> Import-Module DnsServer
> Import-Module DhcpServer
> ```

---

## Table des matières

- [[#Active Directory — Utilisateurs]]
- [[#Active Directory — Groupes]]
- [[#Active Directory — Unités d'organisation (OU)]]
- [[#Active Directory — Domaine & forêt]]
- [[#Active Directory — GPO]]
- [[#DNS Windows Server]]
- [[#DHCP Windows Server]]
- [[#Stockage & disques]]
- [[#Pare-feu & sécurité Windows]]

---

## Active Directory — Utilisateurs

> [!tip] Module requis : `Import-Module ActiveDirectory`

### Rechercher & consulter

|PowerShell|CMD / outil|Description|
|---|---|---|
|`Get-ADUser -Identity alice`|`dsquery user -name alice`|Informations d'un utilisateur|
|`Get-ADUser alice -Properties *`|`dsget user CN=alice,... -all`|Toutes les propriétés|
|`Get-ADUser -Filter * -SearchBase 'OU=RH,DC=dom,DC=local'`|`dsquery user OU=RH,DC=dom,DC=local`|Tous les utilisateurs d'une OU|
|`Get-ADUser -Filter {Enabled -eq $false}`|`dsquery user -disabled`|Comptes désactivés|
|`Search-ADAccount -LockedOut`|`dsquery user -stalepwd 0`|Comptes verrouillés|
|`Search-ADAccount -AccountExpired`|—|Comptes expirés|
|`Search-ADAccount -PasswordNeverExpires`|`dsquery user -passwordnotrequired`|Mots de passe sans expiration|
|`Get-ADUser -Filter {PasswordLastSet -lt (Get-Date).AddDays(-90)}`|—|Mots de passe non changés depuis 90j|

### Créer & modifier

|PowerShell|CMD / outil|Description|
|---|---|---|
|`New-ADUser -Name 'Bob Martin' -SamAccountName bob -UserPrincipalName bob@dom.local -Path 'OU=RH,DC=dom,DC=local' -AccountPassword (Read-Host -AsSecureString) -Enabled $true`|`dsadd user CN=bob,OU=RH,DC=dom,DC=local -pwd * -mustchpwd yes`|Créer un utilisateur|
|`Set-ADUser alice -Title 'Directrice' -Department 'RH' -Office 'Paris'`|`dsmod user CN=alice,... -title Directrice`|Modifier les attributs|
|`Set-ADAccountPassword alice -Reset -NewPassword (ConvertTo-SecureString 'P@ss!' -AsPlainText -Force)`|`net user alice NouveauMdp /domain`|Réinitialiser le mot de passe|
|`Enable-ADAccount alice`|`net user alice /active:yes /domain`|Activer un compte|
|`Disable-ADAccount alice`|`net user alice /active:no /domain`|Désactiver un compte|
|`Unlock-ADAccount alice`|—|Déverrouiller un compte|
|`Set-ADAccountExpiration alice -DateTime '2025-12-31'`|`net user alice /expires:31/12/2025 /domain`|Définir la date d'expiration|
|`Clear-ADAccountExpiration alice`|`net user alice /expires:never /domain`|Supprimer la date d'expiration|
|`Move-ADObject 'CN=alice,...' -TargetPath 'OU=Finance,...'`|`dsmove CN=alice,... -newparent OU=Finance,...`|Déplacer vers une autre OU|
|`Remove-ADUser alice -Confirm:$false`|`dsrm CN=alice,OU=RH,DC=dom,DC=local`|Supprimer un utilisateur|

### Import en masse depuis CSV

```powershell
# Structure CSV : Prenom,Nom,Login,OU,Departement
Import-Csv users.csv | ForEach-Object {
    $mdp = ConvertTo-SecureString "Bienvenue01!" -AsPlainText -Force
    New-ADUser `
        -GivenName      $_.Prenom `
        -Surname        $_.Nom `
        -Name           "$($_.Prenom) $($_.Nom)" `
        -SamAccountName $_.Login `
        -UserPrincipalName "$($_.Login)@dom.local" `
        -Path           "OU=$($_.OU),DC=dom,DC=local" `
        -Department     $_.Departement `
        -AccountPassword $mdp `
        -ChangePasswordAtLogon $true `
        -Enabled        $true
}
```

---

## Active Directory — Groupes

> [!tip] Module requis : `Import-Module ActiveDirectory`

|PowerShell|CMD / outil|Description|
|---|---|---|
|`Get-ADGroup -Filter * -SearchBase 'OU=Groupes,DC=dom,DC=local'`|`dsquery group OU=Groupes,...`|Lister les groupes d'une OU|
|`Get-ADGroup -Filter {GroupScope -eq 'Universal'}`|`dsquery group -scope u`|Groupes universels|
|`Get-ADGroupMember -Identity 'GRP-RH'`|`net group GRP-RH /domain`|Membres d'un groupe|
|`Get-ADGroupMember -Identity 'GRP-RH' -Recursive`|—|Membres récursifs (groupes imbriqués)|
|`Get-ADPrincipalGroupMembership alice`|`dsget user CN=alice,... -memberof`|Groupes d'un utilisateur|
|`New-ADGroup -Name 'GRP-Finance' -GroupScope Global -GroupCategory Security -Path 'OU=Groupes,...'`|`dsadd group CN=GRP-Finance,... -scope g -secgrp yes`|Créer un groupe de sécurité global|
|`New-ADGroup -Name 'GRP-Distribution' -GroupScope Universal -GroupCategory Distribution -Path 'OU=Groupes,...'`|`dsadd group CN=GRP-Distribution,... -scope u -secgrp no`|Créer un groupe de distribution|
|`Add-ADGroupMember 'GRP-Finance' -Members alice,bob`|`net group GRP-Finance alice /add /domain`|Ajouter des membres|
|`Remove-ADGroupMember 'GRP-Finance' -Members alice -Confirm:$false`|`net group GRP-Finance alice /delete /domain`|Retirer des membres|
|`Remove-ADGroup 'GRP-Finance' -Confirm:$false`|`dsrm CN=GRP-Finance,...`|Supprimer un groupe|

### Portées et catégories de groupes

|Type|Portée|Usage|
|---|---|---|
|`DomainLocal`|Ressources du domaine local|Droits d'accès aux ressources|
|`Global`|Domaine courant, visible partout|Regrouper des utilisateurs du même domaine|
|`Universal`|Toute la forêt|Accès multi-domaines|
|`Security`|—|Affecter des permissions|
|`Distribution`|—|Listes de diffusion e-mail|

---

## Active Directory — Unités d'organisation (OU)

> [!tip] Module requis : `Import-Module ActiveDirectory`

|PowerShell|CMD / outil|Description|
|---|---|---|
|`Get-ADOrganizationalUnit -Filter * \| Select-Object Name,DistinguishedName`|`dsquery ou`|Lister toutes les OU|
|`Get-ADOrganizationalUnit -Filter * -SearchBase 'DC=dom,DC=local' \| Sort-Object DistinguishedName`|`dsquery ou -scope subtree`|OU de tout le domaine triées|
|`New-ADOrganizationalUnit -Name 'Finance' -Path 'DC=dom,DC=local'`|`dsadd ou OU=Finance,DC=dom,DC=local`|Créer une OU à la racine|
|`New-ADOrganizationalUnit -Name 'Paris' -Path 'OU=Finance,DC=dom,DC=local'`|`dsadd ou OU=Paris,OU=Finance,DC=dom,DC=local`|Créer une OU imbriquée|
|`Set-ADOrganizationalUnit 'OU=Finance,...' -ProtectedFromAccidentalDeletion $true`|—|Protéger contre la suppression accidentelle|
|`Set-ADOrganizationalUnit 'OU=Finance,...' -Description 'Département Finance'`|`dsmod ou OU=Finance,... -desc "Département Finance"`|Modifier la description|
|`Remove-ADOrganizationalUnit 'OU=Finance,...' -Confirm:$false`|`dsrm OU=Finance,DC=dom,DC=local`|Supprimer une OU|

> [!warning] Suppression d'OU Si la protection contre la suppression accidentelle est activée, désactiver d'abord :
> 
> ```powershell
> Set-ADOrganizationalUnit 'OU=Finance,DC=dom,DC=local' -ProtectedFromAccidentalDeletion $false
> Remove-ADOrganizationalUnit 'OU=Finance,DC=dom,DC=local' -Confirm:$false
> ```

---

## Active Directory — Domaine & forêt

|PowerShell|CMD / outil|Description|
|---|---|---|
|`Get-ADDomain`|`netdom query /domain`|Informations du domaine courant|
|`Get-ADForest`|—|Informations sur la forêt|
|`Get-ADDomainController -Filter *`|`netdom query dc`|Lister les contrôleurs de domaine|
|`Get-ADDomainController -Discover -Service PrimaryDC`|`netdom query fsmo`|Trouver le PDC Emulator|
|`netdom query fsmo`|`netdom query fsmo`|Rôles FSMO (maîtres d'opérations)|
|`Get-ADTrust -Filter *`|`netdom query trust`|Relations d'approbation|
|`Test-ComputerSecureChannel -Repair`|`netdom resetpwd /server:DC1 /ud:admin /pd:*`|Réparer le canal sécurisé avec le domaine|
|`Add-Computer -DomainName dom.local -Credential (Get-Credential) -Restart`|`netdom join %COMPUTERNAME% /domain:dom.local /ud:admin /pd:*`|Joindre une machine au domaine|
|`Remove-Computer -UnjoinDomainCredential (Get-Credential) -Restart -Force`|`netdom remove %COMPUTERNAME% /domain:dom.local /ud:admin /pd:*`|Retirer une machine du domaine|
|`Get-ADObject -Filter {isDeleted -eq $true} -IncludeDeletedObjects`|—|Objets supprimés (corbeille AD)|
|`Restore-ADObject -Identity <GUID>`|—|Restaurer un objet depuis la corbeille AD|

---

## Active Directory — GPO

> [!tip] Module requis : `Import-Module GroupPolicy`

### Consulter

|PowerShell|CMD / outil|Description|
|---|---|---|
|`Get-GPO -All`|`gpresult /r`|Lister toutes les GPO|
|`Get-GPO -Name 'Politique-Bureau'`|—|Détails d'une GPO|
|`Get-GPOReport -Name 'Politique-Bureau' -ReportType HTML -Path rapport.html`|`gpresult /h rapport.html`|Rapport HTML d'une GPO|
|`Get-GPResultantSetOfPolicy -ReportType HTML -Path rsop.html`|`gpresult /h rsop.html`|Stratégie résultante (RSoP)|
|`gpresult /r /scope computer`|`gpresult /r /scope computer`|RSoP machine uniquement|
|`gpresult /r /scope user`|`gpresult /r /scope user`|RSoP utilisateur uniquement|
|`Get-GPInheritance -Target 'OU=RH,DC=dom,DC=local'`|—|Héritage des GPO sur une OU|
|`Get-GPLink -Target 'OU=RH,DC=dom,DC=local'`|—|GPO liées à une OU|

### Créer & lier

|PowerShell|CMD / outil|Description|
|---|---|---|
|`New-GPO -Name 'Politique-Bureau' -Comment 'Bureau RH'`|—|Créer une GPO vide|
|`New-GPLink -Name 'Politique-Bureau' -Target 'OU=RH,DC=dom,DC=local'`|—|Lier une GPO à une OU|
|`New-GPLink -Name 'Politique-Bureau' -Target 'OU=RH,...' -LinkEnabled Yes -Order 1`|—|Lier avec priorité et activation|
|`Set-GPLink -Name 'Politique-Bureau' -Target 'OU=RH,...' -Enforced Yes`|—|Forcer (No Override)|
|`Set-GPLink -Name 'Politique-Bureau' -Target 'OU=RH,...' -LinkEnabled No`|—|Désactiver le lien sans supprimer|
|`Set-GPInheritance -Target 'OU=RH,...' -IsBlocked Yes`|—|Bloquer l'héritage sur une OU|
|`Remove-GPLink -Name 'Politique-Bureau' -Target 'OU=RH,...'`|—|Supprimer un lien (sans supprimer la GPO)|
|`Remove-GPO -Name 'Politique-Bureau' -Confirm:$false`|—|Supprimer une GPO|

### Configurer des paramètres via GPO

```powershell
# Définir une valeur de registre via GPO
Set-GPRegistryValue -Name 'Politique-Bureau' `
    -Key 'HKLM\Software\Policies\MonApp' `
    -ValueName 'ParametreX' `
    -Type DWord `
    -Value 1

# Supprimer une valeur de registre via GPO
Remove-GPRegistryValue -Name 'Politique-Bureau' `
    -Key 'HKLM\Software\Policies\MonApp' `
    -ValueName 'ParametreX'

# Lire une valeur configurée dans une GPO
Get-GPRegistryValue -Name 'Politique-Bureau' `
    -Key 'HKLM\Software\Policies\MonApp'
```

### Appliquer, sauvegarder & restaurer

|PowerShell|CMD / outil|Description|
|---|---|---|
|`Invoke-GPUpdate -Computer PC01 -Force`|`gpupdate /force`|Forcer l'application des GPO|
|`Invoke-GPUpdate -RandomDelayInMinutes 0`|`gpupdate /force /wait:0`|Appliquer sans délai aléatoire|
|`Backup-GPO -Name 'Politique-Bureau' -Path 'C:\GPO-Backup'`|—|Sauvegarder une GPO|
|`Backup-GPO -All -Path 'C:\GPO-Backup'`|—|Sauvegarder toutes les GPO|
|`Restore-GPO -Name 'Politique-Bureau' -Path 'C:\GPO-Backup'`|—|Restaurer une GPO|
|`Import-GPO -BackupId {GUID} -TargetName 'Nouvelle-GPO' -Path 'C:\GPO-Backup'`|—|Importer une GPO depuis une sauvegarde|
|`Copy-GPO -SourceGpoName 'Modele' -TargetName 'Copie-Modele'`|—|Dupliquer une GPO|

---

## DNS Windows Server

> [!tip] Module requis : `Import-Module DnsServer`

### Zones

|PowerShell|CMD / dnscmd|Description|
|---|---|---|
|`Get-DnsServerZone`|`dnscmd /enumzones`|Lister toutes les zones|
|`Get-DnsServerZone -Name 'dom.local'`|`dnscmd /zoneinfo dom.local`|Détails d'une zone|
|`Add-DnsServerPrimaryZone -Name 'dom.local' -ReplicationScope 'Forest'`|`dnscmd /zoneadd dom.local /primary /dp /forest`|Créer une zone primaire intégrée AD|
|`Add-DnsServerPrimaryZone -Name 'dom.local' -ZoneFile 'dom.local.dns'`|`dnscmd /zoneadd dom.local /primary /file dom.local.dns`|Créer une zone primaire (fichier)|
|`Add-DnsServerSecondaryZone -Name 'dom.local' -ZoneFile 'dom.local.dns' -MasterServers 192.168.1.1`|`dnscmd /zoneadd dom.local /secondary 192.168.1.1`|Créer une zone secondaire|
|`Add-DnsServerStubZone -Name 'ext.local' -MasterServers 10.0.0.1 -ReplicationScope Forest`|`dnscmd /zoneadd ext.local /stub 10.0.0.1 /dp /forest`|Créer une zone stub|
|`Add-DnsServerConditionalForwarderZone -Name 'partenaire.com' -MasterServers 10.1.1.1`|`dnscmd /zoneadd partenaire.com /forwarder 10.1.1.1`|Zone de redirecteur conditionnel|
|`Set-DnsServerPrimaryZone -Name 'dom.local' -DynamicUpdate Secure`|`dnscmd /config dom.local /allowupdate 2`|Activer les mises à jour dynamiques sécurisées|
|`Remove-DnsServerZone -Name 'dom.local' -Force`|`dnscmd /zonedelete dom.local /f`|Supprimer une zone|

### Redirecteurs

|PowerShell|CMD / dnscmd|Description|
|---|---|---|
|`Get-DnsServerForwarder`|`dnscmd /info /forwarders`|Afficher les redirecteurs|
|`Add-DnsServerForwarder -IPAddress 8.8.8.8,1.1.1.1`|`dnscmd /resetforwarders 8.8.8.8 1.1.1.1`|Définir les redirecteurs DNS|
|`Remove-DnsServerForwarder -IPAddress 8.8.8.8`|—|Supprimer un redirecteur|
|`Set-DnsServerForwarder -UseRootHint $true`|`dnscmd /config /userecursion 1`|Utiliser les racines si redirecteurs indisponibles|

### Enregistrements (Records)

|PowerShell|CMD / dnscmd|Description|
|---|---|---|
|`Get-DnsServerResourceRecord -ZoneName 'dom.local'`|`dnscmd /enumrecords dom.local @`|Tous les enregistrements d'une zone|
|`Get-DnsServerResourceRecord -ZoneName 'dom.local' -Name 'srv01'`|`nslookup srv01.dom.local`|Enregistrements d'un hôte|
|`Add-DnsServerResourceRecordA -ZoneName 'dom.local' -Name 'srv01' -IPv4Address '192.168.1.10'`|`dnscmd /recordadd dom.local srv01 A 192.168.1.10`|Créer un enregistrement A (IPv4)|
|`Add-DnsServerResourceRecordAAAA -ZoneName 'dom.local' -Name 'srv01' -IPv6Address 'fe80::1'`|`dnscmd /recordadd dom.local srv01 AAAA fe80::1`|Créer un enregistrement AAAA (IPv6)|
|`Add-DnsServerResourceRecordCName -ZoneName 'dom.local' -Name 'www' -HostNameAlias 'srv01.dom.local.'`|`dnscmd /recordadd dom.local www CNAME srv01.dom.local.`|Créer un CNAME (alias)|
|`Add-DnsServerResourceRecordMX -ZoneName 'dom.local' -Name '@' -MailExchange 'mail.dom.local.' -Preference 10`|`dnscmd /recordadd dom.local @ MX 10 mail.dom.local.`|Créer un enregistrement MX|
|`Add-DnsServerResourceRecordPtr -ZoneName '1.168.192.in-addr.arpa' -Name '10' -PtrDomainName 'srv01.dom.local.'`|`dnscmd /recordadd 1.168.192.in-addr.arpa 10 PTR srv01.dom.local.`|Créer un PTR (résolution inverse)|
|`Add-DnsServerResourceRecordTxt -ZoneName 'dom.local' -Name '@' -DescriptiveText 'v=spf1 mx -all'`|`dnscmd /recordadd dom.local @ TXT "v=spf1 mx -all"`|Créer un enregistrement TXT (SPF…)|
|`Remove-DnsServerResourceRecord -ZoneName 'dom.local' -Name 'srv01' -RRType 'A' -Force`|`dnscmd /recorddelete dom.local srv01 A /f`|Supprimer un enregistrement|

### Cache & diagnostics

|PowerShell|CMD / outil|Description|
|---|---|---|
|`Clear-DnsServerCache`|`dnscmd /clearcache`|Vider le cache DNS serveur|
|`Show-DnsServerCache`|`dnscmd /enumcache`|Afficher le cache DNS serveur|
|`Clear-DnsClientCache`|`ipconfig /flushdns`|Vider le cache DNS client|
|`Resolve-DnsName srv01.dom.local`|`nslookup srv01.dom.local`|Résoudre un nom DNS|
|`Resolve-DnsName srv01.dom.local -Server 192.168.1.1`|`nslookup srv01 192.168.1.1`|Résoudre via un serveur spécifique|
|`Resolve-DnsName dom.local -Type MX`|`nslookup -type=MX dom.local`|Résoudre un type d'enregistrement précis|
|`Test-DnsServer -IPAddress 192.168.1.1 -ZoneName 'dom.local'`|—|Tester la santé d'un serveur DNS|

---

## DHCP Windows Server

> [!tip] Module requis : `Import-Module DhcpServer`

### Étendues (Scopes)

|PowerShell|CMD / netsh|Description|
|---|---|---|
|`Get-DhcpServerv4Scope`|`netsh dhcp server show scope`|Lister les étendues DHCPv4|
|`Get-DhcpServerv4Scope -ScopeId 192.168.1.0`|`netsh dhcp server scope 192.168.1.0 show state`|État d'une étendue|
|`Add-DhcpServerv4Scope -Name 'LAN-Bureau' -StartRange 192.168.1.100 -EndRange 192.168.1.200 -SubnetMask 255.255.255.0 -State Active`|`netsh dhcp server add scope 192.168.1.0 255.255.255.0 LAN-Bureau`|Créer une étendue|
|`Set-DhcpServerv4Scope -ScopeId 192.168.1.0 -State Active`|`netsh dhcp server scope 192.168.1.0 set state 1`|Activer une étendue|
|`Set-DhcpServerv4Scope -ScopeId 192.168.1.0 -State InActive`|`netsh dhcp server scope 192.168.1.0 set state 0`|Désactiver une étendue|
|`Set-DhcpServerv4Scope -ScopeId 192.168.1.0 -LeaseDuration 1.00:00:00`|—|Durée du bail (1 jour)|
|`Remove-DhcpServerv4Scope -ScopeId 192.168.1.0 -Force`|`netsh dhcp server delete scope 192.168.1.0 dhcpfullforce`|Supprimer une étendue|

### Options & exclusions

|PowerShell|CMD / netsh|Description|
|---|---|---|
|`Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 -Router 192.168.1.1`|`netsh dhcp server scope 192.168.1.0 set optionvalue 3 IPADDRESS 192.168.1.1`|Passerelle par défaut (option 3)|
|`Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 -DnsServer 192.168.1.10,8.8.8.8`|`netsh dhcp server scope 192.168.1.0 set optionvalue 6 IPADDRESS 192.168.1.10`|Serveurs DNS (option 6)|
|`Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 -DnsDomain 'dom.local'`|`netsh dhcp server scope 192.168.1.0 set optionvalue 15 STRING dom.local`|Suffixe DNS (option 15)|
|`Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 -WinsServer 192.168.1.20`|`netsh dhcp server scope 192.168.1.0 set optionvalue 44 IPADDRESS 192.168.1.20`|Serveur WINS (option 44)|
|`Add-DhcpServerv4ExclusionRange -ScopeId 192.168.1.0 -StartRange 192.168.1.100 -EndRange 192.168.1.110`|`netsh dhcp server scope 192.168.1.0 add excluderange 192.168.1.100 192.168.1.110`|Plage d'exclusion|
|`Remove-DhcpServerv4ExclusionRange -ScopeId 192.168.1.0 -StartRange 192.168.1.100 -EndRange 192.168.1.110`|`netsh dhcp server scope 192.168.1.0 delete excluderange 192.168.1.100 192.168.1.110`|Supprimer une exclusion|

### Réservations & baux

|PowerShell|CMD / netsh|Description|
|---|---|---|
|`Get-DhcpServerv4Lease -ScopeId 192.168.1.0`|`netsh dhcp server scope 192.168.1.0 show clients`|Baux actifs d'une étendue|
|`Get-DhcpServerv4Lease -ScopeId 192.168.1.0 \| Where-Object AddressState -eq 'Active'`|—|Baux actifs seulement|
|`Add-DhcpServerv4Reservation -ScopeId 192.168.1.0 -IPAddress 192.168.1.50 -ClientId 'AA-BB-CC-DD-EE-FF' -Name 'Imprimante'`|`netsh dhcp server scope 192.168.1.0 add reservedip 192.168.1.50 AABBCCDDEEFF Imprimante`|Créer une réservation IP|
|`Remove-DhcpServerv4Reservation -ScopeId 192.168.1.0 -IPAddress 192.168.1.50`|`netsh dhcp server scope 192.168.1.0 delete reservedip 192.168.1.50`|Supprimer une réservation|
|`Remove-DhcpServerv4Lease -ScopeId 192.168.1.0 -IPAddress 192.168.1.150`|—|Libérer un bail|

### Autorisation & failover

|PowerShell|CMD / netsh|Description|
|---|---|---|
|`Add-DhcpServerInDC -DnsName 'SRV-DHCP.dom.local' -IPAddress 192.168.1.5`|`netsh dhcp server initiate auth`|Autoriser le serveur DHCP dans AD|
|`Get-DhcpServerInDC`|`netsh dhcp show server`|Serveurs DHCP autorisés dans AD|
|`Remove-DhcpServerInDC -DnsName 'SRV-DHCP.dom.local'`|—|Retirer l'autorisation|
|`Add-DhcpServerv4Failover -Name 'FO-LAN' -PartnerServer SRV2 -ScopeId 192.168.1.0 -Mode HotStandby -ServerRole Active`|—|Configurer le basculement DHCP|
|`Get-DhcpServerv4Failover`|—|État du failover DHCP|

### Sauvegarde & restauration

```powershell
# Sauvegarder la configuration DHCP
Backup-DhcpServer -ComputerName SRV-DHCP -Path 'C:\DHCP-Backup'

# Restaurer
Restore-DhcpServer -ComputerName SRV-DHCP -Path 'C:\DHCP-Backup'

# Exporter vers XML (migration)
Export-DhcpServer -ComputerName SRV-DHCP -File 'C:\dhcp-export.xml' -Force

# Importer depuis XML
Import-DhcpServer -ComputerName SRV-DHCP2 -File 'C:\dhcp-export.xml' -BackupPath 'C:\DHCP-Backup' -Force
```

---

## Stockage & disques

> [!tip] Module requis : Storage (intégré à Windows depuis PowerShell 4+)

### Disques physiques

|PowerShell|diskpart / CMD|Description|
|---|---|---|
|`Get-Disk`|`diskpart > list disk`|Lister les disques physiques|
|`Get-Disk \| Where-Object PartitionStyle -eq 'RAW'`|—|Disques non initialisés|
|`Get-Disk \| Where-Object IsOffline -eq $true`|—|Disques hors ligne|
|`Set-Disk -Number 1 -IsOffline $false`|`diskpart > select disk 1 > online disk`|Mettre un disque en ligne|
|`Initialize-Disk -Number 1 -PartitionStyle GPT`|`diskpart > select disk 1 > convert gpt`|Initialiser un disque (GPT)|
|`Initialize-Disk -Number 1 -PartitionStyle MBR`|`diskpart > select disk 1 > convert mbr`|Initialiser un disque (MBR)|
|`Get-PhysicalDisk`|—|Disques physiques (état, bus, média)|
|`Get-PhysicalDisk \| Where-Object HealthStatus -ne 'Healthy'`|—|Disques avec problèmes de santé|

> [!tip] GPT vs MBR Préférer **GPT** pour tous les disques modernes (> 2 To, UEFI). MBR est limité à 2 To et 4 partitions primaires.

### Partitions

|PowerShell|diskpart / CMD|Description|
|---|---|---|
|`Get-Partition`|`diskpart > list partition`|Toutes les partitions|
|`Get-Partition -DiskNumber 1`|`diskpart > select disk 1 > list partition`|Partitions d'un disque|
|`New-Partition -DiskNumber 1 -UseMaximumSize -AssignDriveLetter`|`diskpart > select disk 1 > create partition primary > assign`|Créer une partition sur tout l'espace|
|`New-Partition -DiskNumber 1 -Size 50GB -DriveLetter E`|`diskpart > create partition primary size=51200 > assign letter=E`|Créer une partition de taille précise|
|`Remove-Partition -DiskNumber 1 -PartitionNumber 2 -Confirm:$false`|`diskpart > select partition 2 > delete partition`|Supprimer une partition|
|`Resize-Partition -DiskNumber 1 -PartitionNumber 2 -Size 100GB`|`diskpart > select partition 2 > extend size=51200`|Étendre une partition|
|`(Get-PartitionSupportedSize -DiskNumber 1 -PartitionNumber 2).SizeMax`|—|Taille maximale possible|
|`Set-Partition -DriveLetter E -NewDriveLetter F`|`diskpart > select volume E > assign letter=F`|Changer la lettre de lecteur|
|`Add-PartitionAccessPath -DiskNumber 1 -PartitionNumber 2 -AccessPath 'C:\montage'`|`diskpart > select volume 2 > assign mount=C:\montage`|Monter une partition dans un dossier|

### Volumes & formatage

|PowerShell|CMD|Description|
|---|---|---|
|`Get-Volume`|`diskpart > list volume`|Lister les volumes|
|`Get-Volume -DriveLetter E`|`fsutil volume diskfree E:`|Informations d'un volume|
|`Format-Volume -DriveLetter E -FileSystem NTFS -NewFileSystemLabel 'Données' -Confirm:$false`|`format E: /fs:ntfs /q /v:Données`|Formater en NTFS|
|`Format-Volume -DriveLetter E -FileSystem FAT32 -Confirm:$false`|`format E: /fs:fat32 /q`|Formater en FAT32|
|`Format-Volume -DriveLetter E -FileSystem ReFS -Confirm:$false`|—|Formater en ReFS (Windows Server)|
|`Set-Volume -DriveLetter E -NewFileSystemLabel 'Backup'`|`label E: Backup`|Renommer un volume|
|`Repair-Volume -DriveLetter E -Scan`|`chkdsk E: /scan`|Analyser sans démonter|
|`Repair-Volume -DriveLetter E -OfflineScanAndFix`|`chkdsk E: /f /r`|Corriger les erreurs|
|`Optimize-Volume -DriveLetter C -Defrag -Verbose`|`defrag C: /u /v`|Défragmenter|
|`Optimize-Volume -DriveLetter C -ReTrim`|`defrag C: /L`|TRIM (pour les SSD)|

### Images disque & iSCSI

|PowerShell|CMD|Description|
|---|---|---|
|`Mount-DiskImage -ImagePath 'C:\image.iso'`|—|Monter une image ISO ou VHDX|
|`Dismount-DiskImage -ImagePath 'C:\image.iso'`|—|Démonter une image|
|`New-VHD -Path 'C:\disk.vhdx' -SizeBytes 100GB -Dynamic`|—|Créer un disque virtuel VHDX dynamique|
|`New-VHD -Path 'C:\disk.vhd' -SizeBytes 50GB -Fixed`|—|Créer un disque virtuel VHD fixe|
|`Mount-VHD -Path 'C:\disk.vhdx'`|—|Monter un VHD/VHDX|
|`Dismount-VHD -Path 'C:\disk.vhdx'`|—|Démonter un VHD/VHDX|

### Espaces de stockage (Storage Spaces)

```powershell
# Lister les pools et espaces disponibles
Get-StoragePool
Get-PhysicalDisk | Where-Object CanPool -eq $true

# Créer un pool de stockage
$disques = Get-PhysicalDisk | Where-Object CanPool -eq $true
New-StoragePool -FriendlyName 'Pool-Données' -StorageSubSystemFriendlyName '*' -PhysicalDisks $disques

# Créer un volume miroir (RAID 1)
New-VirtualDisk -StoragePoolFriendlyName 'Pool-Données' `
    -FriendlyName 'Disque-Miroir' `
    -Size 500GB `
    -ResiliencySettingName Mirror

# Créer un volume parité (RAID 5-like)
New-VirtualDisk -StoragePoolFriendlyName 'Pool-Données' `
    -FriendlyName 'Disque-Parité' `
    -Size 1TB `
    -ResiliencySettingName Parity
```

---

## Pare-feu & sécurité Windows

### Règles de pare-feu

|PowerShell|CMD / netsh|Description|
|---|---|---|
|`Get-NetFirewallRule`|`netsh advfirewall firewall show rule name=all`|Lister toutes les règles|
|`Get-NetFirewallRule -Enabled True`|`netsh advfirewall firewall show rule name=all \| findstr Enabled`|Règles actives uniquement|
|`Get-NetFirewallRule -Name 'MonApp'`|`netsh advfirewall firewall show rule name="MonApp"`|Détails d'une règle|
|`New-NetFirewallRule -DisplayName 'MonApp-In' -Direction Inbound -Protocol TCP -LocalPort 8080 -Action Allow`|`netsh advfirewall firewall add rule name="MonApp-In" dir=in action=allow protocol=TCP localport=8080`|Autoriser un port entrant|
|`New-NetFirewallRule -DisplayName 'MonApp-Out' -Direction Outbound -Protocol TCP -RemotePort 443 -Action Allow`|`netsh advfirewall firewall add rule name="MonApp-Out" dir=out action=allow protocol=TCP remoteport=443`|Autoriser un port sortant|
|`New-NetFirewallRule -DisplayName 'MonApp-Prog' -Program 'C:\app\app.exe' -Action Allow -Direction Inbound`|`netsh advfirewall firewall add rule name="MonApp-Prog" dir=in action=allow program="C:\app\app.exe"`|Autoriser un programme|
|`New-NetFirewallRule -DisplayName 'Bloc-IP' -RemoteAddress 10.0.0.5 -Action Block -Direction Inbound`|`netsh advfirewall firewall add rule name="Bloc-IP" dir=in action=block remoteip=10.0.0.5`|Bloquer une IP|
|`Set-NetFirewallRule -DisplayName 'MonApp-In' -Enabled False`|`netsh advfirewall firewall set rule name="MonApp-In" new enable=no`|Désactiver une règle|
|`Enable-NetFirewallRule -DisplayName 'MonApp-In'`|`netsh advfirewall firewall set rule name="MonApp-In" new enable=yes`|Activer une règle|
|`Remove-NetFirewallRule -DisplayName 'MonApp-In'`|`netsh advfirewall firewall delete rule name="MonApp-In"`|Supprimer une règle|

### Profils pare-feu (Domaine / Privé / Public)

|PowerShell|CMD / netsh|Description|
|---|---|---|
|`Get-NetFirewallProfile`|`netsh advfirewall show allprofiles`|État de tous les profils|
|`Get-NetFirewallProfile -Name Domain`|`netsh advfirewall show domainprofile`|Profil domaine|
|`Set-NetFirewallProfile -Name Domain -Enabled True`|`netsh advfirewall set domainprofile state on`|Activer le pare-feu (profil domaine)|
|`Set-NetFirewallProfile -Name Public -DefaultInboundAction Block -DefaultOutboundAction Allow`|`netsh advfirewall set publicprofile firewallpolicy blockinbound,allowoutbound`|Politique par défaut (profil public)|
|`Set-NetFirewallProfile -All -Enabled True`|`netsh advfirewall set allprofiles state on`|Activer sur tous les profils|
|`Set-NetFirewallProfile -All -Enabled False`|`netsh advfirewall set allprofiles state off`|Désactiver le pare-feu (tous profils)|

> [!warning] Ne jamais désactiver le pare-feu en production sans solution de remplacement.

### Windows Defender (Antivirus)

|PowerShell|Description|
|---|---|
|`Get-MpComputerStatus`|État global de Windows Defender|
|`Get-MpThreat`|Menaces détectées|
|`Get-MpThreatDetection`|Historique des détections|
|`Start-MpScan -ScanType QuickScan`|Lancer une analyse rapide|
|`Start-MpScan -ScanType FullScan`|Lancer une analyse complète|
|`Start-MpScan -ScanType CustomScan -ScanPath 'C:\dossier'`|Analyser un dossier spécifique|
|`Update-MpSignature`|Mettre à jour les définitions|
|`Get-MpPreference`|Lire la configuration de Defender|
|`Set-MpPreference -DisableRealtimeMonitoring $false`|Activer la protection en temps réel|
|`Add-MpPreference -ExclusionPath 'C:\MonApp'`|Ajouter une exclusion|
|`Remove-MpPreference -ExclusionPath 'C:\MonApp'`|Supprimer une exclusion|

### BitLocker

|PowerShell|CMD / manage-bde|Description|
|---|---|---|
|`Get-BitLockerVolume`|`manage-bde -status`|État de tous les volumes BitLocker|
|`Get-BitLockerVolume -MountPoint C:`|`manage-bde -status C:`|État du volume C:|
|`Enable-BitLocker -MountPoint C: -RecoveryPasswordProtector`|`manage-bde -on C: -RecoveryPassword`|Activer BitLocker (mot de passe de récupération)|
|`Enable-BitLocker -MountPoint C: -TpmProtector`|`manage-bde -on C: -tpm`|Activer BitLocker avec TPM|
|`Suspend-BitLocker -MountPoint C: -RebootCount 1`|`manage-bde -protectors -disable C:`|Suspendre BitLocker (1 redémarrage)|
|`Resume-BitLocker -MountPoint C:`|`manage-bde -protectors -enable C:`|Réactiver BitLocker|
|`Disable-BitLocker -MountPoint C:`|`manage-bde -off C:`|Désactiver BitLocker (déchiffrement)|
|`(Get-BitLockerVolume -MountPoint C:).KeyProtector`|`manage-bde -protectors -get C:`|Afficher les protecteurs (clé de récupération)|
|`BackupToAAD-BitLockerKeyProtector -MountPoint C: -KeyProtectorId (Get-BitLockerVolume C:).KeyProtector[1].KeyProtectorId`|—|Sauvegarder la clé dans Azure AD|

### Audit & journaux de sécurité

|PowerShell|CMD / outil|Description|
|---|---|---|
|`Get-WinEvent -LogName Security -MaxEvents 100`|`wevtutil qe Security /c:100 /f:text`|Derniers événements de sécurité|
|`Get-WinEvent -LogName Security -FilterXPath "*[System[EventID=4625]]"`|—|Échecs d'authentification (ID 4625)|
|`Get-WinEvent -LogName Security -FilterXPath "*[System[EventID=4624]]"`|—|Connexions réussies (ID 4624)|
|`Get-WinEvent -LogName Security -FilterXPath "*[System[EventID=4720]]"`|—|Créations de comptes (ID 4720)|
|`auditpol /get /category:*`|`auditpol /get /category:*`|Politique d'audit actuelle|
|`auditpol /set /category:"Logon/Logoff" /success:enable /failure:enable`|`auditpol /set /category:"Logon/Logoff" /success:enable /failure:enable`|Activer l'audit de connexion|
|`secedit /export /cfg C:\securite.inf`|`secedit /export /cfg C:\securite.inf`|Exporter la politique de sécurité locale|
|`secedit /configure /db secedit.sdb /cfg C:\securite.inf`|`secedit /configure /db secedit.sdb /cfg C:\securite.inf`|Appliquer une politique de sécurité|

---

## 🔗 Voir aussi

- `Get-Help about_ActiveDirectory` — aide intégrée AD
- `Get-Command -Module ActiveDirectory` — toutes les commandes du module AD
- `Get-Command -Module GroupPolicy` — toutes les commandes GPO
- [docs.microsoft.com/windows-server](https://docs.microsoft.com/en-us/windows-server/)

