# CMD / Batch Scripting — Guide

> [!info] Environnement Ce guide couvre les fichiers **.bat** / **.cmd** exécutés dans `cmd.exe` (Windows). Pour des scripts plus puissants, voir [[PowerShell Scripting]]. Tester la version : `ver`

> [!warning] Limites de CMD CMD est fragile : pas de types, pas de tableaux natifs, sensible aux espaces, portée des variables non intuitive. Utiliser PowerShell dès que la complexité augmente.

---

## Table des matières

- [[#Structure de base]]
- [[#Variables]]
- [[#Entrées / sorties]]
- [[#Opérateurs & tests]]
- [[#Structures de contrôle]]
- [[#Fonctions & labels]]
- [[#Boucles FOR avancées]]
- [[#Manipulation de chaînes]]
- [[#Fichiers & système]]
- [[#Redirections & pipes]]
- [[#Gestion des erreurs]]
- [[#Processus & services]]
- [[#Réseau]]
- [[#Astuces & pièges]]

---

## Structure de base

```bat
@echo off
:: Commentaire avec ::  (plus fiable que REM dans les blocs)
REM Commentaire avec REM (POSIX-like, fonctionne partout)

setlocal EnableDelayedExpansion
setlocal EnableExtensions

:: En-tête standard recommandé
@echo off
setlocal EnableDelayedExpansion EnableExtensions

title Mon Script v1.0

:: Récupérer le répertoire du script (équivalent de $PSScriptRoot)
set "SCRIPT_DIR=%~dp0"
set "SCRIPT_NAME=%~n0"
```

> [!tip] `@echo off` vs `echo off` Le `@` empêche l'affichage de la ligne `echo off` elle-même. Sans `@`, cette ligne s'affiche à l'exécution.

---

## Variables

```bat
:: Déclaration et affectation — pas d'espaces autour du =
set NOM=Alice
set AGE=30

:: TOUJOURS guillemeter les set pour éviter les espaces parasites
set "NOM=Alice"
set "CHEMIN=C:\Program Files\Mon App"

:: Lecture d'une variable
echo %NOM%
echo Le nom est : %NOM%

:: Variables d'environnement système
echo %USERNAME%
echo %COMPUTERNAME%
echo %OS%
echo %PROCESSOR_ARCHITECTURE%
echo %NUMBER_OF_PROCESSORS%
echo %SYSTEMROOT%          :: C:\Windows
echo %WINDIR%              :: C:\Windows
echo %TEMP%
echo %TMP%
echo %USERPROFILE%         :: C:\Users\Alice
echo %APPDATA%             :: C:\Users\Alice\AppData\Roaming
echo %LOCALAPPDATA%
echo %PROGRAMFILES%
echo %PROGRAMFILES(X86)%
echo %PUBLIC%
echo %HOMEDRIVE%           :: C:
echo %HOMEPATH%            :: \Users\Alice
echo %PATH%
echo %PATHEXT%             :: Extensions exécutables
echo %DATE%
echo %TIME%
echo %RANDOM%              :: Nombre aléatoire 0-32767
echo %ERRORLEVEL%          :: Code de retour de la dernière commande
echo %CD%                  :: Répertoire courant
echo %~dp0                 :: Répertoire du script en cours

:: Variables arithmétiques
set /a RESULTAT=10+5
set /a RESULTAT=10*5-3
set /a RESULTAT=%AGE%+1
set /a "X=5, Y=10, Z=X*Y"   :: Plusieurs calculs
echo %RESULTAT%

:: Opérateurs /a disponibles
:: + - * / %% (modulo) ^^ (puissance) & | ^ ~ << >>
set /a R=2 ** 8              :: 256
set /a R=17 %% 5             :: 2  (doubler le % dans les scripts)

:: Saisie utilisateur
set /p NOM=Entrez votre nom : 
set /p FICHIER=Chemin du fichier : 

:: Expansion différée (obligatoire pour les variables modifiées dans des boucles)
setlocal EnableDelayedExpansion
set COMPTEUR=0
for /l %%i in (1,1,5) do (
    set /a COMPTEUR+=1
    echo !COMPTEUR!          :: ! au lieu de % pour l'expansion différée
)

:: Vider une variable
set "NOM="
```

> [!warning] Expansion différée Dans un bloc `( )` (if, for…), `%var%` est évaluée **avant** l'exécution du bloc. Pour lire une valeur modifiée dans le bloc même, utiliser `!var!` avec `setlocal EnableDelayedExpansion`.

---

## Entrées / sorties

```bat
:: Afficher du texte
echo Bonjour le monde
echo.                        :: Ligne vide (echo suivi d'un point, sans espace)
echo   texte indenté

:: Afficher sans retour à la ligne (astuce avec set /p)
<nul set /p =Texte sans newline

:: Couleurs (0-F pour fond, texte)
color 0A                     :: Fond noir, texte vert
color                        :: Réinitialiser

:: Lire depuis un fichier
set /p LIGNE=<fichier.txt    :: Lire la première ligne

:: Afficher un fichier
type fichier.txt
more fichier.txt             :: Avec pagination

:: Effacer l'écran
cls
```

---

## Opérateurs & tests

```bat
:: ── Comparaison de chaînes ───────────────────────────────────────
if "%VAR%"=="valeur" echo égal
if not "%VAR%"=="valeur" echo différent
if /i "%VAR%"=="VALEUR" echo égal insensible casse   :: /i = ignore case

:: ── Comparaison numérique ────────────────────────────────────────
if %N% equ 5  echo égal
if %N% neq 5  echo différent
if %N% lss 5  echo inférieur strict
if %N% leq 5  echo inférieur ou égal
if %N% gtr 5  echo supérieur strict
if %N% geq 5  echo supérieur ou égal

:: ── Tests d'existence ────────────────────────────────────────────
if exist "fichier.txt" echo le fichier existe
if exist "C:\dossier\" echo le répertoire existe    :: Slash final obligatoire pour dossier
if not exist "fichier.txt" echo absent

:: ── Test de variable définie ─────────────────────────────────────
if defined NOM echo NOM est défini
if not defined NOM echo NOM n'est pas défini

:: ── Test de ERRORLEVEL ───────────────────────────────────────────
commande
if %errorlevel% equ 0 echo succès
if %errorlevel% neq 0 echo échec

:: Forme abrégée (true si ERRORLEVEL >= valeur)
if errorlevel 1 echo erreur ou plus
if not errorlevel 1 echo succès

:: ── Opérateurs logiques (pas de AND/OR natif — utiliser && et ||) ─
commande1 && commande2       :: commande2 si commande1 réussit (ERRORLEVEL 0)
commande1 || commande2       :: commande2 si commande1 échoue
commande1 & commande2        :: toujours les deux, séquentiellement

:: Simuler AND dans if
if "%A%"=="x" if "%B%"=="y" echo A=x ET B=y

:: Simuler OR (via goto)
if "%VAL%"=="a" goto match
if "%VAL%"=="b" goto match
goto no_match
:match
echo correspond
:no_match
```

---

## Structures de contrôle

```bat
:: ── if / else ────────────────────────────────────────────────────
if "%NOM%"=="Alice" (
    echo Bonjour Alice
    echo Bienvenue
) else (
    echo Bonjour inconnu
)

:: Imbriqué
if %AGE% geq 18 (
    if %AGE% lss 65 (
        echo adulte actif
    ) else (
        echo senior
    )
) else (
    echo mineur
)

:: ── goto (navigation) ────────────────────────────────────────────
goto debut

:section_ignoree
echo jamais affiché

:debut
echo programme démarré

:: ── Sortir du script ─────────────────────────────────────────────
exit /b 0        :: Quitter la fonction/script actuel avec code 0
exit /b 1        :: Quitter avec code d'erreur 1
exit 0           :: Quitter cmd.exe entièrement
```

---

## Fonctions & labels

```bat
@echo off
setlocal

call :Saluer "Alice"
call :Calculer 10 5
echo Résultat : %RESULT%
exit /b 0

:: ── Définition d'une fonction ────────────────────────────────────
:Saluer
    set "prenom=%~1"
    echo Bonjour, %prenom% !
    exit /b 0

:Calculer
    set /a RESULT=%~1 + %~2
    exit /b 0

:: ── Paramètres de fonctions / scripts ───────────────────────────
:: %0  = nom du script ou de la fonction
:: %1 … %9 = arguments positionnels
:: %*  = tous les arguments
:: %~1 = %1 sans guillemets
:: %~f1 = chemin complet
:: %~d1 = lettre de lecteur (C:)
:: %~p1 = chemin sans fichier
:: %~n1 = nom de fichier sans extension
:: %~x1 = extension (.txt)
:: %~s1 = chemin court (8.3)
:: %~a1 = attributs du fichier
:: %~t1 = date/heure du fichier
:: %~z1 = taille du fichier

:: Exemple d'utilisation
:InfoFichier
    echo Chemin complet : %~f1
    echo Répertoire     : %~dp1
    echo Nom            : %~n1
    echo Extension      : %~x1
    echo Taille         : %~z1 octets
    exit /b 0

:: ── shift (décaler les arguments) ────────────────────────────────
:ParseArgs
    if "%~1"=="" exit /b 0
    echo Argument : %~1
    shift
    goto ParseArgs
```

---

## Boucles FOR avancées

```bat
:: ── for classique (compteur) ─────────────────────────────────────
for /l %%i in (1,1,10) do echo %%i
::              début,incrément,fin
for /l %%i in (10,-2,0) do echo %%i    :: 10 8 6 4 2 0
for /l %%i in (0,1,9) do set /a TOTAL+=%%i

:: ── for sur une liste ─────────────────────────────────────────────
for %%f in (pomme banane cerise) do echo %%f

:: ── for sur des fichiers ─────────────────────────────────────────
for %%f in (*.txt) do echo %%f
for %%f in ("C:\temp\*.log") do (
    echo Fichier : %%~nxf
    echo Taille  : %%~zf
    echo Date    : %%~tf
)

:: Récursif /r
for /r "C:\temp" %%f in (*.txt) do echo %%~ff
for /r %%f in (*.bak) do del "%%f"     :: Supprimer tous les .bak récursivement

:: Répertoires /d
for /d %%d in ("C:\*") do echo Dossier : %%d
for /d /r "C:\" %%d in (*) do echo %%d

:: ── for sur la sortie d'une commande /f ──────────────────────────
:: Lire la sortie d'une commande
for /f "tokens=*" %%i in ('dir /b /a-d "C:\temp"') do echo %%i

:: Découper en tokens
for /f "tokens=1,2,3 delims=," %%a in ("un,deux,trois") do (
    echo Premier  : %%a
    echo Deuxième : %%b
    echo Troisième: %%c
)

:: Lire un fichier ligne par ligne
for /f "usebackq tokens=*" %%i in ("fichier.txt") do echo %%i

:: Ignorer les lignes commençant par ; (ou tout autre caractère)
for /f "usebackq eol=# tokens=*" %%i in ("config.txt") do echo %%i

:: Lire en sautant les N premières lignes
for /f "skip=2 tokens=*" %%i in ('commande') do echo %%i

:: Récupérer une valeur spécifique
for /f "tokens=2 delims==" %%v in ('wmic os get Caption /value') do set OS=%%v
for /f "tokens=4 delims= " %%v in ('ver') do set VERSION=%%v
```

---

## Manipulation de chaînes

```bat
:: ── Sous-chaîne  %var:~offset,longueur% ─────────────────────────
set STR=Bonjour le Monde
echo %STR:~0,7%       :: "Bonjour"
echo %STR:~8,2%       :: "le"
echo %STR:~-5%        :: "Monde" (5 derniers caractères)
echo %STR:~0,-6%      :: "Bonjour le" (tout sauf les 6 derniers)

:: ── Remplacement  %var:ancien=nouveau% ───────────────────────────
set STR=Bonjour le Monde
echo %STR:Monde=World%         :: "Bonjour le World"
echo %STR: =_%                 :: "Bonjour_le_Monde" (remplacer espaces)

:: Supprimer une sous-chaîne
echo %STR:Bonjour =%           :: "le Monde"

:: ── Longueur d'une chaîne (pas de fonction native — astuce) ──────
set STR=Bonjour
call :strlen "%STR%" LEN
echo Longueur : %LEN%

:strlen
    set "str_=%~1"
    set LEN=0
    :loop
    if defined str_ (
        set "str_=%str_:~1%"
        set /a LEN+=1
        goto loop
    )
    set "%2=%LEN%"
    exit /b 0

:: ── Tester si une chaîne contient une sous-chaîne ────────────────
echo %STR% | find /i "jour" >nul && echo contient || echo absent
if not "%STR:jour=%"=="%STR%" echo contient

:: ── Mettre en majuscules (via PowerShell) ────────────────────────
for /f %%i in ('powershell -c "'%STR%'.ToUpper()"') do set UPPER=%%i
```

---

## Fichiers & système

```bat
:: ── Navigation ────────────────────────────────────────────────────
cd "C:\Users"
cd /d "D:\Projets"       :: /d pour changer de lecteur aussi
pushd "C:\temp"          :: Sauvegarder et changer
popd                     :: Revenir au répertoire sauvegardé

:: ── Gestion de fichiers ──────────────────────────────────────────
copy source.txt dest.txt
copy source.txt "C:\dest\"
xcopy "C:\source" "C:\dest" /e /i /y /h
::    /e = sous-dossiers vides inclus
::    /i = destination est un dossier
::    /y = pas de confirmation d'écrasement
::    /h = fichiers cachés inclus
::    /s = sous-dossiers non vides
::    /d = seulement fichiers plus récents

robocopy source dest *.* /e /z /b /r:3 /w:5 /log:copie.log
::    /e  = récursif avec dossiers vides
::    /z  = mode redémarrable
::    /b  = mode backup (contourne ACL)
::    /r:3 = 3 tentatives en cas d'échec
::    /w:5 = attendre 5s entre tentatives
::    /mir = miroir (supprime ce qui n'est plus dans source)
::    /xf *.tmp = exclure fichiers .tmp
::    /xd temp logs = exclure dossiers

move fichier.txt "C:\dest\"
rename ancien.txt nouveau.txt
del fichier.txt
del /f /q "C:\temp\*.log"    :: /f = forcer, /q = silencieux
del /s "*.tmp"               :: Récursif

:: ── Gestion de dossiers ──────────────────────────────────────────
mkdir "C:\nouveau\dossier"   :: Crée tout le chemin
md "C:\a\b\c"                :: Alias de mkdir
rmdir "C:\vide"
rmdir /s /q "C:\dossier"     :: Récursif + silencieux

:: ── Attributs ────────────────────────────────────────────────────
attrib +r fichier.txt        :: Lecture seule
attrib -r fichier.txt        :: Enlever lecture seule
attrib +h +s dossier         :: Caché + système
attrib -h -s -r /s /d *      :: Enlever tout récursivement

:: ── Informations ──────────────────────────────────────────────────
dir                          :: Lister le répertoire
dir /b                       :: Noms seuls
dir /b /a-d                  :: Fichiers seulement
dir /b /ad                   :: Dossiers seulement
dir /b /s "*.txt"            :: Récursif
dir /o:n                     :: Trié par nom
dir /o:s                     :: Trié par taille
dir /o:d                     :: Trié par date

:: ── Liens symboliques ────────────────────────────────────────────
mklink lien.txt cible.txt           :: Lien symbolique (fichier)
mklink /d lien_dossier cible_dossier :: Lien symbolique (dossier)
mklink /h lien_hard cible           :: Lien physique (hard link)
mklink /j jonction "C:\cible"       :: Junction (dossiers locaux seulement)

:: ── Date et heure ────────────────────────────────────────────────
echo %DATE%          :: Format local (jj/mm/aaaa en FR)
echo %TIME%          :: hh:mm:ss,cc

:: Extraire les composants (dépend des paramètres régionaux — fragile)
for /f "tokens=1-3 delims=/ " %%a in ("%DATE%") do (
    set JOUR=%%a
    set MOIS=%%b
    set ANNEE=%%c
)

:: Format ISO via PowerShell (fiable)
for /f %%d in ('powershell -c "Get-Date -Format yyyy-MM-dd"') do set DATISO=%%d
```

---

## Redirections & pipes

```bat
:: ── Redirections standard ────────────────────────────────────────
commande > fichier.txt          :: stdout → fichier (écrase)
commande >> fichier.txt         :: stdout → fichier (ajoute)
commande 2> erreurs.txt         :: stderr → fichier
commande 2>> erreurs.txt        :: stderr → fichier (ajoute)
commande > tout.txt 2>&1        :: stdout + stderr → fichier
commande 2>&1 | pipe            :: stderr dans le pipe
commande > nul                  :: Jeter stdout
commande 2> nul                 :: Jeter stderr
commande > nul 2>&1             :: Jeter tout

:: ── Pipes ────────────────────────────────────────────────────────
dir | sort
dir /b | findstr /i "\.txt$"
type fichier.txt | find "motif"
type fichier.txt | find /c "motif"   :: /c = compter les lignes

:: ── find vs findstr ──────────────────────────────────────────────
find "exact" fichier.txt             :: Recherche exacte (case-sensitive par défaut)
find /i "insensible" fichier.txt     :: /i = insensible casse
find /v "absent" fichier.txt         :: /v = lignes NE contenant PAS

findstr "regex" fichier.txt          :: Support regex basique
findstr /i /r "^[0-9]" fichier.txt   :: /r = regex, /i = insensible
findstr /s "motif" "C:\*.txt"        :: /s = récursif
findstr /n "motif" fichier.txt       :: /n = numéros de lignes
findstr /m "motif" *.txt             :: /m = noms de fichiers seulement

:: ── sort ─────────────────────────────────────────────────────────
sort fichier.txt
sort /r fichier.txt           :: Inverse
sort /+3 fichier.txt          :: Trier à partir de la colonne 3
dir /b | sort /r

:: ── clip (presse-papier) ─────────────────────────────────────────
echo Bonjour | clip
dir /b | clip
type fichier.txt | clip
```

---

## Gestion des erreurs

```bat
@echo off
setlocal

:: ── ERRORLEVEL ────────────────────────────────────────────────────
commande
if %errorlevel% neq 0 (
    echo Erreur : code %errorlevel%
    exit /b %errorlevel%
)

:: Raccourci avec && et ||
commande || goto :erreur
commande && echo succès

:: ── Vérification systématique ─────────────────────────────────────
:run
    call :exec "xcopy source dest /e /y" "Copie échouée"
    call :exec "commande2" "Commande2 échouée"
    exit /b 0

:exec
    %~1
    if %errorlevel% neq 0 (
        echo ERREUR: %~2 (code %errorlevel%)
        exit /b %errorlevel%
    )
    exit /b 0

:: ── Journalisation ────────────────────────────────────────────────
set "LOG=C:\logs\script.log"

:log
    echo [%DATE% %TIME%] %~1 >> "%LOG%"
    exit /b 0

call :log "INFO: démarrage du script"
call :log "ERREUR: fichier introuvable"

:: ── Nettoyage à la sortie ─────────────────────────────────────────
:: Pas de trap natif — utiliser un bloc de fin avec goto
@echo off
set "TMPFILE=%TEMP%\temp_%RANDOM%.tmp"

:: ... code principal ...
goto :fin

:erreur
    echo Erreur critique, nettoyage...

:fin
    if exist "%TMPFILE%" del /f /q "%TMPFILE%"
    endlocal
    exit /b %errorlevel%
```

---

## Processus & services

```bat
:: ── Processus ────────────────────────────────────────────────────
tasklist                            :: Lister les processus
tasklist /fi "imagename eq chrome*" :: Filtrer
tasklist /fo csv                    :: Format CSV
tasklist /v                         :: Verbeux

taskkill /im notepad.exe            :: Par nom
taskkill /pid 1234                  :: Par PID
taskkill /im chrome.exe /f          :: /f = forcer
taskkill /im chrome.exe /f /t       :: /t = tuer l'arbre de processus

start "" notepad.exe                :: Lancer un programme
start /wait notepad.exe             :: Attendre la fin
start /b commande                   :: En arrière-plan sans nouvelle fenêtre
start /high programme.exe           :: Priorité haute
start /min programme.exe            :: Minimisé

:: Lancer et récupérer le PID
start "" /b "programme.exe"
for /f "tokens=2" %%p in ('tasklist /fi "imagename eq programme.exe" /fo list ^| find "PID"') do set PID=%%p

:: ── Services ──────────────────────────────────────────────────────
net start                           :: Lister les services en cours
net start "Spooler"                 :: Démarrer
net stop "Spooler"                  :: Arrêter
net pause "Spooler"                 :: Suspendre
net continue "Spooler"              :: Reprendre

sc query                            :: État de tous les services
sc query wuauserv                   :: État d'un service
sc start wuauserv
sc stop wuauserv
sc config wuauserv start= auto      :: Démarrage automatique
sc config wuauserv start= disabled  :: Désactiver

:: ── Tâches planifiées ────────────────────────────────────────────
schtasks /query /fo list
schtasks /create /tn "MaTache" /tr "C:\script.bat" /sc daily /st 08:00
schtasks /run /tn "MaTache"
schtasks /delete /tn "MaTache" /f

:: ── Timeout / pause ───────────────────────────────────────────────
timeout /t 5                :: Attendre 5 secondes (avec compte à rebours)
timeout /t 5 /nobreak       :: Sans interruption par touche
ping -n 6 127.0.0.1 >nul    :: Attendre ~5 secondes (hack classique)
```

---

## Réseau

```bat
:: ── Connectivité ─────────────────────────────────────────────────
ping google.com
ping -n 1 192.168.1.1 >nul && echo accessible || echo inaccessible
tracert google.com
pathping google.com         :: ping + tracert combinés
nslookup google.com

:: ── Configuration réseau ─────────────────────────────────────────
ipconfig                    :: Interfaces réseau
ipconfig /all               :: Détail complet
ipconfig /release           :: Libérer le bail DHCP
ipconfig /renew             :: Renouveler le bail DHCP
ipconfig /flushdns          :: Vider le cache DNS

netstat -an                 :: Connexions et ports
netstat -b                  :: Avec noms d'exécutables
netstat -r                  :: Table de routage
arp -a                      :: Table ARP

:: ── Partages réseau ─────────────────────────────────────────────
net use                                 :: Lecteurs mappés
net use Z: \\serveur\partage
net use Z: \\serveur\partage /persistent:yes
net use Z: /delete
net share                               :: Partages locaux
net view \\serveur                      :: Partages d'un serveur

:: ── Téléchargement (via PowerShell ou certutil) ──────────────────
powershell -c "Invoke-WebRequest -Uri 'https://url' -OutFile 'fichier'"
certutil -urlcache -split -f "https://url" fichier.exe
bitsadmin /transfer MaJob "https://url" "C:\fichier"

:: ── Firewall ─────────────────────────────────────────────────────
netsh advfirewall show allprofiles
netsh advfirewall firewall add rule name="Mon App" dir=in action=allow program="C:\app.exe"
netsh advfirewall firewall delete rule name="Mon App"
```

---

## Astuces & pièges

```bat
:: ── Pièges classiques ─────────────────────────────────────────────

:: PIÈGE 1 : espaces dans les valeurs de variables
set NOM=Alice Martin       :: OK
set NOM = Alice            :: MAUVAIS — crée la variable "NOM " (avec espace)
if "%NOM%"=="Alice" ...    :: Toujours guillemeter dans les comparaisons

:: PIÈGE 2 : chemins avec espaces
cd C:\Program Files        :: ERREUR
cd "C:\Program Files"      :: OK
dir "%USERPROFILE%\Desktop" :: Toujours guillemeter les chemins

:: PIÈGE 3 : % dans les scripts vs interactif
:: En interactif : %VAR%
:: Dans un .bat : %%VAR%% dans les boucles for (%%i, %%f, etc.)
:: ^ est le caractère d'échappement dans cmd

:: PIÈGE 4 : endlocal détruit les variables
setlocal
set RESULT=42
endlocal
echo %RESULT%          :: Vide ! Les variables locales sont perdues

:: Passer une valeur à travers endlocal
setlocal
set RESULT=42
endlocal & set RESULT=%RESULT%   :: Astuce : & évalue avant endlocal

:: ── Astuces utiles ───────────────────────────────────────────────

:: Vérifier si on est admin
net session >nul 2>&1
if %errorlevel% neq 0 (
    echo Ce script nécessite les droits administrateur.
    exit /b 1
)

:: Se relancer en admin
if not "%1"=="admin" (
    powershell -c "Start-Process '%~f0' -ArgumentList 'admin' -Verb RunAs"
    exit /b
)

:: Vérifier si un port est ouvert
powershell -c "Test-NetConnection -ComputerName 'host' -Port 80" | find "True" >nul

:: Obtenir l'IP locale
for /f "tokens=2 delims=:" %%i in ('ipconfig ^| find "IPv4"') do set IP=%%i

:: Générer un UUID
powershell -c "[guid]::NewGuid().ToString()"

:: Trouver un fichier
where notepad.exe           :: Dans le PATH
where /r "C:\" *.exe        :: Récursif

:: Compresser / décompresser (via PowerShell ou expand)
powershell -c "Compress-Archive 'C:\source' 'archive.zip'"
powershell -c "Expand-Archive 'archive.zip' 'C:\dest'"

:: Variables en lecture seule (via reg ou workaround)
:: CMD n'a pas de ReadOnly natif — utiliser PowerShell pour ça

:: ── Encodage ─────────────────────────────────────────────────────
chcp                        :: Afficher la page de code courante
chcp 65001                  :: Passer en UTF-8
chcp 1252                   :: Windows-1252 (Western European)
chcp 850                    :: DOS Latin 1 (défaut ancien)
```

---

## 🔗 Voir aussi

- [[PowerShell Scripting]]
- `help` ou `commande /?` — aide intégrée pour chaque commande
- `cmd /?` — options de cmd.exe
- [ss64.com/nt](https://ss64.com/nt/) — référence complète en ligne