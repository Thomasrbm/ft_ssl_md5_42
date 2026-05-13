<div align="center">

# ft_ssl_md5

**Réimplémentation en C des commandes `md5` et `sha256` d'OpenSSL.**

*Projet 42 — cursus tronc commun, branche cryptographie.*

[![C](https://img.shields.io/badge/language-C-blue.svg)](https://en.wikipedia.org/wiki/C_(programming_language))
[![Norm](https://img.shields.io/badge/norm-42-success.svg)](https://github.com/42School/norminette)
[![Build](https://img.shields.io/badge/build-passing-brightgreen.svg)](#compilation)

</div>

---

## Sommaire

- [Présentation](#présentation)
- [Algorithmes implémentés](#algorithmes-implémentés)
- [Structure du projet](#structure-du-projet)
- [Compilation](#compilation)
- [Utilisation](#utilisation)
  - [Mode commande](#mode-commande)
  - [Mode interactif (REPL)](#mode-interactif-repl)
  - [Drapeaux supportés](#drapeaux-supportés)
- [Exemples](#exemples)
- [Tests](#tests)
- [Référence OpenSSL via Docker](#référence-openssl-via-docker)
- [Implémentation](#implémentation)
- [Ressources](#ressources)
- [Auteur](#auteur)

---

## Présentation

`ft_ssl` est une réimplémentation **from scratch** des sous-commandes de hashage de la suite OpenSSL.
Le binaire produit reproduit fidèlement le comportement, le formatage de sortie et les drapeaux des commandes
`openssl md5` et `openssl sha256`, sans dépendance externe à `libcrypto`.

L'objectif pédagogique est triple :

1. Comprendre en profondeur les algorithmes de hashage cryptographique (Merkle–Damgård, fonctions de compression, padding, endianness).
2. Manipuler les opérations bit à bit et les rotations 32 bits en C.
3. Reproduire à l'identique l'interface utilisateur d'un outil de référence.

---

## Algorithmes implémentés

| Algorithme | RFC / Standard           | Taille du digest | Taille de bloc |
|------------|--------------------------|------------------|----------------|
| **MD5**    | [RFC 1321](https://www.rfc-editor.org/rfc/rfc1321) | 128 bits (32 hex) | 512 bits |
| **SHA-256**| [FIPS 180-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf) | 256 bits (64 hex) | 512 bits |

> **Note de sécurité :** MD5 est considéré comme **cryptographiquement cassé** depuis 2004 (collisions pratiques).
> Il reste utilisable pour des vérifications d'intégrité non adverses. Pour tout usage sécurité, préférer SHA-256.

---

## Structure du projet

```
ft_ssl_md5_42/
├── includes/
│   ├── ft_ssl.h          # interface principale, types, table des algos
│   ├── md5.h             # constantes MD5, macros F/G/H/I, ROTL
│   └── sha256.h          # constantes SHA-256, macros CH/MAJ/SIGMA
├── srcs/
│   ├── ft_ssl.c          # CLI, dispatch, REPL, gestion des flags
│   ├── md5.c             # implémentation MD5
│   └── sha256.c          # implémentation SHA-256
├── commented/            # versions annotées pédagogiques
├── Makefile              # cibles : all, clean, fclean, re, docker, docker-clean
├── Dockerfile            # image Ubuntu 18.04 + OpenSSL 1.1.1 (référence)
└── test_ft_ssl.sh        # suite de tests fonctionnels
```

---

## Compilation

```bash
make            # produit le binaire ./ft_ssl
make clean      # supprime les .o
make fclean     # supprime les .o et le binaire
make re         # fclean + all
```

Flags de compilation : `-Wall -Wextra -Werror` (conformité Norme 42).

---

## Utilisation

```
usage: ./ft_ssl command [flags] [file/string]
```

### Mode commande

```bash
./ft_ssl md5    [-pqr] [-s string] [files...]
./ft_ssl sha256 [-pqr] [-s string] [files...]
```

### Mode interactif (REPL)

Lancé en l'absence d'argument :

```bash
$ ./ft_ssl
ft_ssl> md5 -s "hello"
MD5 ("hello") = 5d41402abc4b2a76b9719d911017c592
ft_ssl> sha256 -s "hello"
SHA256 ("hello") = 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
```

### Drapeaux supportés

| Flag | Effet |
|:----:|-------|
| `-p` | Echo de l'entrée standard, puis hashage |
| `-q` | Mode silencieux : affiche uniquement le digest |
| `-r` | Reverse format : `digest  filename` (style `md5sum`) |
| `-s` | Hash de la chaîne suivante passée en argument |

---

## Exemples

```bash
# Hash d'une chaîne
$ echo -n "42" | ./ft_ssl md5
(stdin)= a1d0c6e83f027327d8461063f4ac58a6

# Hash d'un fichier
$ ./ft_ssl sha256 /etc/hostname
SHA256 (/etc/hostname) = 5e88...

# Mode -s
$ ./ft_ssl md5 -s "Hello, World!"
MD5 ("Hello, World!") = 65a8e27d8879283831b664bd8b7f0ad4

# Mode silencieux
$ ./ft_ssl sha256 -q -s "42"
73475cb40a568e8da8a045ced110137e159f890ac4da883b6b17dc651b3a8049

# Format reverse
$ ./ft_ssl md5 -r -s "42"
a1d0c6e83f027327d8461063f4ac58a6 "42"

# Combinaison de fichiers et chaînes
$ ./ft_ssl md5 -s "foo" /etc/hostname -s "bar"
MD5 ("foo") = acbd18db4cc2f85cedef654fccc4a4d8
MD5 (/etc/hostname) = ...
MD5 ("bar") = 37b51d194a7513e45b56f6524f2d51f2
```

---

## Tests

Une suite de tests fonctionnels couvre les cas usuels et limites
(chaîne vide, absence de newline, fichiers inexistants, combinaisons de flags) :

```bash
make
./test_ft_ssl.sh
```

Sortie attendue : `[OK]` sur l'ensemble des cas, comparés à `openssl` de référence.

---

## Référence OpenSSL via Docker

Le `Makefile` fournit une cible permettant de monter une image Ubuntu 18.04 avec OpenSSL 1.1.1 en référence :

```bash
make docker                    # construit l'image
docker run -it ft_ssl_ref      # entre dans le container

# Dans le container :
openssl
OpenSSL> md5 /tmp/file
OpenSSL> sha256 -r /tmp/file
```

Pour nettoyer : `make docker-clean`.

---

## Implémentation

### MD5 (RFC 1321)

1. **Padding** — On ajoute un bit `1` puis des `0` jusqu'à `length ≡ 56 (mod 64)`,
   puis on concatène la longueur originale en bits sur 64 bits **little-endian**.
2. **Initialisation** des registres `A B C D` avec les constantes magiques.
3. **Compression** — Pour chaque bloc de 512 bits, 64 tours (4 rounds × 16 ops)
   utilisant les fonctions non-linéaires `F`, `G`, `H`, `I`, les rotations `g_s[i]`
   et la table `g_t[i] = floor(2^32 · |sin(i+1)|)`.
4. **Sortie** : concaténation de `A | B | C | D` en little-endian → 32 caractères hex.

### SHA-256 (FIPS 180-4)

1. **Padding** identique à MD5 mais longueur encodée en **big-endian** sur 64 bits.
2. **Expansion** du message : 16 mots de 32 bits étendus à 64 via `σ0`/`σ1`.
3. **Compression** — 64 rounds utilisant `Σ0`, `Σ1`, `Ch`, `Maj` et la table `K[64]`
   (les 32 premiers bits des racines cubiques des 64 premiers nombres premiers).
4. **Sortie** : concaténation `H0…H7` en big-endian → 64 caractères hex.

### Points d'attention

- **Endianness** : MD5 produit son digest en *little-endian*, SHA-256 en *big-endian*.
  C'est la principale source d'erreurs lors d'une réimplémentation.
- **Lecture binaire** : `fopen(path, "rb")` est obligatoire pour ne pas corrompre les fichiers
  contenant `\r` ou des séquences binaires sous Windows.
- **`echo` ajoute un `\n`** : la sortie en mode `-p` retire ce newline final pour rester
  conforme au comportement d'OpenSSL.

---

## Ressources

- RFC 1321 — [The MD5 Message-Digest Algorithm](https://www.rfc-editor.org/rfc/rfc1321)
- FIPS 180-4 — [Secure Hash Standard](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf)
- Wikipedia — [SHA-2 (pseudocode)](https://en.wikipedia.org/wiki/SHA-2#Pseudocode)
- NIST — [Cryptographic Algorithm Validation Program](https://csrc.nist.gov/projects/cryptographic-algorithm-validation-program)

---

## Auteur

**thomasrbm** — 42 School
