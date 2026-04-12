## Level 03

### Analyse

L'analyse du binaire révèle une faille de logique cryptographique (XOR) couplée à une utilisation de system(). Le programme attend un nombre en entrée, effectue un calcul, puis utilise le résultat comme clé de déchiffrement pour débloquer un shell.

main : Initialise le générateur aléatoire (srand) et lit un entier via scanf. Il appelle ensuite la fonction test(input, 0x1337d00d).

test : Calcule la différence entre 0x1337d00d et notre entrée. Si cette différence est inférieure ou égale à 21 ($0x15$), elle appelle decrypt avec cette différence comme argument.

decrypt :Une chaîne chiffrée est stockée sur la pile (ex: 0x757c7d51). Une boucle XOR est appliquée entre chaque octet de cette chaîne et l'argument reçu. Le résultat est comparé avec une chaîne de référence située à l'adresse 0x80489c3. Si la comparaison réussit, system("/bin/sh") est exécuté.

Puisque nous connaissons la chaîne chiffrée (dans le code) et la chaîne attendue (en mémoire), nous pouvons mathématiquement déduire la clé nécessaire.


### Exploitation

Nous devons trouver la valeur de l'input qui, après soustraction dans test, donnera la clé XOR correcte dans decrypt.

En examinant la mémoire avec GDB, on récupère les premiers octets :
Chaîne chiffrée (premier octet à -0x1d(%ebp)) : 0x51 (tiré de 0x757c7d51 en Little Endian).
Chaîne de référence (à 0x80489c3) : Le premier caractère est 'C' (0x43).

la clé XOR: 0x51 ^ 0x43 = 0x12
0x12 en décimal correspond à 18.

Dans la fonction test, le calcul est 0x1337d00d - input = clé. Nous savons que 0x1337d00d vaut 322424845 en décimal. 322424845 - input = 18, input = 322424845 - 18 = 322424827.


Password:
322424827
$ whoami
level04
$ cat /home/users/level04/.pass
...