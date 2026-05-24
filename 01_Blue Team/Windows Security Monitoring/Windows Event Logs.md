## Event Viewer

- Logs Windows = format binaire propriétaire `.evt` / `.evtx`
- Conversion possible en XML via Windows API
- Emplacement :

C:\Windows\System32\winevt\Logs
### Elements of a Windows Event Log

#### Types de logs

|Log|Contenu|
|---|---|
|System Logs|OS, hardware, drivers, changements système|
|Security Logs|logon/logoff, audit, activités suspectes|
|Application Logs|erreurs, événements, warnings apps|
|Directory Service Events|activités Active Directory|
|File Replication Service Events|réplication GPO / scripts|
|DNS Event Logs|événements DNS|
|Custom Logs|logs applicatifs personnalisés|

#### Accès aux logs

|Méthode|Type|
|---|---|
|Event Viewer|GUI|
|wevtutil.exe|CLI|
|Get-WinEvent|PowerShell|

---

## wevtutil.exe

- Outil CLI pour gérer les Windows Event Logs
- Fonctions :
    - récupérer infos logs / publishers
    - query events
    - export / archive
    - clear logs
    - install / uninstall manifests

### Aide

wevtutil.exe /?
wevtutil COMMAND /?

### Syntaxe

wevtutil COMMAND [ARGUMENT ...] [/OPTION:VALUE ...]

- commandes et options **non sensibles à la casse**
- version courte ou longue possible
### Commandes principales

|Commande|Description|
|---|---|
|el|list logs|
|gl|get log config|
|sl|set log config|
|ep|list publishers|
|gp|get publisher info|
|im|install manifest|
|um|uninstall manifest|
|qe|query events|
|gli|log status|
|epl|export log|
|al|archive log|
|cl|clear log|

---

## Get-WinEvent

- Cmdlet PowerShell pour récupérer les event logs (local/remote)
- Remplace `Get-EventLog`
- Support :
    - multi sources
    - XPath / XML / HashTable queries

### Aide

```powershell
Get-Help Get-WinEvent
```
### Exemple 1: Get all logs

```powershell
Get-WinEvent -ListLog *
```

- Liste tous les logs
- `RecordCount` peut être 0/null

### Exemple 2: Providers + logs

```powershell
Get-WinEvent -ListProvider *
```

- `Name` = provider
- `LogLinks` = logs associés

### Exemple 3: Filtering (Where-Object)

```powershell
Get-WinEvent -LogName Application | Where-Object { $_.ProviderName -Match 'WLMS' }
```

⚠️ inefficace sur gros logs

### Filtering optimal (FilterHashtable)

```powershell
Get-WinEvent -FilterHashtable @{  
  LogName='Application'  
  ProviderName='WLMS'  
}
```

### Syntaxe HashTable

```powershell
@{ [name] = [value]; [[name] = [value]] ... }
```

Règles :

- `@{ }`
- `clé = valeur`
- `;` optionnel si nouvelle ligne

### FilterHashtable (usage)

- filtrage natif → plus performant
- construire **clé par clé**
- infos récupérables via Event Viewer

https://docs.microsoft.com/en-us/powershell/scripting/samples/Creating-Get-WinEvent-queries-with-FilterHashtable?view=powershell-7.1

### Exemple avancé

```powershell
Get-WinEvent -FilterHashtable @{  
  LogName='Microsoft-Windows-PowerShell/Operational'  
  ID=4104  
} | Select-Object -Property Message | Select-String -Pattern 'SecureString'
```

https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_hash_tables?view=powershell-7.1
### Notes

- `ID` = Event ID
- `ProviderName` = source
- adapté aux gros volumes de logs
- Event Viewer utile pour préparer les filtres

---

## XPath Queries

- Langage pour requêter XML
- Windows Event Log → support **subset XPath 1.0**
- utilisé avec :
    - `Get-WinEvent`
    - `wevtutil`

### Exemple

// The following query selects all events from the channel or log file where the severity level is less than or equal to 3 and the event occurred in the last 24 hour period.

[System[(Level <= 3) and TimeCreated[timediff(@SystemTime) <= 86400000]]]

- Level ≤ 3
- événements dernières 24h

### Structure

- commence par `*` ou `Event`
- basé sur structure XML (Event Viewer → Details → XML View)

### Construction requête

1. Root

```powershell
Get-WinEvent -LogName Application -FilterXPath '*'
```

2. System

```powershell
Get-WinEvent -LogName Application -FilterXPath '*/System/'
```

3. EventID

```powershell
Get-WinEvent -LogName Application -FilterXPath '*/System/EventID=100'
```

### Exemple résultat

```powershell
Get-WinEvent -LogName Application -FilterXPath '*/System/EventID=100'
```

### wevtutil équivalent

```powershell
wevtutil qe Application /q:*/System[EventID=100] /f:text /c:1
```

|Option|Description|
|---|---|
|/q|XPath query|
|/f:text|output texte|
|/c:1|nombre d’événements|

### Filtre Provider

```powershell
Get-WinEvent -LogName Application -FilterXPath '*/System/Provider[@Name="WLMS"]'
```

### Combinaison conditions

```powershell
Get-WinEvent -LogName Application -FilterXPath '*/System/EventID=101 and */System/Provider[@Name="WLMS"]'
```

### EventData

```powershell
Get-WinEvent -LogName Security -FilterXPath '*/EventData/Data[@Name="TargetUserName"]="System"' -MaxEvents 1
```

### Notes

- `System` recommandé (mais `*` possible)
- attributs → `[@Name="..."]`
- `EventData` pas toujours présent
- `-MaxEvents` limite résultats
- Event Viewer (XML View) = aide construction requêtes

---

