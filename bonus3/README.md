# Bonus3 - Exploitation binaire

## Configuration
- RELRO: No RELRO
- STACK CANARY: No canary found
- NX: NX enabled
- PIE: No PIE
- RPATH: No RPATH
- RUNPATH: No RUNPATH
- Fichier: /home/user/bonus3/bonus3

## Analyse du code

Le programme:
1. Ouvre et lit le fichier "/home/user/end/.pass"
2. Prend un argument en ligne de commande
3. Convertit cet argument en entier
4. Modifie le contenu lu du fichier .pass en fonction de cet entier
5. Compare le contenu modifié avec l'argument original
6. Exécute /bin/sh si la comparaison est vraie

## Exploitation

La vulnérabilité réside dans la manipulation de la chaîne lue et sa comparaison.

Solution:
1. Passer une chaîne vide ("") comme argument
2. Cela force le programme à mettre un octet nul au début de la chaîne lue
3. La comparaison se fait alors entre deux chaînes vides, ce qui est toujours vrai

## Exécution

Pour obtenir un shell:

```
./bonus3 ""
```
