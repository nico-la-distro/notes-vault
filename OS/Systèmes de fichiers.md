### **FAT32**
- Taille max d’un fichier : **4 Go**
- Taille max partition : ~**2 To**
- Pas de journalisation
- Pas de permissions
- Très grande compatibilité
- Technologie ancienne
### **exFAT**
- Taille max fichier : **théoriquement très élevée** (≫ To)
- Taille max partition : **≫ To**
- Pas de journalisation
- Pas de permissions POSIX
- Conçu pour supports amovibles
- Structure simple, faible overhead
### **NTFS**
- Taille max fichier : **≫ To**
- Taille max partition : **≫ To**
- Journalisation
- Permissions ACL
- Compression, chiffrement, liens
- Système robuste et complexe
### **ext4**
- Taille max fichier : **16 To**
- Taille max partition : **1 EiB**
- Journalisation
- Permissions POSIX
- Très stable et performant
- Standard Linux
### **Btrfs**
- Taille max fichier : **16 EiB**
- Taille max partition : **16 EiB**
- Copy-on-write
- Snapshots natifs
- Checksums des données
- Gestion avancée des volumes
### **XFS**
- Taille max fichier : **8 EiB**
- Taille max partition : **8 EiB**
- Journalisation
- Très performant sur gros fichiers
- Peu flexible (redimensionnement)
### **ZFS**
- Taille max fichier : **≈ 16 EiB**
- Taille max pool : **≫ EiB**
- Copy-on-write
- Snapshots, compression, déduplication
- Checksums + auto-réparation
- Très lourd, très robuste
### **Résumé technique**
- **Simple / compatible** : FAT32, exFAT
- **Généraliste robuste** : NTFS, ext4
- **Avancé / entreprise** : Btrfs, XFS, ZFS