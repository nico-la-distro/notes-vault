## **vocabulary**

- **Exploit:** A piece of code that uses a vulnerability present on the target system.
- **Vulnerability:** A design, coding, or logic flaw affecting the target system. The exploitation of a vulnerability can result in disclosing confidential information or allowing the attacker to execute code on the target system.
- **Payload:** An exploit will take advantage of a vulnerability. However, if we want the exploit to have the result we want (gaining access to the target system, read confidential information, etc.), we need to use a payload. Payloads are the code that will run on the target system.
### Auxiliary

Any supporting module, such as scanners, crawlers and fuzzers, can be found here.

Terminal

```markup
root@ip-10-10-135-188:/opt/metasploit-framework/embedded/framework/modules# tree -L 1 auxiliary/
auxiliary/
├── admin
├── analyze
├── bnat
├── client
├── cloud
├── crawler
├── docx
├── dos
├── example.py
├── example.rb
├── fileformat
├── fuzzers
├── gather
├── parser
├── pdf
├── scanner
├── server
├── sniffer
├── spoof
├── sqli
├── voip
└── vsploit

20 directories, 2 files
```

### Encoders

Encoders will allow you to encode the exploit and payload in the hope that a signature-based antivirus solution may miss them.

Signature-based antivirus and security solutions have a database of known threats. They detect threats by comparing suspicious files to this database and raise an alert if there is a match. Thus encoders can have a limited success rate as antivirus solutions can perform additional checks.  

Terminal

```markup
root@ip-10-10-135-188:/opt/metasploit-framework/embedded/framework/modules# tree -L 1 encoders/
encoders/
├── cmd
├── generic
├── mipsbe
├── mipsle
├── php
├── ppc
├── ruby
├── sparc
├── x64
└── x86

10 directories, 0 files
```

### Evasion

While encoders will encode the payload, they should not be considered a direct attempt to evade antivirus software. On the other hand, “evasion” modules will try that, with more or less success.  

Terminal

```markup
root@ip-10-10-135-188:/opt/metasploit-framework/embedded/framework/modules# tree -L 2 evasion/
evasion/
└── windows
    ├── applocker_evasion_install_util.rb
    ├── applocker_evasion_msbuild.rb
    ├── applocker_evasion_presentationhost.rb
    ├── applocker_evasion_regasm_regsvcs.rb
    ├── applocker_evasion_workflow_compiler.rb
    ├── process_herpaderping.rb
    ├── syscall_inject.rb
    ├── windows_defender_exe.rb
    └── windows_defender_js_hta.rb

1 directory, 9 files
```

### Exploits

Exploits, neatly organized by target system.

Terminal

```markup
root@ip-10-10-135-188:/opt/metasploit-framework/embedded/framework/modules# tree -L 1 exploits/
exploits/
├── aix
├── android
├── apple_ios
├── bsd
├── bsdi
├── dialup
├── example_linux_priv_esc.rb
├── example.py
├── example.rb
├── example_webapp.rb
├── firefox
├── freebsd
├── hpux
├── irix
├── linux
├── mainframe
├── multi
├── netware
├── openbsd
├── osx
├── qnx
├── solaris
├── unix
└── windows

20 directories, 4 files
```

### NOPs

NOPs (No OPeration) do nothing, literally. They are represented in the Intel x86 CPU family with 0x90, following which the CPU will do nothing for one cycle. They are often used as a buffer to achieve consistent payload sizes.

Terminal

```markup
root@ip-10-10-135-188:/opt/metasploit-framework/embedded/framework/modules# tree -L 1 nops/
nops/
├── aarch64
├── armle
├── cmd
├── mipsbe
├── php
├── ppc
├── sparc
├── tty
├── x64
└── x86

10 directories, 0 files
```

### Payloads

Payloads are codes that will run on the target system.

Exploits will leverage a vulnerability on the target system, but to achieve the desired result, we will need a payload. Examples could be; getting a shell, loading a malware or backdoor to the target system, running a command, or launching calc.exe as a proof of concept to add to the penetration test report. Starting the calculator on the target system remotely by launching the calc.exe application is a benign way to show that we can run commands on the target system.

Running command on the target system is already an important step but having an interactive connection that allows you to type commands that will be executed on the target system is better. Such an interactive command line is called a "shell". Metasploit offers the ability to send different payloads that can open shells on the target system.

Terminal

```markup
root@ip-10-10-135-188:/opt/metasploit-framework/embedded/framework/modules# tree -L 1 payloads/
payloads/
├── adapters
├── singles
├── stagers
└── stages

4 directories, 0 files
```

You will see four different directories under payloads: adapters, singles, stagers and stages.

- **Adapters:** An adapter wraps single payloads to convert them into different formats. For example, a normal single payload can be wrapped inside a Powershell adapter, which will make a single powershell command that will execute the payload.  
- **Singles:** Self-contained payloads (add user, launch notepad.exe, etc.) that do not need to download an additional component to run.
- **Stagers:** Responsible for setting up a connection channel between Metasploit and the target system. Useful when working with staged payloads. “Staged payloads” will first upload a stager on the target system then download the rest of the payload (stage). This provides some advantages as the initial size of the payload will be relatively small compared to the full payload sent at once.
- **Stages:** Downloaded by the stager. This will allow you to use larger sized payloads.

