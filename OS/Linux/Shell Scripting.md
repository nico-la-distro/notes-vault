# Shell Scripting — Guide (Bash)

> [!info] Référence Ce guide couvre Bash 5+. Les sections marquées `# POSIX` sont compatibles `sh`. Tester la version : `bash --version`

---

## Table des matières

- [[#Shebang & options de sécurité]]
- [[#Variables]]
- [[#Expansion de paramètres]]
- [[#Tableaux (arrays)]]
- [[#Arithmétique]]
- [[#Chaînes de caractères]]
- [[#Tests & conditions]]
- [[#Structures de contrôle]]
- [[#Fonctions]]
- [[#Arguments & options]]
- [[#Redirections & pipes]]
- [[#Substitutions]]
- [[#Heredoc & herestring]]
- [[#Expressions régulières]]
- [[#Globbing & pathname expansion]]
- [[#Gestion des erreurs]]
- [[#Jobs & processus]]
- [[#Bonnes pratiques]]

---

## Shebang & options de sécurité

```bash
#!/usr/bin/env bash
# Préférer env bash pour la portabilité (chemin de bash variable selon distrib)

set -e          # Quitter immédiatement si une commande échoue
set -u          # Traiter les variables non définies comme des erreurs
set -o pipefail # Propager les erreurs dans les pipes (pas seulement la dernière)
set -x          # Mode debug : afficher chaque commande avant exécution

# Forme condensée (la plus courante en production)
set -euo pipefail
```

> [!tip] Bonne habitude Toujours commencer ses scripts avec `set -euo pipefail`. Sans ça, une commande qui échoue au milieu du script continue silencieusement.

---

## Variables

```bash
# Déclaration — pas d'espaces autour du =
nom="Alice"
age=30
readonly PI=3.14159   # Variable en lecture seule

# Accès
echo "$nom"           # Toujours guillemeter pour éviter le word-splitting
echo "${nom}"         # Forme explicite (obligatoire dans les concaténations)

# Portée
local var="valeur"    # Uniquement dans une fonction
export VAR="valeur"   # Disponible pour les sous-processus

# Variables spéciales
echo "$0"   # Nom du script
echo "$1"   # Premier argument
echo "$#"   # Nombre d'arguments
echo "$@"   # Tous les arguments (chacun entre guillemets)
echo "$*"   # Tous les arguments (concaténés en une chaîne)
echo "$?"   # Code de retour de la dernière commande
echo "$$"   # PID du shell courant
echo "$!"   # PID du dernier processus en arrière-plan
echo "$-"   # Options actives du shell (ex: himBHs)
echo "$_"   # Dernier argument de la dernière commande

# Déclaration typée (Bash uniquement)
declare -i entier=42        # Entier (opérations arithmétiques auto)
declare -r CONSTANTE="val"  # Lecture seule
declare -x EXPORT="val"     # Exportée (équivalent export)
declare -l minuscule="ABC"  # Stocké en minuscules → "abc"
declare -u majuscule="abc"  # Stocké en majuscules → "ABC"
declare -a tableau          # Tableau indexé
declare -A dico             # Tableau associatif
declare -p nom              # Afficher la définition d'une variable
```

---

## Expansion de paramètres

```bash
str="Bonjour le monde"
fichier="/home/user/docs/rapport.txt"

# Valeurs par défaut
echo "${var:-défaut}"       # Si var vide/non définie → "défaut" (var inchangée)
echo "${var:=défaut}"       # Si var vide/non définie → assigne "défaut" à var
echo "${var:+autre}"        # Si var définie et non vide → "autre"
echo "${var:?message}"      # Si var vide/non définie → erreur avec message

# Longueur
echo "${#str}"              # 18

# Sous-chaîne  ${var:offset:longueur}
echo "${str:8:2}"           # "le"
echo "${str: -5}"           # "monde" (offset négatif = depuis la fin)

# Suppression de préfixe/suffixe
echo "${fichier##*/}"       # "rapport.txt"   (supprime jusqu'au dernier /)
echo "${fichier#*/}"        # "home/user/docs/rapport.txt" (supprime jusqu'au premier /)
echo "${fichier%%.*}"       # "/home/user/docs/rapport"    (supprime depuis le dernier .)
echo "${fichier%.*}"        # "/home/user/docs/rapport"    (supprime depuis le dernier .)
echo "${fichier%/*}"        # "/home/user/docs"            (répertoire parent)

# Remplacement  ${var/cherche/remplace}
echo "${str/monde/Bash}"    # "Bonjour le Bash"  (première occurrence)
echo "${str//e/E}"          # "Bonjour lE mondE" (toutes les occurrences)
echo "${str/#Bonjour/Hey}"  # "Hey le monde"     (ancré au début)
echo "${str/%monde/world}"  # "Bonjour le world" (ancré à la fin)

# Casse (Bash 4+)
echo "${str^}"              # "Bonjour le monde" (1ère lettre en majuscule)
echo "${str^^}"             # "BONJOUR LE MONDE" (tout en majuscules)
echo "${str,}"              # "bonjour le monde" (1ère lettre en minuscule)
echo "${str,,}"             # "bonjour le monde" (tout en minuscules)

# Indirection
var="nom"
nom="Alice"
echo "${!var}"              # "Alice" (valeur de la variable dont le nom est dans $var)

# Liste de variables commençant par un préfixe
echo "${!BASH*}"            # BASH BASHOPTS BASHPID BASH_ALIASES …
```

---

## Tableaux (arrays)

```bash
# ── Tableaux indexés ──────────────────────────────────────────────
fruits=("pomme" "banane" "cerise")
fruits[3]="datte"

echo "${fruits[0]}"         # "pomme"
echo "${fruits[-1]}"        # "datte" (dernier élément)
echo "${fruits[@]}"         # Tous les éléments
echo "${#fruits[@]}"        # Nombre d'éléments : 4
echo "${!fruits[@]}"        # Indices : 0 1 2 3

# Ajouter des éléments
fruits+=("figue" "grenade")

# Supprimer un élément
unset 'fruits[1]'           # Supprime "banane", l'indice 1 reste absent

# Sous-tableau  ${tableau[@]:offset:longueur}
echo "${fruits[@]:1:2}"     # "cerise datte"

# Copier un tableau
copie=("${fruits[@]}")

# Itérer
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done

# ── Tableaux associatifs (Bash 4+) ───────────────────────────────
declare -A ages
ages["Alice"]=30
ages["Bob"]=25
ages+=( ["Charlie"]=35 ["Diana"]=28 )

echo "${ages["Alice"]}"     # 30
echo "${!ages[@]}"          # Toutes les clés
echo "${ages[@]}"           # Toutes les valeurs
echo "${#ages[@]}"          # Nombre de paires

# Vérifier si une clé existe
[[ -v ages["Alice"] ]] && echo "clé présente"

# Itérer sur les paires
for cle in "${!ages[@]}"; do
    echo "$cle → ${ages[$cle]}"
done
```

---

## Arithmétique

```bash
# Évaluation arithmétique  $(( ))
a=10; b=3

echo $(( a + b ))    # 13
echo $(( a - b ))    # 7
echo $(( a * b ))    # 30
echo $(( a / b ))    # 3  (division entière)
echo $(( a % b ))    # 1  (modulo)
echo $(( a ** b ))   # 1000 (puissance)

# Opérateurs d'assignation
(( a += 5 ))
(( a++ ))            # post-incrément
(( ++a ))            # pré-incrément

# Opérateurs bit à bit
echo $(( 0b1010 & 0b1100 ))   # AND → 8
echo $(( 0b1010 | 0b1100 ))   # OR  → 14
echo $(( 0b1010 ^ 0b1100 ))   # XOR → 6
echo $(( 5 << 2 ))            # Shift gauche → 20
echo $(( 20 >> 2 ))           # Shift droite → 5

# Dans des conditions
if (( a > 10 )); then echo "grand"; fi

# Bases numériques
echo $(( 16#FF ))   # hex → 255
echo $(( 8#17 ))    # octal → 15
echo $(( 2#1010 ))  # binaire → 10

# Virgule flottante → utiliser bc ou awk
echo "scale=4; 10/3" | bc          # 3.3333
awk 'BEGIN { printf "%.4f\n", 10/3 }'  # 3.3333
```

---

## Chaînes de caractères

```bash
str="  Bonjour le monde  "

# Longueur
echo "${#str}"                          # 22

# Supprimer les espaces (trim) — aucune commande native, patterns suffisent
trimmed="${str#"${str%%[! ]*}"}"        # ltrim
trimmed="${trimmed%"${trimmed##*[! ]}"}" # rtrim

# Découpage avec IFS
IFS=',' read -ra parts <<< "a,b,c,d"
echo "${parts[1]}"                      # "b"

# Tester si une chaîne contient une sous-chaîne
if [[ "$str" == *"monde"* ]]; then echo "contient"; fi

# Tester si une chaîne commence/finit par
[[ "$str" == "Bonjour"* ]] && echo "commence par Bonjour"
[[ "$str" == *"monde" ]]   && echo "finit par monde"

# Concaténation
a="Hello"; b=" World"
c="${a}${b}"                            # "Hello World"

# Répétition (Bash 4+)
printf '%0.s-' {1..20}                 # --------------------

# Conversion vers tableau de caractères
read -ra chars <<< "$(echo "abc" | grep -o .)"
```

---

## Tests & conditions

```bash
# ── Test de fichiers ──────────────────────────────────────────────
[[ -e fichier ]]    # Existe (fichier ou répertoire)
[[ -f fichier ]]    # Fichier ordinaire
[[ -d fichier ]]    # Répertoire
[[ -L fichier ]]    # Lien symbolique
[[ -r fichier ]]    # Lisible
[[ -w fichier ]]    # Modifiable
[[ -x fichier ]]    # Exécutable
[[ -s fichier ]]    # Taille > 0
[[ -z fichier ]]    # Taille = 0 (vide)
[[ f1 -nt f2 ]]     # f1 plus récent que f2
[[ f1 -ot f2 ]]     # f1 plus ancien que f2
[[ f1 -ef f2 ]]     # f1 et f2 pointent sur le même inode

# ── Test de chaînes ───────────────────────────────────────────────
[[ -z "$str" ]]     # Chaîne vide
[[ -n "$str" ]]     # Chaîne non vide
[[ "$a" == "$b" ]]  # Égalité
[[ "$a" != "$b" ]]  # Différence
[[ "$a" < "$b" ]]   # Ordre lexicographique inférieur
[[ "$a" > "$b" ]]   # Ordre lexicographique supérieur
[[ "$str" =~ regex ]] # Correspond à l'expression régulière

# ── Test numérique ────────────────────────────────────────────────
[[ $a -eq $b ]]     # Égal
[[ $a -ne $b ]]     # Différent
[[ $a -lt $b ]]     # Inférieur strict
[[ $a -le $b ]]     # Inférieur ou égal
[[ $a -gt $b ]]     # Supérieur strict
[[ $a -ge $b ]]     # Supérieur ou égal
# Alternative avec (( )) plus lisible pour les entiers
(( a == b )), (( a < b )), (( a >= b ))

# ── Combinaisons logiques ─────────────────────────────────────────
[[ cond1 && cond2 ]]   # ET
[[ cond1 || cond2 ]]   # OU
[[ ! cond ]]           # NON

# ── Court-circuit ─────────────────────────────────────────────────
commande && echo "succès"      # Exécute si la commande réussit
commande || echo "échec"       # Exécute si la commande échoue
commande || exit 1             # Pattern courant pour stopper sur erreur
```

> [!warning] `[[ ]]` vs `[ ]` Préférer toujours `[[ ]]` en Bash : pas de word-splitting, supporte `&&`, `||`, `=~`, et les patterns. `[ ]` est POSIX mais piégeux.

---

## Structures de contrôle

```bash
# ── if / elif / else ──────────────────────────────────────────────
if [[ $age -ge 18 ]]; then
    echo "majeur"
elif [[ $age -ge 13 ]]; then
    echo "adolescent"
else
    echo "enfant"
fi

# ── case ──────────────────────────────────────────────────────────
case "$langue" in
    fr|FR)      echo "Français" ;;
    en|EN)      echo "Anglais"  ;;
    de)         echo "Allemand" ;;
    es|pt)      echo "Ibérique" ;;
    *)          echo "Inconnu"  ;;  # défaut
esac

# Bash 4+ : ;& continue sur le cas suivant, ;;& teste les cas suivants
case "$str" in
    *hello*)  echo "contient hello" ;;&
    *world*)  echo "contient world" ;;
esac

# ── for ───────────────────────────────────────────────────────────
# Sur une liste
for item in un deux trois; do
    echo "$item"
done

# Sur un tableau
for item in "${tableau[@]}"; do
    echo "$item"
done

# Style C
for (( i=0; i<10; i++ )); do
    echo "$i"
done

# Sur des fichiers
for f in /etc/*.conf; do
    echo "$f"
done

# ── while ─────────────────────────────────────────────────────────
compteur=0
while (( compteur < 5 )); do
    echo "$compteur"
    (( compteur++ ))
done

# Lire un fichier ligne par ligne
while IFS= read -r ligne; do
    echo "$ligne"
done < fichier.txt

# Lire la sortie d'une commande
while IFS= read -r ligne; do
    echo "$ligne"
done < <(find . -name "*.sh")   # process substitution

# ── until ─────────────────────────────────────────────────────────
until [[ -f "/tmp/signal" ]]; do
    sleep 1
done

# ── select (menu interactif) ──────────────────────────────────────
PS3="Choisissez un fruit : "
select fruit in pomme banane cerise quitter; do
    case $fruit in
        quitter) break ;;
        *)       echo "Vous avez choisi : $fruit" ;;
    esac
done

# ── Contrôle de boucle ────────────────────────────────────────────
break           # Quitter la boucle
break 2         # Quitter 2 niveaux de boucles imbriquées
continue        # Passer à l'itération suivante
continue 2      # Passer à l'itération suivante du niveau supérieur
```

---

## Fonctions

```bash
# Déclaration (les deux syntaxes sont équivalentes)
function saluer() {
    local prenom="$1"           # local limite la portée à la fonction
    local -r TITRE="M."         # local + lecture seule
    echo "Bonjour, $TITRE $prenom !"
}

calculer() {
    local -i a=$1 b=$2
    echo $(( a + b ))           # "return" ne peut renvoyer qu'un entier 0-255
}                               # Pour des valeurs, utiliser echo + substitution

# Appel
saluer "Alice"
resultat=$(calculer 10 5)       # Capturer la sortie

# Valeur de retour
est_pair() {
    (( $1 % 2 == 0 ))           # Le code de retour de (( )) est 0 si vrai
}
est_pair 4 && echo "pair"

# Fonctions récursives
factorielle() {
    local n=$1
    (( n <= 1 )) && echo 1 && return
    echo $(( n * $(factorielle $(( n - 1 ))) ))
}

# Passer un tableau par référence (Bash 4.3+)
doubler() {
    local -n ref=$1             # nameref : ref est un alias de la variable passée
    for i in "${!ref[@]}"; do
        ref[$i]=$(( ref[$i] * 2 ))
    done
}
nums=(1 2 3 4)
doubler nums
echo "${nums[@]}"               # 2 4 6 8

# Fonctions variadiques
somme() {
    local total=0
    for n in "$@"; do
        (( total += n ))
    done
    echo "$total"
}
somme 1 2 3 4 5                 # 15
```

---

## Arguments & options

```bash
# Arguments positionnels
echo "Script : $0"
echo "Arg 1  : $1"
echo "Tous   : $@"
echo "Nombre : $#"

shift           # Décale $2→$1, $3→$2, etc.
shift 2         # Décale de 2 positions

# ── getopts (options courtes) ─────────────────────────────────────
usage() {
    echo "Usage: $0 [-v] [-f fichier] [-n nombre] argument"
    exit 1
}

verbose=false; fichier=""; nombre=1

while getopts ":vf:n:h" opt; do
    case $opt in
        v)  verbose=true ;;
        f)  fichier="$OPTARG" ;;
        n)  nombre="$OPTARG" ;;
        h)  usage ;;
        :)  echo "Option -$OPTARG nécessite un argument" >&2; exit 1 ;;
        \?) echo "Option inconnue : -$OPTARG" >&2; exit 1 ;;
    esac
done
shift $(( OPTIND - 1 ))         # Retirer les options, garder les arguments restants
echo "Arguments restants : $@"

# ── Parsing manuel (options longues) ─────────────────────────────
while [[ $# -gt 0 ]]; do
    case "$1" in
        --verbose|-v)   verbose=true; shift ;;
        --fichier=*)    fichier="${1#*=}"; shift ;;
        --fichier)      fichier="$2"; shift 2 ;;
        --nombre)       nombre="$2"; shift 2 ;;
        --)             shift; break ;;         # Fin des options
        -*)             echo "Option inconnue : $1" >&2; exit 1 ;;
        *)              break ;;
    esac
done
```

---

## Redirections & pipes

```bash
# ── Redirections standards ────────────────────────────────────────
commande > fichier          # stdout → fichier (écrase)
commande >> fichier         # stdout → fichier (ajoute)
commande 2> erreurs.log     # stderr → fichier
commande 2>> erreurs.log    # stderr → fichier (ajoute)
commande &> tout.log        # stdout + stderr → fichier
commande > out.log 2>&1     # idem (ordre important : 2>&1 après >)
commande 2>&1 | pipe        # stderr dans le pipe
commande > /dev/null        # Jeter stdout
commande 2>/dev/null        # Jeter stderr
commande &>/dev/null        # Jeter tout

# ── Descripteurs de fichiers personnalisés ────────────────────────
exec 3> journal.log         # Ouvrir fd 3 en écriture
echo "message" >&3          # Écrire sur fd 3
exec 3>&-                   # Fermer fd 3

exec 4< données.txt         # Ouvrir fd 4 en lecture
read -r ligne <&4           # Lire depuis fd 4
exec 4<&-                   # Fermer fd 4

# ── tee (écrire et passer) ────────────────────────────────────────
commande | tee fichier              # stdout → écran + fichier
commande | tee -a fichier           # ajoute au fichier
commande | tee f1 f2 | suite        # vers plusieurs fichiers

# ── Pipes et pipefail ─────────────────────────────────────────────
# Avec set -o pipefail, le code de retour du pipe est celui de la commande
# qui a échoué (pas forcément la dernière)
cat inexistant.txt | grep motif     # Sans pipefail : exit 0 ! Avec : exit 1

# Codes de retour individuels dans un pipe
ls -la | grep txt | wc -l
echo "${PIPESTATUS[@]}"             # ex: "0 0 0"
```

---

## Substitutions

```bash
# ── Substitution de commande ──────────────────────────────────────
date=$(date +%Y-%m-%d)          # Forme moderne (préférable)
date=`date +%Y-%m-%d`           # Ancienne forme (éviter)

# Imbriquée (facile avec $() contrairement aux backticks)
resultat=$(echo $(( $(cat /proc/sys/vm/swappiness) * 2 )))

# ── Process substitution ──────────────────────────────────────────
# Traite la sortie d'une commande comme un fichier temporaire

# Comparer deux listes
diff <(ls dir1/) <(ls dir2/)

# Lire depuis plusieurs sources simultanément
while IFS= read -r ligne; do
    echo "$ligne"
done < <(find . -name "*.sh" | sort)

# Envoyer vers plusieurs destinations
commande > >(tee sortie.log) 2> >(tee erreurs.log >&2)

# ── Substitution arithmétique ─────────────────────────────────────
x=5
echo "Résultat : $(( x * (x + 1) / 2 ))"   # 15
```

---

## Heredoc & herestring

```bash
# ── Heredoc ───────────────────────────────────────────────────────
# Les variables sont interpolées
cat << EOF
Bonjour $USER,
Nous sommes le $(date +%d/%m/%Y).
EOF

# Sans interpolation (guillemeter le délimiteur)
cat << 'EOF'
Ceci est littéral : $USER $(date)
EOF

# Indenter avec tiret (supprime les tabulations initiales)
cat <<- EOF
	Ligne indentée avec des tabs
	Autre ligne
EOF

# Vers un fichier
cat > config.txt << EOF
[settings]
user=$USER
date=$(date +%Y-%m-%d)
EOF

# Passer à une commande
mysql -u root << EOF
USE mabase;
SELECT * FROM users LIMIT 10;
EOF

# ── Herestring ────────────────────────────────────────────────────
# Passer une chaîne à stdin
read a b c <<< "un deux trois"
echo "$a $b $c"                     # "un deux trois"

grep "motif" <<< "$variable"

base64 --decode <<< "SGVsbG8gV29ybGQ="
```

---

## Expressions régulières

```bash
# Opérateur =~ dans [[ ]]
str="user@example.com"
regex='^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'

if [[ "$str" =~ $regex ]]; then
    echo "Email valide"
    echo "Match complet : ${BASH_REMATCH[0]}"
    # Les groupes capturants sont dans BASH_REMATCH[1], [2], ...
fi

# Groupes capturants
date_str="2024-12-25"
if [[ "$date_str" =~ ^([0-9]{4})-([0-9]{2})-([0-9]{2})$ ]]; then
    annee="${BASH_REMATCH[1]}"   # 2024
    mois="${BASH_REMATCH[2]}"    # 12
    jour="${BASH_REMATCH[3]}"    # 25
fi

# Stocker la regex dans une variable (obligatoire pour les regex complexes)
ip_regex='^([0-9]{1,3}\.){3}[0-9]{1,3}$'
[[ "192.168.1.1" =~ $ip_regex ]] && echo "IP valide"

# grep avec ERE (-E) ou PCRE (-P)
echo "test123" | grep -E '^[a-z]+[0-9]+$'
echo "test123" | grep -P '(?<=[a-z])\d+'   # lookahead/lookbehind

# sed avec regex
echo "Bonjour Monde" | sed 's/\b[A-Z][a-z]*/(&)/g'   # (Bonjour) (Monde)

# awk avec regex
echo "192.168.1.1 accès OK" | awk '/^[0-9]/ { print $1 }'
```

---

## Globbing & pathname expansion

```bash
# ── Globs de base ─────────────────────────────────────────────────
ls *.txt            # Tous les .txt
ls fichier?.log     # Un seul caractère quelconque
ls [abc]*           # Commence par a, b ou c
ls [!abc]*          # Ne commence pas par a, b ou c
ls [0-9]*           # Commence par un chiffre

# ── Globbing étendu — activer avec shopt ──────────────────────────
shopt -s extglob

ls ?(motif)         # 0 ou 1 occurrence
ls *(motif)         # 0 ou N occurrences
ls +(motif)         # 1 ou N occurrences
ls @(motif)         # Exactement 1 occurrence
ls !(motif)         # Tout sauf ce motif

# Exemples concrets
ls !(*.txt)             # Tous les fichiers sauf les .txt
ls +(*.jpg|*.png)       # Fichiers jpg ou png
rm !(rapport|backup).*  # Supprimer tout sauf rapport.* et backup.*

# ── Globstar (récursif) ───────────────────────────────────────────
shopt -s globstar

ls **/*.sh          # Tous les .sh récursivement
ls **/logs/         # Tous les répertoires "logs" récursivement

# ── Brace expansion ───────────────────────────────────────────────
echo {a,b,c}.txt        # a.txt b.txt c.txt
echo {1..5}             # 1 2 3 4 5
echo {01..05}           # 01 02 03 04 05
echo {a..z}             # a b c ... z
echo {1..10..2}         # 1 3 5 7 9  (incrément de 2)

mkdir -p projet/{src,tests,docs,bin}
cp fichier.{txt,bak}    # Copie fichier.txt vers fichier.bak

# ── Options shopt utiles ──────────────────────────────────────────
shopt -s nocaseglob     # Insensible à la casse
shopt -s nullglob       # Pattern sans correspondance → rien (pas d'erreur)
shopt -s dotglob        # Inclure les fichiers cachés (commençant par .)
shopt -s failglob       # Erreur si aucune correspondance
```

---

## Gestion des erreurs

```bash
# ── Codes de retour ───────────────────────────────────────────────
commande
echo $?             # 0 = succès, 1-255 = erreur

# Convention : 0 succès, 1 erreur générale, 2 mauvaise utilisation,
# 126 non exécutable, 127 commande introuvable, 128+N signal N

# ── trap ──────────────────────────────────────────────────────────
# Exécuter du code quand un signal/événement se produit

# Nettoyage à la sortie (EXIT est toujours appelé)
nettoyage() {
    local code=$?
    rm -f /tmp/fichier_temp_$$
    [[ $code -ne 0 ]] && echo "Script terminé avec erreur : $code" >&2
    exit $code
}
trap nettoyage EXIT

# Signaux courants
trap 'echo "Interrompu"; exit 130' INT    # Ctrl+C
trap 'echo "Terminé";    exit 143' TERM   # kill
trap ''                            HUP    # Ignorer les hangups
trap - INT                                # Restaurer le comportement par défaut

# Afficher la ligne d'erreur (avec set -e)
trap 'echo "Erreur ligne $LINENO" >&2' ERR

# Fichier temporaire sécurisé avec nettoyage automatique
tmp=$(mktemp)
trap 'rm -f "$tmp"' EXIT

# ── Fonctions d'erreur ────────────────────────────────────────────
die() {
    echo "ERREUR: $*" >&2
    exit 1
}

verifier() {
    [[ -f "$1" ]] || die "Fichier introuvable : $1"
    [[ -r "$1" ]] || die "Fichier non lisible : $1"
}

# Exécuter ou mourir
run() { "$@" || die "La commande a échoué : $*"; }

# ── Journalisation ────────────────────────────────────────────────
LOG_FILE="/var/log/monscript.log"

log() {
    local niveau="$1"; shift
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$niveau] $*" | tee -a "$LOG_FILE" >&2
}

log INFO  "Démarrage du script"
log WARN  "Fichier de config manquant, utilisation des valeurs par défaut"
log ERROR "Connexion à la base de données échouée"
```

---

## Jobs & processus

```bash
# ── Arrière-plan ──────────────────────────────────────────────────
commande &                  # Lancer en arrière-plan
echo "PID : $!"             # PID du dernier processus en arrière-plan

# ── wait ──────────────────────────────────────────────────────────
pid1=$!
commande2 &
pid2=$!

wait $pid1                  # Attendre un processus spécifique
wait $pid2
wait                        # Attendre tous les processus en arrière-plan

# Parallélisme avec gestion d'erreur
pids=()
for item in "${items[@]}"; do
    traiter "$item" &
    pids+=($!)
done
for pid in "${pids[@]}"; do
    wait "$pid" || echo "Échec du processus $pid"
done

# ── jobs / fg / bg ────────────────────────────────────────────────
jobs                        # Lister les jobs
fg %1                       # Ramener le job 1 au premier plan
bg %1                       # Continuer le job 1 en arrière-plan
kill %1                     # Tuer le job 1

# ── Sous-shells ───────────────────────────────────────────────────
# Les variables modifiées dans un sous-shell n'affectent pas le parent
(
    cd /tmp
    export VAR="valeur locale"
    commandes...
)
# Ici, on est toujours dans le répertoire original, VAR inchangée

# ── Coprocesses (Bash 4+) ─────────────────────────────────────────
# Lancer un processus en arrière-plan avec des FDs bidirectionnels
coproc MON_PROC { bc -l; }

echo "3.14 * 2" >&"${MON_PROC[1]}"   # Envoyer vers le coprocess
read resultat <&"${MON_PROC[0]}"      # Lire la réponse
echo "$resultat"                       # 6.28000...

# ── Limites et priorités ──────────────────────────────────────────
ulimit -n 4096              # Max fichiers ouverts
ulimit -v 1048576           # Max mémoire virtuelle (Ko)
ulimit -t 60                # Max CPU (secondes)

nice -n 10 commande         # Lancer avec priorité réduite
renice +5 -p $pid           # Changer la priorité d'un processus

# ── IPC simple via FIFOs ──────────────────────────────────────────
mkfifo /tmp/pipe_$$
trap 'rm -f /tmp/pipe_$$' EXIT

producteur > /tmp/pipe_$$ &
consommateur < /tmp/pipe_$$
```

---

## Bonnes pratiques

```bash
# ── Structure type d'un script ────────────────────────────────────
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'                 # IFS stricte : seulement newline et tab

readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"
readonly VERSION="1.0.0"

usage() {
    cat << EOF
Usage: $SCRIPT_NAME [OPTIONS] argument

Options:
  -h, --help      Afficher cette aide
  -v, --verbose   Mode verbeux
  -V, --version   Afficher la version

EOF
    exit 0
}

main() {
    # Logique principale ici
    :
}

main "$@"
```

> [!tip] Règles d'or
> 
> 1. **Guillemeter toutes les variables** : `"$var"` — protège du word-splitting et du globbing
> 2. **`set -euo pipefail`** en haut de chaque script
> 3. **`local`** dans toutes les fonctions pour éviter les fuites de variables
> 4. **`readonly`** pour les constantes
> 5. **Tester les entrées** avant de les utiliser (`[[ -f "$fichier" ]]`)
> 6. **Nettoyage avec `trap ... EXIT`** pour les ressources temporaires
> 7. **`shellcheck`** — linter indispensable : `shellcheck monscript.sh`

```bash
# ── Patterns utiles ───────────────────────────────────────────────

# Répertoire du script courant (résout les symlinks)
DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# Verrou : empêcher deux instances simultanées
[[ -f /tmp/monscript.lock ]] && { echo "Déjà en cours" >&2; exit 1; }
touch /tmp/monscript.lock
trap 'rm -f /tmp/monscript.lock' EXIT

# Vérifier que les dépendances sont installées
for cmd in jq curl git; do
    command -v "$cmd" &>/dev/null || { echo "Manque : $cmd" >&2; exit 1; }
done

# Lire un fichier .env
while IFS='=' read -r cle valeur; do
    [[ "$cle" =~ ^#|^$ ]] && continue      # Ignorer commentaires et lignes vides
    export "${cle}=${valeur}"
done < .env

# Spinner pendant une longue opération
spinner() {
    local pid=$1
    local chars='⠋⠙⠹⠸⠼⠴⠦⠧⠇⠏'
    while kill -0 "$pid" 2>/dev/null; do
        for (( i=0; i<${#chars}; i++ )); do
            printf "\r%s En cours…" "${chars:i:1}"
            sleep 0.1
        done
    done
    printf "\r✓ Terminé\n"
}

longue_commande &
spinner $!
wait $!
```

---

## 🔗 Voir aussi

- [[Raccourcis terminal Linux]]
- [[Commandes Linux essentielles]]
- `man bash` — la référence complète
- `shellcheck` — [shellcheck.net](https://www.shellcheck.net/) — linter en ligne
- `bash --version` — vérifier la version installée