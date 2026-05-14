
![[kernel unified hierarchy.png]]

## **hierarchy** 

- `/`: Root directory, the top level of the file system.
- `/home`: User home directories.
- `/bin`: Essential binary executables.
- `/sbin`: System administration binaries.
- `/etc`: Configuration files.
- `/var`: Variable data (logs, spool files).
- `/usr`: User programs and data.
- `/lib`: Shared libraries.
- `/tmp`: Temporary files.
- `/opt`: Third-party applications.

![[FHS linux.png]]

## **chmod**

![[chmod numcod.png]]

## **/etc/shadow

The encrypted password field contains the hashed passphrase with four components: prefix (algorithm id), options (parameters), salt, and hash. It is saved in the format `$prefix$options$salt$hash`

|Prefix|Algorithm|
|---|---|
|`$y$`|yescrypt is a scalable hashing scheme and is the default and recommended choice in new systems|
|`$gy$`|gost-yescrypt uses the GOST R 34.11-2012 hash function and the yescrypt hashing method|
|`$7$`|scrypt is a password-based key derivation function|
|`$2b$`, `$2y$`, `$2a$`, `$2x$`|bcrypt is a hash based on the Blowfish block cipher originally developed for OpenBSD but supported on a recent version of FreeBSD, NetBSD, Solaris 10 and newer, and several Linux distributions|
|`$6$`|sha512crypt is a hash based on SHA-2 with 512-bit output originally developed for GNU libc and commonly used on (older) Linux systems|
|`$md5`|SunMD5 is a hash based on the MD5 algorithm originally developed for Solaris|
|`$1$`|md5crypt is a hash based on the MD5 algorithm originally developed for FreeBSD|
![[hash linux.png]]

- **Algorithme** : yescrypt
- **Paramètres** : j9T
- **Salt** : valeur aléatoire
- **Hash** : empreinte finale

## **ressources**

https://roadmap.sh/linux GOAT

## **tar**

tar sert à regrouper plusieurs fichiers et dossiers en un seul fichier tout en conservant la structure, les permissions, les dates et les propriétaires. Il ne réduit pas la taille des données.

La compression (gzip, bzip2, xz) sert à réduire la taille des données en exploitant les répétitions. Elle fonctionne sur un flux de données ou sur un fichier unique et ne sait pas gérer plusieurs fichiers ni leurs métadonnées.

On peut utiliser tar seul :  
tar -cf archive.tar dossier/

On peut compresser un fichier seul :  
gzip fichier

Le plus courant est de combiner les deux en une seule commande :  
tar -czf archive.tar.gz dossier/

Options importantes de tar :  
-c créer une archive  
-x extraire  
-t lister le contenu  
-f nom du fichier (obligatoire)  
-z gzip  
-j bzip2  
-J xz  
-v affichage (optionnel)

Extraction :  
tar -xzf archive.tar.gz

À retenir : tar regroupe, la compression réduit la taille, tar peut appeler directement l’outil de compression choisi.