Metasploit has a subtle way to help you identify single (also called “inline”) payloads and staged payloads.

- generic/shell_reverse_tcp
- windows/x64/shell/reverse_tcp

Both are reverse Windows shells. The former is an inline (or single) payload, as indicated by the “_” between “shell” and “reverse”. While the latter is a staged payload, as indicated by the “/” between “shell” and “reverse”.

  

### Post

Post modules will be useful on the final stage of the penetration testing process listed above, post-exploitation.

  

Terminal

```terminal
root@ip-10-10-135-188:/opt/metasploit-framework/embedded/framework/modules# tree -L 1 post/
post/
├── aix
├── android
├── apple_ios
├── bsd
├── firefox
├── hardware
├── linux
├── multi
├── networking
├── osx
├── solaris
└── windows

12 directories, 0 files
```

If you wish to familiarize yourself further with these modules, you can find them under the modules folder of your Metasploit installation. For the AttackBox these are under /opt/metasploit-framework/embedded/framework/modules

## **ranking**

![[metasploit ranking.png]]


## **some commands**

| Contexte      | Commande                                       |
| ------------- | ---------------------------------------------- |
| Terminal      | `msfconsole`                                   |
| msfconsole    | `use exploit/windows/smb/ms17_010_eternalblue` |
| msfconsole    | `use auxiliary/scanner/smb/smb_ms17_010`       |
| msfconsole    | `back`                                         |
| msfconsole    | `sessions`                                     |
| msfconsole    | `sessions -i <id>`                             |
| Module actif  | `show options`                                 |
| Module actif  | `set RHOSTS <ip>`                              |
| Module actif  | `set RPORT <port>`                             |
| Module actif  | `set PAYLOAD <payload>`                        |
| Module actif  | `set LHOST <ip_attacker>`                      |
| Module actif  | `set LPORT <port>`                             |
| Module actif  | `set SESSION <id>`                             |
| Module actif  | `setg RHOSTS <ip>`                             |
| Module actif  | `unset RHOSTS`                                 |
| Module actif  | `unset all`                                    |
| Module actif  | `unsetg RHOSTS`                                |
| Module actif  | `check`                                        |
| Module actif  | `run`                                          |
| Module actif  | `exploit`                                      |
| Module actif  | `exploit -z`                                   |
| Meterpreter   | `background`                                   |
| Meterpreter   | `exit`                                         |
| Shell distant | `whoami`                                       |
| Shell distant | `ipconfig`                                     |
| Shell distant | `dir`                                          |

## **meterpreter commands**

🔹 Core Commands

|Commande|Description|
|---|---|
|background|Met la session Meterpreter en arrière-plan|
|exit|Termine la session Meterpreter|
|guid|Affiche le GUID de la session|
|help|Affiche le menu d’aide|
|info|Affiche les informations sur un module Post|
|irb|Ouvre un shell Ruby interactif|
|load|Charge une ou plusieurs extensions Meterpreter|
|migrate|Migre Meterpreter vers un autre processus|
|run|Exécute un script Meterpreter ou un module Post|
|sessions|Permet de basculer rapidement entre les sessions|

📁 File System Commands

|Commande|Description|
|---|---|
|cd|Change le répertoire courant|
|ls / dir|Liste les fichiers du répertoire courant|
|pwd|Affiche le répertoire de travail actuel|
|edit|Permet d’éditer un fichier|
|cat|Affiche le contenu d’un fichier|
|rm|Supprime un fichier|
|search|Recherche des fichiers|
|upload|Téléverse un fichier ou un dossier|
|download|Télécharge un fichier ou un dossier|

🌐 Networking Commands

|Commande|Description|
|---|---|
|arp|Affiche le cache ARP de la machine cible|
|ifconfig|Liste les interfaces réseau|
|netstat|Affiche les connexions réseau actives|
|portfwd|Redirige un port local vers un service distant|
|route|Affiche ou modifie la table de routage|

🖥️ System Commands

|Commande|Description|
|---|---|
|clearev|Efface les journaux d’événements|
|execute|Exécute une commande|
|getpid|Affiche l’ID du processus courant|
|getuid|Affiche l’utilisateur exécutant Meterpreter|
|kill|Termine un processus par PID|
|pkill|Termine un processus par nom|
|ps|Liste les processus en cours|
|reboot|Redémarre la machine cible|
|shell|Ouvre un shell système|
|shutdown|Éteint la machine cible|
|sysinfo|Affiche les informations système (OS, architecture, etc.)|

## Types payoads

|Type|C’est quoi|Exemple|
|---|---|---|
|**Single (single-stage)**|Tout-en-un : le payload complet est livré d’un coup|reverse shell “complet” en une seule charge|
|**Stager**|Petit stub qui **établit la connexion** et **récupère** la suite|`reverse_tcp` stager qui télécharge le stage suivant|
|**Stage (staged payload)**|Le **vrai payload** téléchargé/exécuté après le stager|`meterpreter` (stage)|