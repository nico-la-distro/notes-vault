- **Cloud computing** = utiliser des ressources IT (**serveurs, stockage, réseau**) via **Internet**.
    
- Permet une app **accessible partout**, **fiable**, **scalable**.

Le cloud permet de :

- **scaler facilement**
- **réduire les coûts**
- **améliorer la fiabilité**
- **se concentrer sur le produit** (pas le hardware)

## Avantages & Inconvénients

|**Bénéfices du cloud**|**Inconvénients / limites**|
|---|---|
|**Scalabilité** : ressources ↑↓ selon besoin|**Dépendance à Internet** (accès/service)|
|**Paiement à l’usage** : moins de coût initial|**Coûts variables** (peuvent augmenter si mauvaise gestion)|
|**Déploiement rapide** (on-demand)|**Moins de contrôle** en public cloud|
|**Haute disponibilité** (meilleure résilience)|**Dépendance au fournisseur** (vendor lock-in possible)|
|**Accès global** (utilisateurs partout)|**Contraintes de conformité** (données sensibles/réglementation)|
|**Sécurité infra gérée par le provider**|**Responsabilité partagée** : mauvaise config côté client = risque|
|**Moins de hardware à gérer**|**Complexité** (services, architecture, coûts)|

**A retenir** :

- **+** Flexible, rapide, scalable, souvent plus rentable au départ
    
- **−** Dépendance (internet/provider), coûts à surveiller, conformité/sécurité à bien gérer

---

## Types & Models

|Type|Idée|
|---|---|
|**Public**|Peu cher, scalable, pas d’infra à gérer|
|**Private**|Plus de contrôle / conformité|
|**Hybrid**|Mix privé + public|

---

| Modèle   | Tu gères     | Provider gère | Acronymes                              |
| -------- | ------------ | ------------- | -------------------------------------- |
| **IaaS** | OS + app     | Matériel      | **Infrastructure as a Service (IaaS)** |
| **PaaS** | App          | Infra + OS    | **Platform as a Service (PaaS)**       |
| **SaaS** | Rien (usage) | Tout          | **Software as a Service (SaaS)**       |

- **Infrastructure as a Service (IaaS):** You rent basic computing resources such as virtual servers, storage, and networking. You are responsible for managing the operating system and your application, while the provider manages the physical hardware.
- **Platform as a Service (PaaS):** The cloud provider manages the infrastructure and the operating system. You focus on building, deploying, and running your application without worrying about servers.
- **Software as a Service (SaaS):** You use a complete application over the internet. The provider manages everything, and you access the software through a browser or app, for example, Gmail or Zoom.

---

## Providers

- **AWS** (leader)
- **Azure**
- **GCP**
- Alibaba Cloud
- IBM Cloud
- Oracle Cloud

![[Cloud Providers.png]]

---

## Exemples d'utilisation

- **Netflix** : scale mondial, dispo élevée
- **Spotify** : gère gros volume users/music
- **Instagram** : stockage massif + diffusion rapide
- **E-commerce** : pics de trafic (Black Friday)

---

## Exercice Cloud THM

|Terme|Définition|
|---|---|
|**EC2**|Machine virtuelle (serveur) dans le cloud|
|**Instance Type**|Puissance de la VM (CPU/RAM) + coût|

- **EC2 = VM**
- **Instance type = puissance + prix**
- **IaaS = accès OS complet**
- **Region = emplacement géographique**
- **3 instances** : 1 app + 2 study machines
- **Stop instances inutilisées = baisse des coûts**

| Terme             | Définition courte                                    |
| ----------------- | ---------------------------------------------------- |
| **Public Cloud**  | Cloud partagé via Internet                           |
| **Private Cloud** | Cloud dédié à une seule entreprise                   |
| **Hybrid Cloud**  | Mélange public + privé                               |
| **IaaS**          | Location de ressources (serveurs, stockage, réseau)  |
| **PaaS**          | Environnement prêt pour développer/déployer des apps |
| **SaaS**          | Logiciel en ligne prêt à l’emploi (ex: Gmail, Zoom)  |
| **EC2**           | VM/serveur cloud d’AWS                               |