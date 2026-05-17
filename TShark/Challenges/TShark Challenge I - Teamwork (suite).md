## Case: Teamwork!

An alert has been triggered: "The threat research team discovered a suspicious domain that could be a potential threat to the organisation."

The case was assigned to you. Inspect the provided **teamwork.pcap** located in `~/Desktop/exercise-files` and create artefacts for detection tooling.

**Your tools:** TShark, [VirusTotal(opens in new tab)](https://www.virustotal.com/gui/home/upload).

## Questions

Investigate the contacted domains.  
Investigate the domains by using VirusTotal.  
According to VirusTotal, there is a domain marked as malicious/suspicious.

### What is the full URL of the malicious/suspicious domain address?

Enter your answer in defanged format.

We first check all dns query name with this command :

```bash
tshark -r teamwork.pcap -T fields -e dns.qry.name | awk NF | sort -r | uniq -c | sort -r
```

command explaination :

|Partie|Rôle|
|---|---|
|`tshark -r teamwork.pcap`|Lit le fichier pcap|
|`-T fields`|Sortie en mode champs bruts|
|`-e dns.qry.name`|Extrait uniquement le champ nom de requête DNS|
|`awk NF`|Supprime les lignes vides|
|`sort -r`|Trie (prérequis pour `uniq`)|
|`uniq -c`|Déduplique + compte les occurrences|
|`sort -r`|Trie du plus fréquent au moins fréquent|

![[tshark challenge 1 q1.png]]

we can see a suspicious domain name with 19 occurences, so we check on VirusTotal :

![[tshark challenge 1 q1 virustotal.png]]

SOCRadar reported this URL as a Phishing URL

We need to defang. Go to CyberChef then defang this url to answer this question.

**Answer :** hxxp[://]www[.]paypal[.]com4uswebappsresetaccountrecovery[.]timeseaways[.]com/

### When was the URL of the malicious/suspicious domain address first submitted to VirusTotal?

Go to details in VirusTotal :

![[tshark challenge 1 q2.png]]

**Answer :** 2017-04-17 22:52:53 UTC

### Which known service was the domain trying to impersonate?

Easy question here

**Answer :** Paypal

### What is the IP address of the malicious domain?

We need to display a dns query type A with the malicious URL. So this command should work :

```bash
tshark -r teamwork.pcap -Y 'dns.qry.name contains "www.paypal.com4uswebappsresetaccountrecovery.timeseaways.com" and dns.qry.type == 1'
```

command explaination :

| Partie                        | Rôle                                               |
| ----------------------------- | -------------------------------------------------- |
| `tshark -r teamwork.pcap`     | Lit le fichier pcap                                |
| `-Y '...'`                    | Applique un display filter                         |
| `dns.qry.name contains "..."` | Filtre les requêtes DNS contenant ce domaine exact |
| `dns.qry.type == 1`           | Garde uniquement les requêtes de type A (IPv4)     |

![[tshark challenge 1 q4.png]]

Here is our answer. We just need to defang with cyberchef

**Answer :** 184[.]154[.]127[.]226

### What is the email address that was used?

Enter your answer in defanged format. (**format:** aaa[at]bbb[.]ccc)

To solve this question we need to filter post http methods, then searh for an email or a 'user'

this command should work :

```bash
tshark -r teamwork.pcap -Y 'http.request.method matches "POST"' -V | grep -i 'user'
```

command explanation :

|Partie|Rôle|
|---|---|
|`tshark -r teamwork.pcap`|Lit le fichier pcap|
|`-Y 'http.request.method matches "POST"'`|Filtre uniquement les requêtes HTTP POST|
|`-V`|Affiche le détail complet de chaque paquet|
|`grep -i 'user'`|Filtre les lignes contenant "user" (insensible à la casse)|

![[tshark challenge 1 q5.png]]

We need to defang with cyberchef

**Answer :** johnny5alive[at]gmail[.]com

---

## Next challenge

- [TShark Challenge II: Directory](https://tryhackme.com/r/room/tsharkchallengestwo)

