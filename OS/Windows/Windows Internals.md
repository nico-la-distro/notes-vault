Room focus : internals Windows -> processus, formats de fichiers, kernel.

Pertinence red team : comprendre les internals permet l'évasion et l'exploitation lors du développement d'outils offensifs.

---

## Processes

Un processus = représentation de l'exécution d'un programme. Une application peut avoir plusieurs processus.

Composants clés (haut niveau) :

|Composant|Rôle|
|---|---|
|Private Virtual Address Space|Mémoire virtuelle allouée au processus|
|Executable Program|Code et données dans l'espace mémoire|
|Open Handles|Accès aux ressources système|
|Security Context|Token d'accès (utilisateur, groupes, privilèges)|
|Process ID (PID)|Identifiant numérique unique|
|Threads|Unité d'exécution du processus|

Structure en mémoire :

|Composant|Rôle|
|---|---|
|Code|Instructions exécutées|
|Global Variables|Variables stockées|
|Process Heap|Heap de stockage des données|
|Process Resources|Ressources additionnelles|
|Environment Block|Structure de données du processus|

![[structure_en_memoire.png]]

Exemples de processus Windows natifs : `MsMpEng` (Defender), `wininit` (clavier/souris), `lsass` (credentials).

**Vecteurs d'attaque sur les processus :**

- Process Injection -> T1055
- Process Hollowing -> T1055.012
- Process Masquerading -> T1055.013

**Outils d'observation :** Process Hacker 2, Process Explorer, Procmon, Task Manager.

Champs utiles dans le Task Manager : Name, PID, Status, User name.

---

## Threads

Thread = unité d'exécution d'un processus, schedulée par le kernel selon le CPU, la mémoire et la priorité.

Partagent avec le processus parent : code, variables globales, ressources.

Données propres au thread :

|Composant|Rôle|
|---|---|
|Stack|Données spécifiques au thread (exceptions, appels)|
|Thread Local Storage|Pointeurs vers un environnement de données unique|
|Stack Argument|Valeur unique assignée au thread|
|Context Structure|Valeurs des registres machine (maintenues par le kernel)|

**Intérêt offensif :** les threads sont ciblés pour l'exécution de code, souvent chaînés avec des appels API dans des techniques plus larges.

---

## Virtual Memory

Chaque processus a un espace d'adressage virtuel privé. Un **memory manager** traduit les adresses virtuelles en adresses physiques -> les processus n'écrivent jamais directement en mémoire physique, ce qui évite les collisions entre applications.

Si une application utilise plus de mémoire virtuelle que de RAM disponible, le memory manager **page** l'excédent sur le disque.

![[virtual_memory.png]]

L'espace d'adressage virtuel est découpé en deux zones :

- **Zone basse** -> réservée aux processus utilisateur
- **Zone haute** -> réservée au kernel/OS

Sur 32-bit (4 GB total) :

```
0x00000000 - 0x7FFFFFFF  -> processus (2 GB)
0x80000000 - 0xFFFFFFFF  -> kernel    (2 GB)
```

Sur 64-bit (256 TB total) -> même découpage moitié/moitié, mais l'espace est tellement grand que ça ne pose plus aucun problème.

Sur 32-bit, si une app a besoin de plus de 2 GB -> `increaseUserVA` ou **AWE** permettent de modifier le découpage. Sur 64-bit ce n'est plus nécessaire.

**Pourquoi c'est utile en offensif :** en regardant une adresse mémoire, tu sais immédiatement si elle appartient à un processus ou au kernel.

![[theorical_maximum_virtual_addess.png]]

---

## Dynamic Link Libraries

DLL = fichier contenant du code et des données partageable entre plusieurs programmes simultanément.

