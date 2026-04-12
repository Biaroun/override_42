## Level 07

### Analyse

L'analyse de l'exécutable (via objdump) révèle une boucle interactive permettant de lire (read_number) et d'écrire (store_number) des entiers dans un tableau alloué sur la pile (stack).

La vulnérabilité principale est un Out-of-Bounds Write (écriture hors limites) : le programme ne vérifie jamais si l'index fourni par l'utilisateur dépasse la taille réelle du tableau. Cela permet d'écrire n'importe où sur la pile, et notamment d'écraser l'adresse de retour (EIP) de la fonction main pour détourner le flux d'exécution.

Le programme refuse d'écrire si l'index est un multiple de 3 (Index % 3 == 0) mais ce filtre modulo 3 peut être contourné grâce à un Integer Overflow (dépassement d'entier) inhérent à l'architecture 32-bits.

### Exploitation


Nous allons rediriger l'EIP vers la fonction system() et lui passer la chaîne "/bin/sh" en argument.
En analysant la pile, on constate que l'adresse de retour (EIP) du main se trouve à 456 octets du début du tableau.
Puisque chaque case du tableau stocke un entier de 4 octets :
Index cible = 456 / 4 = 114

Faux_Index * 4 = 456 + 4 294 967 296
L'index 114 est bloqué car 114 % 3 = 0.En 32-bits, la mémoire "boucle" tous les 4 294 967 296 octets (2^{32}). L'adresse finale est calculée en faisant Index * 4.Pour faire un tour complet et retomber sur la case 114, on ajoute un quart de la limite mémoire à notre index :114 + (4 294 967 296 / 4) = 1073741938. L'index 1073741938 pointe physiquement sur l'EIP

Adresse de system : 0xf7e6aed0 = 4159090384, l'adresse de "/bin/sh" : 0xf7f897ec = 4160264172.

Input command: store
 Number: 4159090384
 Index: 1073741938

 Input command: store
 Number: 4160264172
 Index: 116

 Input command: quit