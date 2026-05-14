# PowerShell — Guide 

> [!info] Version de référence Ce guide couvre **PowerShell 7+** (cross-platform). Les sections marquées `# PS5` sont spécifiques à Windows PowerShell 5.1. Vérifier la version : `$PSVersionTable`

---

## Table des matières

- [[#Exécution & politique de scripts]]
- [[#Variables & types]]
- [[#Opérateurs]]
- [[#Chaînes de caractères]]
- [[#Tableaux & listes]]
- [[#Hashtables & dictionnaires]]
- [[#Tests & conditions]]
- [[#Structures de contrôle]]
- [[#Fonctions & paramètres]]
- [[#Pipeline & objets]]
- [[#Gestion des erreurs]]
- [[#Fichiers & système]]
- [[#Modules & imports]]
- [[#Jobs & parallélisme]]
- [[#Remoting]]
- [[#Bonnes pratiques]]

---

## Exécution & politique de scripts

```powershell
# Vérifier la politique d'exécution
Get-ExecutionPolicy
Get-ExecutionPolicy -List         # Par portée

# Modifier la politique
Set-ExecutionPolicy RemoteSigned  # Recommandé pour les scripts locaux
Set-ExecutionPolicy Bypass -Scope Process  # Pour la session courante uniquement

# Politiques disponibles
# Restricted    → aucun script (défaut Windows)
# AllSigned     → scripts signés uniquement
# RemoteSigned  → scripts locaux libres, distants signés
# Unrestricted  → tout autorisé (avec avertissement)
# Bypass        → tout autorisé sans avertissement
# Undefined     → hérite du niveau supérieur

# Débloquer un fichier téléchargé
Unblock-File -Path .\monscript.ps1

# Exécuter en contournant (ponctuellement)
powershell -ExecutionPolicy Bypass -File .\monscript.ps1

# Shebang PowerShell (pour scripts cross-platform)
#!/usr/bin/env pwsh
```

> [!warning] Sécurité Ne jamais utiliser `Set-ExecutionPolicy Unrestricted` en production. Préférer `RemoteSigned` ou signer ses scripts.

---

## Variables & types

```powershell
# Déclaration — pas de mot-clé obligatoire
$nom = "Alice"
$age = 30
$actif = $true

# Types explicites
[string]$texte    = "Bonjour"
[int]$entier      = 42
[double]$decimal  = 3.14
[bool]$drapeau    = $false
[datetime]$date   = Get-Date
[array]$tableau   = @(1, 2, 3)

# Constantes et lecture seule
Set-Variable -Name PI -Value 3.14159 -Option ReadOnly
Set-Variable -Name MAX -Value 100    -Option Constant  # Ne peut plus être supprimée

# Portée (scope)
$global:variable    = "partout"       # Globale
$script:variable    = "dans le script"
$local:variable     = "locale"        # Défaut
$private:variable   = "invisible aux enfants"

# Variables automatiques importantes
$null               # Valeur nulle
$true / $false      # Booléens
$_  / $PSItem       # Objet courant dans le pipeline
$?                  # Succès de la dernière commande
$LASTEXITCODE       # Code de sortie du dernier programme externe
$Error              # Tableau des erreurs récentes
$Error[0]           # Dernière erreur
$PSScriptRoot       # Répertoire du script courant
$PSCommandPath      # Chemin complet du script courant
$MyInvocation       # Informations sur la commande en cours
$env:USERNAME       # Variables d'environnement via $env:
$env:PATH
$env:USERPROFILE

# Vérifier si une variable existe
Test-Path variable:\nom
Get-Variable -Name nom -ErrorAction SilentlyContinue

# Supprimer une variable
Remove-Variable -Name nom
$nom = $null         # Vider sans supprimer

# Type checking & conversion
$val.GetType()
$val -is [string]
$val -as [int]       # Conversion silencieuse (null si échec)
[int]"42"            # Conversion forcée (exception si échec)
```

---

## Opérateurs

```powershell
# ── Arithmétique ──────────────────────────────────────────────────
5 + 3; 5 - 3; 5 * 3; 5 / 3; 5 % 3
[Math]::Pow(2, 10)      # 1024
[Math]::Sqrt(16)        # 4
[Math]::Round(3.567, 2) # 3.57

# ── Comparaison ───────────────────────────────────────────────────
-eq   # Égal
-ne   # Différent
-lt   # Inférieur strict
-le   # Inférieur ou égal
-gt   # Supérieur strict
-ge   # Supérieur ou égal

# Variantes insensibles à la casse
-ieq / -ceq   # i = insensible, c = sensible à la casse
"ABC" -eq "abc"     # $true (insensible par défaut)
"ABC" -ceq "abc"    # $false

# ── Logique ───────────────────────────────────────────────────────
-and  # ET
-or   # OU
-not  # NON
-xor  # OU exclusif
!     # Alias de -not

# ── Chaînes ───────────────────────────────────────────────────────
-like    "Alice" -like "Al*"       # Wildcard (* et ?)
-notlike
-match   "alice@test.com" -match '\w+@\w+'   # Regex
-notmatch
-replace "Bonjour" -replace "jour","soir"    # Regex replace
-split   "a,b,c" -split ","                  # Découper
-join    @("a","b","c") -join ","            # Joindre

# ── Collections ───────────────────────────────────────────────────
-in      "pomme" -in @("pomme","banane")   # $true
-notin
-contains @("pomme","banane") -contains "pomme"

# ── Bit à bit ─────────────────────────────────────────────────────
-band  -bor  -bxor  -bnot  -shl  -shr

# ── Affectation ───────────────────────────────────────────────────
$x += 5;  $x -= 2;  $x *= 3;  $x /= 2;  $x %= 3
$x++; $x--; ++$x; --$x

# ── Null-coalescing (PS7+) ────────────────────────────────────────
$val = $null ?? "défaut"            # "défaut"
$val ??= "défaut"                   # Assigne si null
$obj?.Propriété                     # Null-conditional (pas d'exception si null)
$tableau?[0]                        # Null-conditional sur index

# ── Ternaire (PS7+) ───────────────────────────────────────────────
$result = ($age -ge 18) ? "majeur" : "mineur"
```

---

## Chaînes de caractères

```powershell
# ── Types de guillemets ───────────────────────────────────────────
$nom = "Alice"
"Bonjour $nom"                  # Interpolation → "Bonjour Alice"
'Bonjour $nom'                  # Littéral → "Bonjour $nom"
"Résultat : $(2 + 2)"           # Expression interpolée → "Résultat : 4"
"Tab:`t  Newline:`n  Quote:`""  # Séquences d'échappement

# ── Méthodes .NET ─────────────────────────────────────────────────
$str = "  Bonjour le Monde  "

$str.Trim()                     # "Bonjour le Monde"
$str.TrimStart()                # "Bonjour le Monde  "
$str.TrimEnd()                  # "  Bonjour le Monde"
$str.ToUpper()                  # "  BONJOUR LE MONDE  "
$str.ToLower()                  # "  bonjour le monde  "
$str.Replace("Monde", "World")  # "  Bonjour le World  "
$str.Contains("Bonjour")        # $true
$str.StartsWith("  Bon")        # $true
$str.EndsWith("de  ")           # $true
$str.IndexOf("le")              # 9
$str.Substring(9, 2)            # "le"
$str.Split(" ")                 # Tableau de mots
$str.Length                     # 22

# ── Opérateurs ────────────────────────────────────────────────────
"a" * 5                         # "aaaaa"
"Bonjour" + " " + "Monde"       # Concaténation
-join @("a","b","c")            # "abc"
@("a","b","c") -join "-"        # "a-b-c"
"un,deux,trois" -split ","      # @("un","deux","trois")
"Bonjour" -replace "jour","soir"  # "Bonsoir"
"TEST" -replace '(.)', '$1 '    # "T E S T "  (regex avec groupe)

# ── Here-strings ─────────────────────────────────────────────────
# Avec interpolation
$texte = @"
Bonjour $nom,
Nous sommes le $(Get-Date -Format 'dd/MM/yyyy').
"@

# Sans interpolation
$texte = @'
Ceci est littéral : $nom $(Get-Date)
'@

# ── Formatage ─────────────────────────────────────────────────────
"Pi = {0:F4}" -f 3.14159        # "Pi = 3.1416"
"{0,-10} {1,10}" -f "gauche","droite"  # Alignement
[string]::Format("{0:N2}", 1234567)    # "1 234 567,00"

# Format numérique
"{0:D5}" -f 42          # "00042"   (entier sur 5 chiffres)
"{0:X}" -f 255          # "FF"      (hexadécimal)
"{0:B}" -f 10           # "1010"    (binaire, PS7+)
"{0:P1}" -f 0.856       # "85,6%"   (pourcentage)
```

---

## Tableaux & listes

```powershell
# ── Tableaux fixes ────────────────────────────────────────────────
$vide    = @()
$fruits  = @("pomme", "banane", "cerise")
$mixte   = @(1, "deux", $true, (Get-Date))
$matrice = @(@(1,2,3), @(4,5,6), @(7,8,9))

# Accès
$fruits[0]          # "pomme"
$fruits[-1]         # "cerise" (dernier)
$fruits[1..2]       # @("banane","cerise") (plage)
$fruits[-2..-1]     # Deux derniers

# Modification (crée un nouveau tableau)
$fruits += "datte"  # Ajouter (coûteux — copie le tableau)
$fruits[0] = "kiwi" # Modifier un élément existant

# Propriétés et méthodes
$fruits.Count
$fruits.Length
$fruits.Contains("banane")
$fruits.IndexOf("banane")
[array]::Reverse($fruits)
[array]::Sort($fruits)

# ── ArrayList (redimensionnable, performant) ──────────────────────
$liste = [System.Collections.ArrayList]@()
$liste = [System.Collections.ArrayList]@("a", "b", "c")

[void]$liste.Add("d")           # [void] supprime l'output du retour
$liste.AddRange(@("e", "f"))
$liste.Remove("b")              # Par valeur
$liste.RemoveAt(0)              # Par index
$liste.Insert(1, "X")           # Insérer à l'index 1
$liste.Contains("c")
$liste.Count

# ── List<T> générique (PS5+, type strict) ────────────────────────
$liste = [System.Collections.Generic.List[string]]::new()
$liste.Add("Alice")
$liste.Add("Bob")

# ── Opérations sur tableaux ───────────────────────────────────────
# Filtrer
$fruits | Where-Object { $_ -like "b*" }
$fruits.Where({ $_ -like "b*" })        # Plus rapide (méthode native)

# Trier
$fruits | Sort-Object
$fruits | Sort-Object -Descending
@(3,1,4,1,5) | Sort-Object -Unique

# Transformer
$fruits | ForEach-Object { $_.ToUpper() }
$fruits.ForEach({ $_.ToUpper() })        # Plus rapide

# Agréger
$nombres = @(1,2,3,4,5)
($nombres | Measure-Object -Sum).Sum           # 15
($nombres | Measure-Object -Average).Average   # 3
($nombres | Measure-Object -Maximum).Maximum   # 5
```

---

## Hashtables & dictionnaires

```powershell
# ── Hashtable ─────────────────────────────────────────────────────
$personne = @{
    Nom    = "Alice"
    Age    = 30
    Actif  = $true
}

# Accès
$personne["Nom"]
$personne.Nom               # Notation pointée (équivalente)

# Modification
$personne["Ville"] = "Paris"  # Ajouter
$personne.Age = 31             # Modifier
$personne.Remove("Actif")      # Supprimer

# Tester l'existence d'une clé
$personne.ContainsKey("Nom")
"Nom" -in $personne.Keys

# Itération
foreach ($cle in $personne.Keys) {
    "$cle : $($personne[$cle])"
}
$personne.GetEnumerator() | ForEach-Object { "$($_.Key) → $($_.Value)" }

# Propriétés
$personne.Keys
$personne.Values
$personne.Count

# ── Hashtable ordonnée ────────────────────────────────────────────
$ordonnee = [ordered]@{
    Premier  = 1
    Deuxieme = 2
    Troisieme = 3
}
# Les clés sont retournées dans l'ordre d'insertion

# ── Nested hashtables ─────────────────────────────────────────────
$config = @{
    Database = @{
        Host = "localhost"
        Port = 5432
        Name = "mabase"
    }
    Cache = @{
        TTL = 3600
    }
}
$config.Database.Host       # "localhost"
$config["Database"]["Port"] # 5432

# ── PSCustomObject (préférable aux hashtables pour les données) ───
$obj = [PSCustomObject]@{
    Nom  = "Alice"
    Age  = 30
    Ville = "Paris"
}
$obj.Nom                    # "Alice"
$obj | Add-Member -NotePropertyName "Email" -NotePropertyValue "alice@test.com"
$obj | Select-Object Nom, Age
```

---

## Tests & conditions

```powershell
# ── if / elseif / else ────────────────────────────────────────────
if ($age -ge 18) {
    "Majeur"
} elseif ($age -ge 13) {
    "Adolescent"
} else {
    "Enfant"
}

# ── switch ────────────────────────────────────────────────────────
switch ($jour) {
    "Lundi"    { "Début de semaine" }
    "Vendredi" { "Fin de semaine" }
    { $_ -in @("Samedi","Dimanche") } { "Week-end" }
    default    { "Milieu de semaine" }
}

# switch avec options
switch -Regex ($texte) {
    '^\d+$'   { "Nombre entier" }
    '^\d+\.\d+$' { "Décimal" }
    '^[a-z]+$' { "Minuscules uniquement" }
}

switch -Wildcard ($extension) {
    "*.jpg" { "Image JPEG" }
    "*.png" { "Image PNG" }
    "*.gif" { "Image GIF" }
}

# switch -File (lire un fichier ligne par ligne)
switch -File "log.txt" {
    { $_ -match "ERROR" }   { Write-Warning $_ }
    { $_ -match "WARNING" } { Write-Host $_ -ForegroundColor Yellow }
}
```

---

## Structures de contrôle

```powershell
# ── for ───────────────────────────────────────────────────────────
for ($i = 0; $i -lt 10; $i++) {
    Write-Host $i
}

# ── foreach ───────────────────────────────────────────────────────
foreach ($fruit in $fruits) {
    Write-Host $fruit
}

# Alias : % est l'alias de ForEach-Object dans le pipeline
1..10 | % { $_ * 2 }

# ── while ─────────────────────────────────────────────────────────
$compteur = 0
while ($compteur -lt 5) {
    $compteur++
}

# ── do-while / do-until ───────────────────────────────────────────
do {
    $saisie = Read-Host "Entrez un nombre"
} while ($saisie -notmatch '^\d+$')

do {
    $saisie = Read-Host "Entrez 'oui'"
} until ($saisie -eq "oui")

# ── Contrôle de boucle ────────────────────────────────────────────
break               # Quitter la boucle
continue            # Passer à l'itération suivante

# Labels pour les boucles imbriquées
:externe foreach ($i in 1..3) {
    foreach ($j in 1..3) {
        if ($j -eq 2) { continue externe }
        "$i-$j"
    }
}

# ── Plages ────────────────────────────────────────────────────────
1..10               # @(1,2,3,4,5,6,7,8,9,10)
10..1               # Décroissant
'a'..'z'            # Caractères
```

---

## Fonctions & paramètres

```powershell
# ── Fonction de base ──────────────────────────────────────────────
function Saluer {
    param([string]$Prenom)
    "Bonjour, $Prenom !"
}
Saluer -Prenom "Alice"
Saluer "Alice"          # Sans nom de paramètre

# ── Paramètres avancés (CmdletBinding) ───────────────────────────
function Copier-Fichiers {
    [CmdletBinding(SupportsShouldProcess, ConfirmImpact='Medium')]
    param(
        [Parameter(Mandatory, Position=0, ValueFromPipeline,
                   HelpMessage="Chemin source")]
        [ValidateNotNullOrEmpty()]
        [string]$Source,

        [Parameter(Mandatory, Position=1)]
        [string]$Destination,

        [Parameter()]
        [ValidateSet("Quiet","Normal","Verbose")]
        [string]$Mode = "Normal",

        [Parameter()]
        [ValidateRange(1, 100)]
        [int]$Threads = 4,

        [Parameter()]
        [ValidateScript({ Test-Path $_ -PathType Container })]
        [string]$WorkDir,

        [Parameter()]
        [ValidatePattern('^\d{4}-\d{2}-\d{2}$')]
        [string]$DateFilter,

        [switch]$Recurse,
        [switch]$Force
    )

    begin {
        # Exécuté une fois avant le pipeline
        Write-Verbose "Démarrage avec $Threads threads"
    }

    process {
        # Exécuté pour chaque objet du pipeline
        if ($PSCmdlet.ShouldProcess($Source, "Copier vers $Destination")) {
            Write-Verbose "Copie de $Source"
        }
    }

    end {
        # Exécuté une fois après le pipeline
        Write-Verbose "Terminé"
    }
}

# ── Validations disponibles ───────────────────────────────────────
[ValidateNotNull()]
[ValidateNotNullOrEmpty()]
[ValidateSet("A","B","C")]
[ValidateRange(1, 100)]
[ValidateLength(3, 20)]
[ValidatePattern('regex')]
[ValidateScript({ Test-Path $_ })]
[ValidateCount(1, 5)]               # Pour les tableaux

# ── Valeurs de retour ─────────────────────────────────────────────
function Calculer {
    param([int]$a, [int]$b)
    return $a + $b      # return explicite — optionnel, toute valeur non capturée est retournée
}

# Attention : TOUT output est retourné implicitement
function Piegeuse {
    "sortie 1"              # Retourné !
    $result = 42
    return $result          # Retourné aussi → fonction renvoie @("sortie 1", 42)
}
# Solution : rediriger vers $null ou [void]
[void](commande-avec-output)
commande-avec-output | Out-Null

# ── Fonctions filtre (pipeline) ───────────────────────────────────
filter Majuscule { $_.ToUpper() }
"bonjour", "monde" | Majuscule

# ── Splatting (passer des paramètres via hashtable) ───────────────
$params = @{
    Path        = "C:\temp"
    Filter      = "*.log"
    Recurse     = $true
    ErrorAction = "SilentlyContinue"
}
Get-ChildItem @params       # @ au lieu de $ pour le splatting
```

---

## Pipeline & objets

```powershell
# Le pipeline passe des objets, pas du texte (contrairement à Bash)

# ── Filtrer : Where-Object ────────────────────────────────────────
Get-Process | Where-Object { $_.CPU -gt 10 }
Get-Process | Where-Object CPU -gt 10          # Syntaxe simplifiée
Get-Process | ? { $_.Name -like "chrome*" }    # Alias ?

# ── Transformer : Select-Object ───────────────────────────────────
Get-Process | Select-Object Name, Id, CPU
Get-Process | Select-Object -First 5
Get-Process | Select-Object -Last 5
Get-Process | Select-Object -Skip 10
Get-Process | Select-Object -Unique
Get-Process | Select-Object Name, @{Name="Mo"; Expression={[Math]::Round($_.WS/1MB,1)}}

# ── Trier : Sort-Object ───────────────────────────────────────────
Get-Process | Sort-Object CPU -Descending
Get-Process | Sort-Object Name, CPU             # Tri multi-colonnes
Get-Process | Sort-Object -Property @{E="CPU"; Descending=$true}, Name

# ── Grouper : Group-Object ────────────────────────────────────────
Get-Process | Group-Object -Property ProcessName
Get-Service | Group-Object Status | Select-Object Name, Count

# ── Mesurer : Measure-Object ──────────────────────────────────────
Get-Process | Measure-Object CPU -Sum -Average -Maximum -Minimum
Get-Content fichier.txt | Measure-Object -Line -Word -Character

# ── Transformer chaque objet : ForEach-Object ────────────────────
1..10 | ForEach-Object { $_ * $_ }              # Carrés
Get-ChildItem | ForEach-Object { $_.FullName }

# ── Sélectionner des propriétés imbriquées ────────────────────────
Get-Process | Select-Object Name, @{
    Name = "MemMB"
    Expression = { [Math]::Round($_.WorkingSet64 / 1MB, 2) }
}

# ── Exporter / importer ───────────────────────────────────────────
Get-Process | Export-Csv -Path procs.csv -NoTypeInformation
Import-Csv procs.csv | Where-Object { [int]$_.Id -gt 1000 }

Get-Process | ConvertTo-Json | Out-File procs.json
Get-Content procs.json | ConvertFrom-Json

$data | Export-Clixml data.xml      # Format PowerShell natif (préserve les types)
$data = Import-Clixml data.xml

# ── Tee-Object ────────────────────────────────────────────────────
Get-Process | Tee-Object -FilePath procs.txt | Where-Object { $_.CPU -gt 5 }
Get-Process | Tee-Object -Variable processus   # Stocker dans une variable ET continuer
```

---

## Gestion des erreurs

```powershell
# ── ErrorAction ───────────────────────────────────────────────────
# Par commande
Get-Item "inexistant" -ErrorAction SilentlyContinue   # Ignorer
Get-Item "inexistant" -ErrorAction Stop               # Transformer en exception
Get-Item "inexistant" -ErrorAction Ignore             # Ignorer sans $Error
Get-Item "inexistant" -ErrorAction Inquire            # Demander à l'utilisateur

# Globale pour le script
$ErrorActionPreference = "Stop"    # Toute erreur devient une exception

# ── Try / Catch / Finally ─────────────────────────────────────────
try {
    $contenu = Get-Content "fichier.txt" -ErrorAction Stop
    [xml]$xml = $contenu                    # Peut lever une exception .NET
}
catch [System.IO.FileNotFoundException] {
    Write-Error "Fichier introuvable : $_"
}
catch [System.Xml.XmlException] {
    Write-Warning "XML invalide : $($_.Exception.Message)"
}
catch {
    # Attrape tout le reste
    Write-Error "Erreur inattendue : $($_.Exception.GetType().Name)"
    Write-Error $_.Exception.Message
    Write-Error $_.ScriptStackTrace
    throw   # Relancer l'exception
}
finally {
    # Toujours exécuté
    Write-Verbose "Nettoyage"
}

# ── Informations sur l'erreur ─────────────────────────────────────
$Error[0]                           # Dernière erreur
$Error[0].Exception.Message
$Error[0].Exception.GetType().Name
$Error[0].InvocationInfo.ScriptLineNumber
$Error[0].ScriptStackTrace
$Error.Clear()                      # Vider le tableau

# ── throw personnalisé ────────────────────────────────────────────
throw "Message d'erreur"
throw [System.ArgumentException]::new("Argument invalide : $param")
throw [System.IO.IOException] "Fichier inaccessible"

# ── Trap ──────────────────────────────────────────────────────────
trap {
    Write-Error "Erreur interceptée : $_"
    continue    # ou break pour sortir
}

# ── Journalisation structurée ─────────────────────────────────────
function Write-Log {
    param(
        [ValidateSet("INFO","WARN","ERROR","DEBUG")]
        [string]$Niveau = "INFO",
        [string]$Message
    )
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $ligne = "[$timestamp] [$Niveau] $Message"
    Add-Content -Path "$PSScriptRoot\script.log" -Value $ligne
    switch ($Niveau) {
        "ERROR" { Write-Error $Message }
        "WARN"  { Write-Warning $Message }
        "DEBUG" { Write-Debug $Message }
        default { Write-Verbose $Message }
    }
}
```

---

## Fichiers & système

```powershell
# ── Navigation ────────────────────────────────────────────────────
Set-Location "C:\Users"         # cd
Get-Location                    # pwd
Push-Location "C:\temp"         # Empiler le répertoire courant
Pop-Location                    # Revenir au précédent

# ── Fichiers et répertoires ───────────────────────────────────────
Get-ChildItem -Path "C:\temp" -Filter "*.log" -Recurse -Force
Get-ChildItem | Where-Object { !$_.PSIsContainer }  # Fichiers seulement
Get-ChildItem | Where-Object { $_.PSIsContainer }   # Répertoires seulement
Get-ChildItem -File                                  # PS3+
Get-ChildItem -Directory                             # PS3+

New-Item -Path "C:\temp\test.txt" -ItemType File -Force
New-Item -Path "C:\temp\nouveau" -ItemType Directory -Force
New-Item -ItemType Directory -Force -Path "C:\a\b\c"  # Crée tout le chemin

Copy-Item "source.txt" "dest.txt" -Force
Copy-Item "C:\source" "C:\dest" -Recurse -Force

Move-Item "source.txt" "dest.txt"
Rename-Item "ancien.txt" "nouveau.txt"
Remove-Item "fichier.txt" -Force
Remove-Item "C:\dossier" -Recurse -Force

Test-Path "C:\temp\fichier.txt"
Test-Path "C:\temp" -PathType Container

# ── Contenu de fichiers ───────────────────────────────────────────
Get-Content "fichier.txt"
Get-Content "fichier.txt" -Raw          # Une seule chaîne
Get-Content "fichier.txt" -Tail 20      # 20 dernières lignes
Get-Content "fichier.txt" -Wait         # Suivre en temps réel (tail -f)
Get-Content "fichier.txt" -Encoding UTF8

Set-Content "fichier.txt" "Contenu"     # Écraser
Add-Content "fichier.txt" "Ligne"       # Ajouter
Out-File "fichier.txt" -Encoding UTF8 -Append

# ── Chemins ───────────────────────────────────────────────────────
Split-Path "C:\dossier\fichier.txt"           # "C:\dossier"
Split-Path "C:\dossier\fichier.txt" -Leaf     # "fichier.txt"
Split-Path "C:\dossier\fichier.txt" -Qualifier # "C:"
Join-Path "C:\dossier" "sous\fichier.txt"     # "C:\dossier\sous\fichier.txt"
Resolve-Path ".\relatif"                      # Chemin absolu
[System.IO.Path]::GetExtension("fichier.txt") # ".txt"
[System.IO.Path]::GetFileNameWithoutExtension("fichier.txt") # "fichier"
[System.IO.Path]::GetTempPath()               # Répertoire temp

# ── Propriétés de fichier ─────────────────────────────────────────
$fi = Get-Item "fichier.txt"
$fi.Length                  # Taille en octets
$fi.LastWriteTime
$fi.CreationTime
$fi.Extension
$fi.BaseName
$fi.DirectoryName
$fi.Attributes

# ── Registre Windows ──────────────────────────────────────────────
Get-Item "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion"
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion" -Name ProductName
Set-ItemProperty "HKCU:\Software\MonApp" -Name "Setting" -Value "valeur"
New-Item "HKCU:\Software\MonApp" -Force

# ── Variables d'environnement ─────────────────────────────────────
$env:USERNAME
$env:COMPUTERNAME
$env:PATH
$env:TEMP
[System.Environment]::GetEnvironmentVariable("PATH", "Machine")   # Niveau machine
[System.Environment]::SetEnvironmentVariable("MA_VAR", "valeur", "User")
```

---

## Modules & imports

```powershell
# ── Gestion des modules ───────────────────────────────────────────
Get-Module                              # Modules chargés
Get-Module -ListAvailable               # Modules disponibles
Import-Module Az                        # Charger un module
Import-Module ./MonModule.psm1          # Depuis un fichier local
Remove-Module MonModule

# ── PowerShell Gallery ────────────────────────────────────────────
Find-Module -Name PSReadLine
Install-Module -Name Az -Scope CurrentUser
Install-Module -Name Pester -Force -SkipPublisherCheck
Update-Module -Name Az
Uninstall-Module -Name Az

# ── Structure d'un module (.psm1) ────────────────────────────────
# MonModule.psm1
function Public-Function {
    param([string]$Param)
    Private-Helper $Param
}

function Private-Helper {
    # Interne — non exportée
}

Export-ModuleMember -Function Public-Function
# ou dans le manifeste (.psd1)

# ── Manifeste de module (.psd1) ───────────────────────────────────
New-ModuleManifest -Path MonModule.psd1 `
    -RootModule MonModule.psm1 `
    -ModuleVersion "1.0.0" `
    -Author "Moi" `
    -FunctionsToExport @("Public-Function")

# ── Dot-sourcing (inclure un script dans la portée courante) ──────
. .\fonctions.ps1       # Les fonctions définies dans fonctions.ps1 sont disponibles ici
```

---

## Jobs & parallélisme

```powershell
# ── Background Jobs ───────────────────────────────────────────────
$job = Start-Job -ScriptBlock {
    Start-Sleep 5
    "Résultat du job"
}
Get-Job                         # Lister les jobs
$job.State                      # NotStarted, Running, Completed, Failed
Wait-Job $job                   # Attendre
Receive-Job $job                # Récupérer les résultats
Remove-Job $job

# Passer des variables
$val = 42
$job = Start-Job -ScriptBlock { param($x) $x * 2 } -ArgumentList $val

# ── ForEach-Object -Parallel (PS7+) ──────────────────────────────
$resultats = 1..10 | ForEach-Object -Parallel {
    Start-Sleep -Milliseconds (Get-Random -Maximum 500)
    "Traité : $_"
} -ThrottleLimit 4          # Max 4 en simultané

# Accéder aux variables du scope parent avec $using:
$multiplicateur = 3
1..5 | ForEach-Object -Parallel {
    $_ * $using:multiplicateur
} -ThrottleLimit 3

# ── Runspaces (plus performant pour grand volume) ─────────────────
$pool = [RunspaceFactory]::CreateRunspacePool(1, 8)
$pool.Open()

$jobs = foreach ($item in 1..20) {
    $ps = [PowerShell]::Create()
    $ps.RunspacePool = $pool
    [void]$ps.AddScript({ param($n) $n * $n }).AddArgument($item)
    @{ PS = $ps; Handle = $ps.BeginInvoke() }
}

$resultats = $jobs | ForEach-Object {
    $_.PS.EndInvoke($_.Handle)
    $_.PS.Dispose()
}
$pool.Close()

# ── Start-Process ─────────────────────────────────────────────────
Start-Process notepad.exe
Start-Process -FilePath "cmd.exe" -ArgumentList "/c dir C:\" -Wait -NoNewWindow
$proc = Start-Process python -ArgumentList "script.py" -PassThru
$proc.WaitForExit()
$proc.ExitCode
```

---

## Remoting

```powershell
# ── Configuration ────────────────────────────────────────────────
Enable-PSRemoting -Force            # Sur la machine cible
Test-WSMan -ComputerName serveur01

# ── Session interactive ───────────────────────────────────────────
Enter-PSSession -ComputerName serveur01
Enter-PSSession -ComputerName serveur01 -Credential (Get-Credential)
Exit-PSSession

# ── Sessions persistantes ─────────────────────────────────────────
$session = New-PSSession -ComputerName serveur01 -Credential $cred
Invoke-Command -Session $session -ScriptBlock { Get-Process }
Remove-PSSession $session

# ── Commandes sur plusieurs machines ─────────────────────────────
Invoke-Command -ComputerName srv01,srv02,srv03 -ScriptBlock {
    Get-Service -Name wuauserv
} -ThrottleLimit 10

# ── SSH Remoting (PS7+, cross-platform) ───────────────────────────
$session = New-PSSession -HostName user@linux-server -SSHTransport
Invoke-Command -HostName user@linux-server -ScriptBlock { uname -a }

# ── Copier des fichiers via session ───────────────────────────────
Copy-Item -Path "C:\local\fichier.txt" -Destination "C:\remote\" -ToSession $session
Copy-Item -Path "C:\remote\fichier.txt" -Destination "C:\local\" -FromSession $session
```

---

## Bonnes pratiques

```powershell
# ── Structure type d'un script ────────────────────────────────────
#Requires -Version 7.0
#Requires -Modules Az
#Requires -RunAsAdministrator

[CmdletBinding(SupportsShouldProcess)]
param(
    [Parameter(Mandatory)]
    [string]$InputPath,

    [Parameter()]
    [string]$OutputPath = "$PSScriptRoot\output",

    [switch]$Force
)

Set-StrictMode -Version Latest      # Erreur sur variables non définies, etc.
$ErrorActionPreference = "Stop"

# ── Naming conventions ────────────────────────────────────────────
# Fonctions  : Verbe-Nom en PascalCase (Get-Config, Set-Value, New-Report)
# Variables  : $PascalCase pour les paramètres, $camelCase pour les locales
# Constantes : $SCREAMING_SNAKE_CASE ou Set-Variable -Option Constant

# ── Approve-Verb ──────────────────────────────────────────────────
Get-Verb           # Liste des verbes approuvés
# Courants : Get Set New Remove Add Update Start Stop Invoke Test Copy Move

# ── Write-* vs return ─────────────────────────────────────────────
Write-Output    # Vers le pipeline (retournable)
Write-Host      # Directement à l'écran (ne passe pas dans le pipeline)
Write-Verbose   # Avec -Verbose ou $VerbosePreference
Write-Debug     # Avec -Debug ou $DebugPreference
Write-Warning   # Flux warning
Write-Error     # Flux erreur (non-terminant par défaut)
throw           # Terminant

# ── Fichier temporaire sécurisé ───────────────────────────────────
$tmp = [System.IO.Path]::GetTempFileName()
try {
    # utiliser $tmp
} finally {
    Remove-Item $tmp -ErrorAction SilentlyContinue
}

# ── Vérification des dépendances ─────────────────────────────────
foreach ($cmd in @("git", "docker", "kubectl")) {
    if (-not (Get-Command $cmd -ErrorAction SilentlyContinue)) {
        throw "Dépendance manquante : $cmd"
    }
}

# ── Profilage ─────────────────────────────────────────────────────
Measure-Command { Get-ChildItem -Recurse C:\Windows }

# Profiler une section
$sw = [System.Diagnostics.Stopwatch]::StartNew()
# ... code ...
$sw.Stop()
"Durée : $($sw.Elapsed.TotalMilliseconds) ms"
```

> [!tip] Outils recommandés
> 
> - **PSScriptAnalyzer** : linter officiel — `Install-Module PSScriptAnalyzer`
> - **Pester** : framework de tests — `Install-Module Pester`
> - **PSReadLine** : améliore le terminal — inclus dans PS7
> - **Oh My Posh** : prompt enrichi

---

## 🔗 Voir aussi

- [[CMD Batch Scripting]]
- `Get-Help about_*` — documentation intégrée complète
- `Get-Help about_Operators`
- `Get-Help about_Regular_Expressions`
- `Get-Help about_Scopes`
- [docs.microsoft.com/powershell](https://docs.microsoft.com/en-us/powershell/)