Avantages : modularité, réutilisation du code, économie de RAM (une seule copie en mémoire physique pour tous les processus qui l'utilisent).

Quand un programme dépend d'une DLL -> vecteurs d'attaque :

- DLL Hijacking -> T1574.001
- DLL Side-Loading -> T1574.002
- DLL Injection -> T1055.001

**Deux méthodes de chargement :**

**Load-time dynamic linking** -> la DLL est chargée au démarrage du programme. Nécessite un fichier `.h` et `.lib` au moment de la compilation. Appels directs aux fonctions de la DLL.

cpp

```cpp
#include "sampleDLL.h"
HelloWorld(); // appel direct
```

**Run-time dynamic linking** -> la DLL est chargée manuellement pendant l'exécution via `LoadLibrary`/`LoadLibraryEx`, puis `GetProcAddress` pour récupérer la fonction.

cpp

```cpp
hinstDLL = LoadLibrary("sampleDLL.dll");
HelloWorld = (DLLPROC) GetProcAddress(hinstDLL, "HelloWorld");
(HelloWorld);
FreeLibrary(hinstDLL);
```

**Intérêt offensif du run-time linking :** pas besoin de fichiers externes à la compilation, et le transfert d'une seule DLL entre zones mémoire est plus discret.

---

## Portable Executable Format

Le format PE définit la structure des fichiers exécutables et objets Windows. Un PE est composé de fichiers PE + COFF (**C**ommon **O**bject **F**ile **F**ormat).

![[portable_executable_format.png]]

**Structure d'un fichier PE (7 composants) :**

|Composant|Rôle|
|---|---|
|DOS Header|Identifie le fichier comme `.exe` (commence par `MZ`)|
|DOS Stub|Affiche "This program cannot be run in DOS mode" -> ignorable|
|PE File Header|Métadonnées du binaire (signature `PE`, format, flags)|
|Image Optional Header|Informations importantes malgré le nom (point d'entrée, tailles)|
|Data Dictionaries|Partie de l'optional header, pointent vers les données de l'image|
|Section Table|Définit les sections disponibles et leur contenu|
|Sections|Contenu réel du fichier|

**Sections principales :**

|Section|Contenu|
|---|---|
|`.text`|Code exécutable et point d'entrée|
|`.data`|Données initialisées (strings, variables)|
|`.rdata` / `.idata`|Imports (WinAPI) et DLLs|
|`.reloc`|Informations de relocation|
|`.rsrc`|Ressources (images, icônes)|
|`.debug`|Informations de debug|

**Signature d'identification rapide dans un hex dump :**

- `MZ` en offset 0 -> DOS header -> c'est un `.exe`
- `PE` -> début du PE file header

---

### Interacting with Windows Internals

L'interface principale pour interagir avec les internals Windows = **Windows API (Win32 API)**.

**Modes processeur :**

|User Mode|Kernel Mode|
|---|---|
|Pas d'accès direct au hardware|Accès direct au hardware|
|Espace mémoire virtuel privé|Espace mémoire virtuel partagé unique|
|Accès mémoire limité|Accès à toute la mémoire physique|

![[usermode_kernelmode.png]]

Par défaut, les applications tournent en user mode. Un system call ou API call déclenche le passage en kernel mode -> c'est le **Switching Point**.

Certains langages ajoutent une couche supplémentaire avant l'API (ex: C# passe par le CLR avant d'atteindre Win32).

**Proof-of-concept : injection d'une message box en mémoire**

4 étapes -> 4 appels WinAPI :

cpp

```cpp
// 1. Obtenir un handle sur le processus cible
HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);

// 2. Allouer de la mémoire dans le processus cible
remoteBuffer = VirtualAllocEx(hProcess, NULL, sizeof payload,
    MEM_RESERVE | MEM_COMMIT, PAGE_EXECUTE_READWRITE);

// 3. Écrire le payload dans la mémoire allouée
WriteProcessMemory(hProcess, remoteBuffer, payload, sizeof payload, NULL);

// 4. Créer un thread distant pour exécuter le payload
remoteThread = CreateRemoteThread(hProcess, NULL, 0,
    (LPTHREAD_START_ROUTINE)remoteBuffer, NULL, 0, NULL);
```

Ces 4 appels (`OpenProcess`, `VirtualAllocEx`, `WriteProcessMemory`, `CreateRemoteThread`) sont la base de la **process injection** -> à retenir.

---

## Windows Internals - Résumé

### Processes

- Processus = conteneur de ressources (mémoire, handles, token de sécurité)
- Composants clés : virtual address space, threads, PID, security context
- Vecteurs d'attaque : Process Injection (T1055), Hollowing (T1055.012), Masquerading (T1055.013)
- Outils d'observation : Process Hacker 2, Process Explorer, Procmon

### Threads

- Thread = unité d'exécution réelle sur le CPU (registres + stack propres)
- Partagent la mémoire du processus parent
- Ciblés pour l'exécution de code malveillant (`CreateRemoteThread`)

### Virtual Memory

- Chaque processus a un espace d'adressage virtuel privé -> le memory manager traduit vers la RAM physique
- Évite les collisions entre processus, permet le partage de DLLs
- Découpage : moitié basse = processus, moitié haute = kernel
- Débordement RAM -> pagination sur disque (page file)

### DLLs

- Fichier contenant des fonctions partagées entre plusieurs programmes
- Load-time : chargée au démarrage, nécessite `.h` + `.lib`
- Run-time : chargée pendant l'exécution via `LoadLibrary` + `GetProcAddress` -> privilégié en offensif
- Vecteurs d'attaque : DLL Hijacking (T1574.001), Side-Loading (T1574.002), Injection (T1055.001)

### Portable Executable Format

- Structure de tout exécutable Windows
- Signature : `MZ` (DOS header) + `PE` (PE header)
- Sections importantes : `.text` (code), `.data` (variables), `.rdata`/`.idata` (imports/DLLs), `.rsrc` (ressources)

### Interacting with Windows Internals

- Interface principale : **Win32 API**
- User mode -> Kernel mode via system calls (**Switching Point**)
- Pattern de base pour la process injection :

```
OpenProcess          -> handle sur le processus cible
VirtualAllocEx       -> allouer de la mémoire
WriteProcessMemory   -> écrire le payload
CreateRemoteThread   -> exécuter le payload
```

