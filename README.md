### README - Projet Rainfall

## Introduction

Le projet Rainfall est une initiative de sécurité informatique visant à explorer et comprendre les vulnérabilités couramment présentes dans les logiciels. À travers plusieurs niveaux de difficulté croissante, les participants apprennent à identifier et exploiter différentes failles de sécurité dans des programmes binaires. Chaque niveau présente un nouveau défi, renforçant les compétences en analyse de code, débogage et exploitation de vulnérabilités.

## Objectifs du Projet

Le projet Rainfall a plusieurs objectifs pédagogiques :

1. **Compréhension des vulnérabilités** : Familiariser les participants avec les types de vulnérabilités courantes, telles que les débordements de tampon, les chaînes de format, et les mauvaises utilisations de pointeurs.
2. **Développement de compétences en débogage** : Utiliser des outils de débogage pour analyser le comportement des programmes et identifier les failles de sécurité.
3. **Exploitation pratique** : Apprendre et mettre en pratique les techniques d'exploitation pour tirer parti des vulnérabilités identifiées et atteindre des objectifs spécifiques, tels que l'obtention de mots de passe ou de privilèges accrus.
4. **Automatisation et scripting** : Développer des scripts pour automatiser les processus d'exploitation, rendant les tâches répétitives plus efficaces.

## Méthodologie

Les participants sont invités à :

1. **Analyser les programmes** : Examiner le code source ou désassemblé pour identifier les points faibles et comprendre le fonctionnement interne des programmes.
2. **Utiliser des outils de sécurité** : Appliquer des outils comme GDB, objdump, ltrace, et strace pour déboguer les programmes et tracer leurs exécutions.
3. **Développer des exploits** : Écrire des scripts, souvent en Python, pour exploiter les vulnérabilités et automatiser les attaques.
4. **Vérifier les résultats** : Confirmer l'exploitation réussie en atteignant les objectifs définis, comme l'accès à des fichiers protégés ou l'obtention de privilèges utilisateur accrus.

## Outils Utilisés

Pour atteindre ces objectifs, les outils suivants sont fréquemment utilisés :

1. **GDB (GNU Debugger)** : Un outil puissant pour déboguer les programmes et analyser les flux d'exécution.
2. **Python** : Utilisé pour écrire des scripts d'exploitation et automatiser les tâches.
3. **Objdump** : Pour désassembler les binaires et examiner le code machine.
4. **Ltrace et Strace** : Pour tracer les appels de bibliothèque et les appels système, respectivement.

## Petit point sur les sécurité configuré 

1. **GCC stack protector support: Enabled**
   - C'est une protection du compilateur contre les débordements de pile.
   - Mais elle n'est pas activée pour ce programme spécifique.

2. **Strict user copy checks: Disabled**
   - Concerne la copie de données entre l'espace utilisateur et le noyau.
   - Désactivé ici, ce qui peut être moins sécurisé.

3. **Restrict /dev/mem access: Enabled**
   - Limite l'accès direct à la mémoire physique.
   - C'est une bonne chose pour la sécurité.

4. **Restrict /dev/kmem access: Enabled**
   - Limite l'accès à la mémoire du noyau.
   - Également bon pour la sécurité.

5. **grsecurity / PaX: No GRKERNSEC**
   - Ce sont des patchs de sécurité avancés pour le noyau Linux.
   - Ils ne sont pas présents ici.

6. **Kernel Heap Hardening: No KERNHEAP**
   - Protection supplémentaire pour le tas du noyau.
   - Non activée ici.

7. **System-wide ASLR: Off**
   - ASLR rend aléatoire l'emplacement des zones mémoire.
   - Désactivé ici, ce qui facilite l'exploitation.

8. **RELRO: No RELRO**
   - Protection qui rend certaines sections mémoire en lecture seule.
   - Non activée, ce qui peut être exploité.

9. **STACK CANARY: No canary found**
   - Protection contre les débordements de pile.
   - Absente ici, rendant les débordements de pile plus faciles.

10. **NX: NX disabled**
    - NX empêche l'exécution de code sur la pile.
    - Désactivé, permettant l'exécution de code injecté sur la pile.

11. PIE: No PIE
    - Rend aléatoire l'emplacement du programme en mémoire.
    - Non activé, facilitant la prédiction des adresses.

12. RPATH/RUNPATH: No RPATH, No RUNPATH
    - Chemins de recherche pour les bibliothèques partagées.
    - Non définis, ce qui est généralement plus sûr.

## Conclusion

Le projet Rainfall offre une immersion pratique dans le monde de la sécurité informatique, permettant aux participants de développer des compétences critiques en analyse de vulnérabilités et en exploitation. C'est une excellente opportunité pour quiconque souhaite approfondir ses connaissances en sécurité logicielle et en programmation défensive.