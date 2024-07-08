## Introduction

Ce projet vise à identifier et exploiter une vulnérabilité dans le programme binaire `level0` pour progresser d'un niveau à l'autre.

## Étapes suivies

### 1. Analyse du système

Le système a certaines protections activées, mais l'ASLR (Address Space Layout Randomization) est désactivé. L'ASLR rend les adresses mémoire imprévisibles, rendant les exploits plus difficiles. Sans ASLR, les adresses sont fixes et donc plus faciles à cibler.

### 2. Exploration initiale

Liste des fichiers disponibles :
```bash
ls
```
Exécution du programme :
```bash
./level0
Segmentation fault (core dumped)
```

### 3. Débogage avec GDB

Débogage pour localiser la faute de segmentation :
```bash
gdb level0
(gdb) b main
(gdb) run
(gdb) continue
```

### 4. Désassemblage de la fonction `main`

Pour comprendre le fonctionnement de la fonction `main`, nous utilisons la commande `disas main` dans `gdb` :
```gdb
(gdb) disas main
Dump of assembler code for function main:
   0x08048ec0 <+0>:	push   %ebp
   0x08048ec1 <+1>:	mov    %esp,%ebp
   0x08048ec3 <+3>:	and    $0xfffffff0,%esp
   0x08048ec6 <+6>:	sub    $0x20,%esp
   0x08048ec9 <+9>:	mov    0xc(%ebp),%eax
   0x08048ecc <+12>:	add    $0x4,%eax
   0x08048ecf <+15>:	mov    (%eax),%eax
   0x08048ed1 <+17>:	mov    %eax,(%esp)
   0x08048ed4 <+20>:	call   0x8049710 <atoi>
   0x08048ed9 <+25>:	cmp    $0x1a7,%eax
   0x08048ede <+30>:	jne    0x8048f58 <main+152>
   ...
```
Le désassemblage montre une comparaison après une conversion d'entrée en entier :
```assembly
call   atoi
cmp    $0x1a7,%eax  ; 423 en décimal
```

### 5. Fournir l'argument correct

Exécution du programme avec l'argument attendu :
```bash
./level0 423
```

### 6. Extraction du mot de passe du niveau suivant

Après avoir fourni l'argument correct, l'accès aux fichiers devient possible :
```bash
$ ls
ls: cannot open directory .: Permission denied
$ .pass
/bin/sh: 2: .pass: not found
$ cat /home/user/level0/.pass
cat: /home/user/level0/.pass: Permission denied
$ cat /home/user/level1/.pass                        
1fe8a524fa4bec01ca4ea2a869af2a02260d4a7d5fe7e7c24d8617e6dca12d3a
$ su level1
```

### Conclusion

Le programme `level0` attend l'argument `423`. En fournissant cet argument, le programme permet de lire le mot de passe du niveau suivant dans `/home/user/level1/.pass`.