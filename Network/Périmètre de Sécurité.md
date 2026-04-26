
## ACL (couche 2 OSI)

![[périmètre sécurité couche 2 ACL.png]]

## Parefeu stateless/stateful (couche 4)

**5-tuple (stateful)**

![[périmètre sécurité 5-tuple.png]]

permet aux parefeu stateful de garder une trace de la connexion. toute connexion entrante est autorisé si elle correspond à une connexion sortante. (Suivis de connexion TCP/IP)


**Stateless**

![[périmètre sécurité stateless.png]]

## NAT filtrage (couche 4)

![[périmètre sécurité NAT.png]]


## Proxy et ALG (couche 7)

![[périmètre sécurité proxy.png]]

ALG (application layer gateway) : inspecte les packets jusqu'à la couche 7
- permet d'ouvrir les port de manière dynamique uniquement pour les sessions correspondantes (ex: FTP, SIP)

Le proxy cache l'utilisateur d'internet

