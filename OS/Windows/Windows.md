## **Authentification / AD**

| Concept / Protocole              | Rôle principal                                              | Contexte / Usage                           | Points clés / Interaction                                                                  | Risques / Limites                                                     |
| -------------------------------- | ----------------------------------------------------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------ | --------------------------------------------------------------------- |
| **NTLM**                         | Authentification héritée                                    | Windows, postes locaux ou anciens services | Basé sur **hash NTLM** ; peut être relayé ou réutilisé pour authentifier sans mot de passe | **Vulnérable** : Pass-the-Hash, relai NTLM ; pas de SSO sécurisé      |
| **Kerberos**                     | Authentification sécurisée par tickets                      | Active Directory / entreprises / SSO       | Remplace NTLM ; fonctionne avec **TGT et tickets de service** ; SSO interne                | Complexe à configurer ; Golden Ticket si krbtgt compromis             |
| **SSO (Single Sign-On)**         | Accès à plusieurs services après une seule authentification | Entreprises avec AD / Kerberos             | T’appuies sur Kerberos → un mot de passe → tickets pour tous les services                  | Dépend de la durée des tickets ; compromis = accès multiple           |
| **TGT (Ticket Granting Ticket)** | “Passe-partout” temporaire pour obtenir tickets de service  | Kerberos                                   | Généré à la connexion ; chiffré avec clé dérivée du mot de passe ; durée limitée (~10h)    | Si volé → Golden Ticket ; durée configurable                          |
| **Ticket de service**            | Autorisation pour accéder à un service spécifique           | Kerberos / SSO                             | Requiert un TGT ; utilisé pour chaque service sans retaper le mot de passe                 | Valable uniquement pour le service ciblé ; expire après durée définie |
| **Pass-the-Hash (PtH)**          | Exploitation des hash NTLM pour authentification            | Réseau Windows / NTLM                      | Permet mouvement latéral sans connaître le mot de passe                                    | Si compte admin réutilisé → compromission multiple machines           |
| **Comptes admins réutilisés**    | Facteur aggravant pour PtH                                  | Réseau Windows                             | Même mot de passe ou hash sur plusieurs machines                                           | Facilite propagation latérale et compromission complète               |
| **Golden Ticket**                | Fausse TGT pour accès illimité                              | Kerberos / AD                              | Exploite le hash du compte `krbtgt` pour générer TGT valides                               | Permet contrôle total du domaine ; attaque critique                   |
| **Silver Ticket**                | Fausse ticket de service pour un service spécifique         | Kerberos / AD                              | Crée un ticket pour un service précis, pas pour tout le domaine                            | Limité à un service, moins visible qu’un Golden Ticket                |

**Notes de lecture pratiques**

- **NTLM = legacy / vulnérable → Kerberos = moderne / sécurisé**
- **SSO repose sur Kerberos** pour éviter de taper le mot de passe à chaque service
- **TGT = ticket central de SSO**
- **Pass-the-Hash + comptes admin réutilisés = vecteur principal NTLM**
- **Golden/Silver Ticket = attaques liées à Kerberos, exploitation avancée**
- KDC (Key Distribution Center) = 

---

