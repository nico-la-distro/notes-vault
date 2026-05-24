### LSASS — Local Security Authority Subsystem Service

Processus Windows (`lsass.exe`) responsable de l'authentification sur le système. Il gère :

- La vérification des credentials (login utilisateur)
- La création des tokens d'accès
- Le stockage en mémoire des credentials actifs (hashes NTLM, tickets Kerberos)

C'est précisément parce qu'il stocke les credentials en mémoire qu'il est ciblé par Mimikatz — un attaquant qui dump LSASS récupère les hashes des utilisateurs connectés, potentiellement celui d'un admin de domaine.

**Ce qui est normal :** `svchost.exe` accède régulièrement à LSASS — c'est légitime.

**Ce qui est suspect :** n'importe quel autre process qui y accède, surtout depuis un chemin inhabituel (`\Temp\`, `\Downloads\`…).

---

