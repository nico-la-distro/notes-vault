## LSASS - Local Security Authority Subsystem Service

Processus Windows (`lsass.exe`) responsable de l'authentification sur le système. Il gère :

- La vérification des credentials (login utilisateur)
- La création des tokens d'accès
- Le stockage en mémoire des credentials actifs (hashes NTLM, tickets Kerberos)

C'est précisément parce qu'il stocke les credentials en mémoire qu'il est ciblé par Mimikatz -> un attaquant qui dump LSASS récupère les hashes des utilisateurs connectés, potentiellement celui d'un admin de domaine.

**Ce qui est normal :** `svchost.exe` accède régulièrement à LSASS -> c'est légitime.

**Ce qui est suspect :** n'importe quel autre process qui y accède, surtout depuis un chemin inhabituel (`\Temp\`, `\Downloads\`…).

---

## LLMNR - Link-Local Multicast Name Resolution

Protocole Windows de résolution de noms en fallback. Quand le DNS ne répond pas, Windows broadcaste sur le réseau local : _"quelqu'un connaît ce nom d'hôte ?"_

Il gère :

- La résolution de noms quand le DNS échoue
- L'envoi d'une requête multicast à toutes les machines du sous-réseau
- L'authentification automatique NTLMv2 vers celui qui répond

C'est précisément parce qu'il accepte n'importe quelle réponse sans vérification qu'il est exploité via Responder -> un attaquant qui écoute le réseau répond à la place du vrai serveur, et la machine victime lui envoie automatiquement son hash NTLMv2, crackable offline avec hashcat.

Ce qui est normal : une requête LLMNR sans réponse suspecte, sur un réseau sans serveurs manquants.

Ce qui est suspect : un hôte qui répond à des requêtes LLMNR pour des noms inexistants, des authentifications NTLMv2 vers des IPs inconnues, des échecs d'auth (Event ID 4625) en rafale depuis une même machine.

**Remédiation** : désactiver LLMNR via GPO — `Computer Configuration → Administrative Templates → Network → DNS Client → Turn off Multicast Name Resolution → Enabled`.

---

