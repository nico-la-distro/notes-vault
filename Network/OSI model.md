![[OSI model.png]]

![[TCP-IP + OSI.png]]
![[OSI+TCPIP.png]]
![[OSI Fr.png]]

1.  Physical Layer :
- physical transmission (electrical, mechanical, and procedural)
- cables / connectors / signaling

2. Date Link Layer
- error-free transmission of data frames
- ensure data integrity, handles flow control, and manages access to the physical medium
(Ethernet is an example of a data link layer protocol)

3. Network Layer
- addressing and routing of data packets
- best path for data transmission (considering factors such as network congestion, packet priority, and network topology)
(IP [Internet Protocol] is a widely used network layer protocol)

4. Transport Layer
- provides reliable, end-to-end data transport services
- breaks down the data into smaller segments
- ensures that they are correctly delivered to the destination
(Transmission Control Protocol [TCP] is a well-known transport layer protocol)

5. Session Layer
- establishes, manages, and terminates communication sessions between two network applications
- allows applications to synchronize their communication and provides mechanisms for checkpointing and recovery
- This layer ensures that data exchange occurs in an organized and secure manner

6. Presentation Layer
- responsible for data representation and conversion
- handles tasks such as data encryption, compression, and formatting
- ensuring that data sent by the application layer of one system can be understood by the application layer of another system

7. Application Layer
- closest layer to the end user
- enables users to access network services and interacts with software applications
(HTTP (Hypertext Transfer Protocol) for web browsing and SMTP (Simple Mail Transfer Protocol) for email communication)
_______________________________________________________
Importante Point :
Actual network protocols and technologies often combine multiple layers into a single implementation for efficiency and practicality. The TCP/IP protocol suite, which is widely used in today's internet, is an example of such an implementation.

---

|Couche|Nom|Rôle principal|Mots-clés / exemples|
|--:|---|---|---|
|7|Application|Donne aux **applications** un moyen d’utiliser le réseau|navigateur, client mail, API…|
|6|Présentation|**Standardise** les données + gère **chiffrement / compression / transformations**|encodage, TLS, compression|
|5|Session|**Établit / maintient / synchronise** une session de communication (évite le mélange entre flux)|sessions multiples (onglets)|
|4|Transport|Choix du protocole + **découpe** en morceaux|**TCP** (fiable) / **UDP** (rapide), segments/datagrams|
|3|Réseau|Trouve la **destination** et la **route** via adressage logique|**IP**, routage, IPv4 (ex: 192.168.1.1)|
|2|Liaison de données|Ajoute adressage **physique (MAC)** + format trame + **détection d’erreurs**|**NIC**, MAC (spoofable)|
|1|Physique|Transmet des **signaux** (électriques/ondes) et convertit binaire ↔ signaux|câbles, radio, impulsions|