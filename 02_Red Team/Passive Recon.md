## Introduction

Outils couverts :

- `whois` -> requêtes WHOIS
- `nslookup` -> requêtes DNS
- `dig` -> requêtes DNS (plus détaillé)
- [DNSDumpster](https://dnsdumpster.com/) -> service en ligne
- [Shodan.io](https://www.shodan.io/) -> service en ligne

## Passive Versus Active Recon

**Passive** : collecte d'infos sans interaction directe avec la cible -> légal, indétectable. Exemples : DNS lookup public, lecture d'offres d'emploi, articles de presse, réseaux sociaux.

**Active** : interaction directe avec la cible -> détectable, nécessite autorisation légale. Exemples : connexion HTTP/FTP/SMTP, social engineering, intrusion physique.

Réponses aux questions :

- Facebook page d'une entreprise -> **P** (passif)
- Ping du serveur web -> **A** (actif)
- Social engineering à une fête -> **A** (actif)

## Whois

Protocole WHOIS -> TCP port 43. Maintenu par le registrar du domaine.

Infos récupérables :

- Registrar (via qui le domaine est enregistré)
- Coordonnées du registrant (souvent masquées par un service de privacy)
- Dates : création, mise à jour, expiration
- Name Servers

bash

```bash
whois DOMAIN_NAME
whois tryhackme.com
```

Note : beaucoup de services WHOIS masquent les emails pour éviter le spam/harvest.

## nslookup and dig

### nslookup

bash

```bash
nslookup DOMAIN_NAME
nslookup OPTIONS DOMAIN_NAME SERVER
nslookup -type=A tryhackme.com 1.1.1.1
nslookup -type=MX tryhackme.com
```

|Query type|Résultat|
|---|---|
|A|IPv4|
|AAAA|IPv6|
|CNAME|Canonical Name|
|MX|Serveurs mail|
|SOA|Start of Authority|
|TXT|Enregistrements TXT|

DNS publics : Cloudflare `1.1.1.1` / `1.0.0.1`, Google `8.8.8.8` / `8.8.4.4`, Quad9 `9.9.9.9`

### dig

Retourne plus d'infos que `nslookup` (ex: TTL par défaut).

bash

```bash
dig DOMAIN_NAME TYPE
dig @SERVER DOMAIN_NAME TYPE
dig tryhackme.com MX
dig @1.1.1.1 tryhackme.com MX
```

---

## DNSDumpster

`nslookup` et `dig` ne trouvent pas les sous-domaines. DNSDumpster oui.

[https://dnsdumpster.com/](https://dnsdumpster.com/)

Retourne : sous-domaines, enregistrements DNS (A, MX, TXT), IP résolues, géolocalisation, graphe exportable.

## Shodan.io

Moteur de recherche pour appareils connectés (pas des pages web). Se connecte à chaque IP accessible et indexe les réponses.

[https://www.shodan.io/](https://www.shodan.io/) | [Help / query fundamentals](https://help.shodan.io/the-basics/search-query-fundamentals)

Infos récupérables : IP, hébergeur, localisation, type et version du serveur. Utilisation : rechercher un domaine ou une IP directement.

Room dédiée : [TryHackMe Shodan.io](https://tryhackme.com/room/shodan)

## Summary

|But|Commande|
|---|---|
|WHOIS record|`whois tryhackme.com`|
|DNS A (IPv4)|`nslookup -type=A tryhackme.com`|
|DNS MX via serveur précis|`nslookup -type=MX tryhackme.com 1.1.1.1`|
|DNS TXT|`nslookup -type=TXT tryhackme.com`|
|DNS A avec dig|`dig tryhackme.com A`|
|DNS MX via serveur précis|`dig @1.1.1.1 tryhackme.com MX`|
|DNS TXT avec dig|`dig tryhackme.com TXT`|

Ressource complémentaire : [DNS in Detail](https://tryhackme.com/room/dnsindetail)

