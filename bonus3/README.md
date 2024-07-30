# Bonus3 - Exploitation binaire

## Configuration
- RELRO: No RELRO
- STACK CANARY: No canary found
- NX: NX enabled
- PIE: No PIE
- RPATH: No RPATH
- RUNPATH: No RUNPATH
- Fichier: /home/user/bonus3/bonus3

## Code décompilé

```c
undefined4 main(int param_1,int param_2)
{
  undefined4 uVar1;
  int iVar2;
  undefined4 *puVar3;
  byte bVar4;
  undefined4 local_98 [16];
  undefined local_57;
  char local_56 [66];
  FILE *local_14;
  
  bVar4 = 0;
  local_14 = fopen("/home/user/end/.pass","r");
  puVar3 = local_98;
  for (iVar2 = 0x21; iVar2 != 0; iVar2 = iVar2 + -1) {
    *puVar3 = 0;
    puVar3 = puVar3 + (uint)bVar4 * -2 + 1;
  }
  if ((local_14 == (FILE *)0x0) || (param_1 != 2)) {
    uVar1 = 0xffffffff;
  }
  else {
    fread(local_98,1,0x42,local_14);
    local_57 = 0;
    iVar2 = atoi(*(char **)(param_2 + 4));
    *(undefined *)((int)local_98 + iVar2) = 0;
    fread(local_56,1,0x41,local_14);
    fclose(local_14);
    iVar2 = strcmp((char *)local_98,*(char **)(param_2 + 4));
    if (iVar2 == 0) {
      execl("/bin/sh","sh",0);
    }
    else {
      puts(local_56);
    }
    uVar1 = 0;
  }
  return uVar1;
}
```

## Analyse du code

Le programme:
1. Ouvre et lit le fichier "/home/user/end/.pass"
2. Prend un argument en ligne de commande
3. Convertit cet argument en entier
4. Modifie le contenu lu du fichier .pass en fonction de cet entier
5. Compare le contenu modifié avec l'argument original
6. Exécute /bin/sh si la comparaison est vraie

## Exploitation

La vulnérabilité réside dans la manipulation de la chaîne lue et sa comparaison.

Solution:
1. Passer une chaîne vide ("") comme argument
2. Cela force le programme à mettre un octet nul au début de la chaîne lue
3. La comparaison se fait alors entre deux chaînes vides, ce qui est toujours vrai

## Exécution

Pour obtenir un shell:

```
./bonus3 ""
```
